# DVA-C02 — Phần 1: Compute & Containers

> **Tổng quan chuỗi tài liệu DVA-C02:**
> - **Phần 1** (file này): Compute & Containers — Lambda, EC2, ECS, EKS, Elastic Beanstalk
> - Phần 2: Storage & Database — S3, EBS, EFS, DynamoDB, RDS, Aurora, ElastiCache
> - Phần 3: Application Integration — SQS, SNS, EventBridge, Step Functions, Kinesis, API Gateway
> - Phần 4: Security & Identity — IAM, Cognito, KMS, Secrets Manager, SSM Parameter Store, STS
> - Phần 5: Deployment & Monitoring — CI/CD (CodeCommit/Build/Deploy/Pipeline), CloudFormation, SAM, CDK, CloudWatch, X-Ray
> - Phần 6: Networking & Edge + Tips thi & câu hỏi "bẫy" hay gặp

---

## 1. AWS Lambda (⭐ CỰC KỲ QUAN TRỌNG — chiếm nhiều câu nhất trong bài thi)

### 1.1 Giới hạn (limits) — PHẢI THUỘC LÒNG

| Thuộc tính | Giới hạn |
|---|---|
| **Memory** | 128 MB → 10,240 MB (10 GB), bước nhảy 1 MB |
| **vCPU** | Tỷ lệ thuận với memory. Đạt **1 vCPU tại 1,769 MB**, tối đa ~6 vCPU ở 10 GB |
| **Timeout tối đa** | **15 phút** (900 giây) |
| **Deployment package — zip upload trực tiếp** | 50 MB |
| **Deployment package — zip qua S3** | 250 MB (đã giải nén) |
| **Container image** | **10 GB** |
| **/tmp storage** | 512 MB mặc định, có thể cấu hình tới **10 GB** |
| **Environment variables** | Tổng size 4 KB |
| **Concurrent executions (account)** | 1,000 mặc định (soft limit, xin tăng được) |
| **Invocation payload — sync** | 6 MB request + response |
| **Invocation payload — async** | 256 KB |
| **Layers** | Tối đa **5 layers/function**, mỗi layer ≤ 250 MB (đã giải nén, cộng cả function) |

> 💡 **Trick:** Đề thường hỏi "function xử lý file 8GB, chạy 2 phút" → KHÔNG dùng Lambda được (do /tmp chỉ 10GB max, hoặc do timeout nếu >15 phút). Dùng **Fargate** hoặc **Batch**.

### 1.2 Invocation types

- **Synchronous** (CLI, API Gateway, ALB, Cognito): caller chờ kết quả. Lỗi → caller tự retry.
- **Asynchronous** (S3, SNS, EventBridge): Lambda **tự động retry 2 lần** (tổng 3 lần). Thất bại → gửi vào **DLQ** (SQS/SNS) hoặc **Destinations** (SQS/SNS/Lambda/EventBridge).
- **Event source mapping / Poll-based** (SQS, Kinesis, DynamoDB Streams, Kafka): Lambda tự poll.

> ⚠️ **Bẫy thường gặp:** DLQ chỉ áp dụng cho **async invocation**. Với SQS event source → thông điệp sẽ quay lại queue và cuối cùng vào **source queue's DLQ** (cấu hình ở SQS, không phải ở Lambda).

### 1.3 Destinations vs DLQ

| | DLQ | Destinations |
|---|---|---|
| Xuất hiện | Cũ | Mới hơn, khuyến nghị |
| Targets | SQS, SNS | SQS, SNS, Lambda, EventBridge |
| On success | ❌ | ✅ |
| On failure | ✅ | ✅ |
| Metadata đầy đủ | ❌ | ✅ |

### 1.4 Concurrency

- **Reserved concurrency**: dành riêng slot cho function → đảm bảo có slot khi cần, **đồng thời** giới hạn tối đa (throttle bảo vệ downstream như RDS).
- **Provisioned concurrency**: khởi tạo sẵn execution environments → **loại bỏ cold start**. Tính phí. Dùng với alias/version, KHÔNG dùng với `$LATEST`.
- **Unreserved**: tổng còn lại phải ≥ **100** (nếu bạn reserve quá tay, các function khác sẽ bị throttle).

> 💡 **Trick cold start:** Giải pháp theo thứ tự ưu tiên:
> 1. **Provisioned Concurrency** (hiệu quả nhất)
> 2. Tăng memory (khởi tạo nhanh hơn)
> 3. Viết bằng ngôn ngữ khởi động nhanh (Node.js, Python > Java, .NET)
> 4. Giảm kích thước package
> 5. Init SDK clients **bên ngoài handler** (tái dùng giữa các invocation)

### 1.5 Lambda@Edge vs CloudFront Functions

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript (nhẹ) | Node.js, Python |
| Memory | Rất nhỏ | Tới 10 GB |
| Timeout | **< 1 ms** | 5s (viewer), 30s (origin) |
| Triggers | Viewer request/response | Tất cả 4 trigger CloudFront |
| Use case | Header manipulation, URL rewrite, A/B test nhẹ | Xử lý logic phức tạp, gọi service khác |

### 1.6 Lambda Layers & Extensions

- **Layers**: chia sẻ code/dependency giữa nhiều functions. Giải nén vào `/opt`. Max 5 layers.
- **Extensions**: code chạy song song với function, truy cập telemetry. Dùng cho observability (Datadog, Dynatrace...), secrets caching.

### 1.7 VPC-enabled Lambda

- Khi cần truy cập RDS/ElastiCache/resources trong VPC.
- Lambda tạo **ENI** trong subnet được chỉ định.
- **Hyperplane ENIs** (từ 2019): shared giữa các function → không còn cold start do ENI.
- Để Lambda trong VPC truy cập internet → cần **NAT Gateway** ở public subnet (KHÔNG gắn IGW trực tiếp vào Lambda được).

> ⚠️ **Bẫy:** Lambda trong VPC mà không có NAT → **KHÔNG gọi được AWS service** (trừ khi dùng **VPC Endpoints**).

### 1.8 Versions & Aliases

- **Version**: immutable snapshot (`$LATEST` là mutable).
- **Alias**: pointer tới version. Hỗ trợ **weighted routing** (canary deployment — ví dụ 90% v1, 10% v2).
- Alias **KHÔNG** trỏ được tới alias khác.

### 1.9 Event Source Mapping — Batch & Error handling

- **SQS Standard**: batch size tối đa 10,000. **Batch window** (MaximumBatchingWindowInSeconds).
- **SQS FIFO**: xử lý theo MessageGroupId, không đổi thứ tự trong group.
- **Kinesis/DynamoDB Streams**:
  - **ParallelizationFactor**: 1-10, tăng concurrency per shard.
  - **BisectBatchOnFunctionError**: chia đôi batch khi lỗi để xác định record lỗi.
  - **MaximumRetryAttempts**, **MaximumRecordAgeInSeconds**.
  - **On-failure destination**: SQS/SNS.

### 1.10 SnapStart (Java)

- Chỉ cho **Java 11+**. Snapshot execution environment → giảm cold start đáng kể (có thể đạt sub-second).
- Không dùng được chung với Provisioned Concurrency.
- **Miễn phí** nhưng snapshot có phí lưu trữ nhỏ.

---

## 2. Amazon EC2

### 2.1 Instance types (nhớ họ chữ cái)

| Họ | Use case |
|---|---|
| **T** (t3, t4g) | Burstable, general purpose, rẻ cho workload không đều |
| **M** (m5, m6i, m7g) | General purpose cân đối CPU/RAM |
| **C** (c5, c6i, c7g) | Compute-optimized — HPC, batch, gaming |
| **R** (r5, r6i) | Memory-optimized — in-memory DB, caching |
| **X** (x1, x2) | Extreme memory — SAP HANA |
| **I** (i3, i4i) | Storage-optimized — NoSQL, data warehouse (NVMe SSD) |
| **D / H** | HDD storage lớn |
| **P / G / Inf / Trn** | GPU / ML / inference / training |

- Chữ cái cuối: `g` = Graviton (ARM), `n` = enhanced networking, `d` = local NVMe, `a` = AMD EPYC, `i` = Intel.

### 2.2 Purchasing options

| Option | Khi nào dùng | Tiết kiệm |
|---|---|---|
| **On-Demand** | Workload ngắn hạn, không dự đoán được | 0% |
| **Reserved (Standard)** | Workload ổn định 1-3 năm | Tới 72% |
| **Reserved (Convertible)** | Cần đổi instance family | Tới 66% |
| **Savings Plans** | Commitment $/h, linh hoạt hơn RI | Tới 72% |
| **Spot** | Fault-tolerant (batch, CI, ML training) | Tới 90% |
| **Dedicated Host** | Compliance, BYOL (Windows, Oracle) | — |
| **Dedicated Instance** | Isolation cấp hardware | — |
| **Capacity Reservations** | Đảm bảo capacity trong AZ, KHÔNG giảm giá | — |

### 2.3 Spot Instances

- **Spot Request**: one-time hoặc persistent.
- **Spot Fleet**: mix Spot + On-Demand, nhiều instance types/AZ.
- **Spot Block**: đã **bị khai tử** — không còn trong đề thi mới.
- **Interruption notice**: 2 phút warning qua instance metadata (`http://169.254.169.254/latest/meta-data/spot/instance-action`).
- **Strategies**: `lowestPrice`, `diversified`, `capacityOptimized` (khuyến nghị — ít bị gián đoạn nhất), `priceCapacityOptimized` (mới, tốt nhất cho cost + stability).

### 2.4 User Data & Instance Metadata

- **User Data**: script chạy **1 lần duy nhất** khi instance khởi động lần đầu (chạy bằng root).
- **Instance Metadata Service (IMDS)**:
  - `http://169.254.169.254/latest/meta-data/` — không cần credentials, không ra ngoài network.
  - **IMDSv1** (query trực tiếp) — có thể bị SSRF attack.
  - **IMDSv2** (token-based, PUT để lấy token, GET với header `X-aws-ec2-metadata-token`) — **luôn ưu tiên**.

> 💡 **Bẫy thi:** "User data chạy mỗi khi instance reboot" → **SAI**. Chỉ chạy 1 lần đầu (trừ khi hack cloud-init).

### 2.5 Placement Groups

| Type | Mô tả | Use case |
|---|---|---|
| **Cluster** | Gom instance trong 1 rack, cùng AZ — **low latency** | HPC |
| **Spread** | Phân tán qua nhiều rack, max **7 instances/AZ** | Critical app cần isolation |
| **Partition** | Chia thành nhóm rack riêng (tới 7 partitions/AZ) | HDFS, Cassandra, Kafka |

### 2.6 EC2 Hibernate

- RAM lưu xuống **EBS root volume** (phải encrypted).
- Start lại từ trạng thái cũ — không mất process.
- Giới hạn: RAM < 150 GB, một số instance types không hỗ trợ.

---

## 3. Elastic Load Balancing (ELB)

### 3.1 Bốn loại ELB

| Loại | Layer | Protocol | Use case chính |
|---|---|---|---|
| **ALB** | 7 | HTTP/HTTPS/gRPC/WebSocket | Web app, microservice, path/host-based routing |
| **NLB** | 4 | TCP/UDP/TLS | Cực kỳ low-latency, millions req/s, **static IP** |
| **GLB** | 3 | IP | Deploy fleet 3rd-party appliances (firewall, IDS) |
| **CLB** | 4+7 | HTTP/HTTPS/TCP | **Legacy, tránh dùng** |

### 3.2 ALB — tính năng quan trọng

- **Routing**: path-based (`/api/*`), host-based (`api.example.com`), header-based, query-string-based, HTTP method.
- **Target types**: Instance, IP, Lambda, ALB (ALB-to-ALB).
- **Sticky session (cookie)**: duration-based, application-based.
- **Headers thêm vào**: `X-Forwarded-For` (client IP), `X-Forwarded-Proto`, `X-Forwarded-Port`.
- **Redirect HTTP→HTTPS** cấu hình trực tiếp trong listener rule.

### 3.3 NLB — tính năng quan trọng

- **Static IP per AZ** (hoặc gán **Elastic IP**).
- Preserve client source IP — KHÔNG cần `X-Forwarded-For`.
- Hỗ trợ **TCP, UDP, TLS passthrough**.
- Target types: Instance, IP, ALB.

### 3.4 Health Checks

- **Healthy/Unhealthy threshold**: số lần consecutive pass/fail.
- Protocol có thể khác traffic protocol (ví dụ traffic HTTPS nhưng health check HTTP).
- **Connection Draining (CLB)** / **Deregistration Delay (ALB/NLB)**: mặc định 300s, range 0-3600s.

---

## 4. Auto Scaling Groups (ASG)

### 4.1 Scaling policies

| Policy | Mô tả |
|---|---|
| **Target Tracking** | "Giữ CPU 40%" — AWS tự tính toán (**đơn giản nhất, khuyến nghị**) |
| **Step Scaling** | Scale theo bậc dựa trên CloudWatch alarm |
| **Simple Scaling** | Scale 1 bước rồi chờ cooldown |
| **Scheduled** | Theo thời gian cố định |
| **Predictive** | ML dự đoán → scale trước |

### 4.2 Cooldown & Warmup

- **Cooldown period**: mặc định 300s — ASG tạm dừng scaling action sau mỗi scale.
- **Instance warmup** (cho Target Tracking/Step): instance chưa được tính metric cho tới khi warmup xong.

### 4.3 Lifecycle Hooks

- Delay instance khi `Pending:Wait` hoặc `Terminating:Wait`.
- Dùng để run custom script, drain connection, upload log.
- Trigger tới **SNS/SQS/EventBridge**.

### 4.4 Termination policy

- Mặc định: chọn AZ có nhiều instance nhất → chọn instance có **Launch Template/Config cũ nhất** → chọn instance gần billing hour nhất.
- **Instance Protection**: bảo vệ khỏi scale-in.

> 💡 **Bẫy:** Termination protection (EC2) ≠ Scale-in protection (ASG). Termination protection KHÔNG ngăn ASG terminate.

### 4.5 Launch Template vs Launch Configuration

- **Launch Configuration**: cũ, immutable, không còn được khuyến nghị.
- **Launch Template**: mới, có versioning, hỗ trợ đầy đủ tính năng EC2 mới (mixed instances, T2/T3 unlimited, ...).

---

## 5. Amazon ECS (Elastic Container Service)

### 5.1 Launch Types

| | **EC2 Launch Type** | **Fargate** |
|---|---|---|
| Quản lý server | Bạn quản lý | Serverless — AWS quản |
| Scaling | Scale EC2 + scale task | Chỉ scale task |
| Pricing | Trả cho EC2 | Trả cho vCPU + RAM của task |
| Networking | bridge/host/awsvpc | **awsvpc only** |

### 5.2 Task Definition — các khái niệm

- **Task Definition**: JSON blueprint (image, CPU, memory, port, env, IAM role, volume...).
- **Task**: instance running của task definition.
- **Service**: đảm bảo số lượng task chạy, integrate với ELB.
- **Cluster**: logical grouping.

### 5.3 IAM Roles trong ECS — **Bẫy phổ biến**

Hai role riêng biệt:
1. **Task Execution Role**: dùng bởi **ECS agent** để pull image từ ECR, push log tới CloudWatch, lấy secrets từ SSM/Secrets Manager.
2. **Task Role**: dùng bởi **container của bạn** — để gọi AWS API (VD: ghi vào S3, DynamoDB).

> ⚠️ Đừng gán quyền ứng dụng vào Task Execution Role!

### 5.4 Networking modes (EC2 launch type)

- **bridge**: mặc định, NAT. Có thể dynamic host port (port 0).
- **host**: share network stack với host → port conflict.
- **awsvpc**: mỗi task có **ENI riêng + Security Group riêng** (bắt buộc với Fargate).
- **none**: không network.

### 5.5 Service Auto Scaling

- Dựa trên CPU, memory utilization, custom metric, ALB request count per target.
- **Target Tracking** dễ dùng nhất.

### 5.6 ECS Task Placement (chỉ EC2 launch type)

- **Placement Strategies**:
  - `binpack`: tận dụng tối đa CPU/memory (ít instance nhất) → tiết kiệm.
  - `random`: phân phối ngẫu nhiên.
  - `spread`: phân tán qua attribute (VD `attribute:ecs.availability-zone`).
- **Placement Constraints**:
  - `distinctInstance`: mỗi task trên 1 instance khác nhau.
  - `memberOf`: theo cluster query language.

### 5.7 ECS + Load Balancer — Dynamic Port Mapping

- Khi dùng ALB + ECS với bridge mode và port 0 → ECS tự gán random port, ALB tự route tới đúng port qua ENI.
- NLB cũng hỗ trợ, CLB thì KHÔNG.

### 5.8 ECS Rolling Update

- **minimumHealthyPercent** (mặc định 100%): số task tối thiểu phải khỏe trong lúc deploy.
- **maximumPercent** (mặc định 200%): số task tối đa được chạy trong lúc deploy.
- Ví dụ 4 task, min 50%, max 200% → có thể có 2-8 task lúc deploy.

### 5.9 ECS Deployment với CodeDeploy — Blue/Green

- Tạo task definition mới → CodeDeploy tạo replacement task set → test → shift traffic → terminate old.
- Traffic shifting: `Canary` (2 steps), `Linear` (đều đặn), `AllAtOnce`.

---

## 6. Amazon EKS (Elastic Kubernetes Service)

### 6.1 Các điểm cần nhớ

- **Control plane**: AWS quản lý, multi-AZ, trả phí $0.10/h per cluster.
- **Data plane (worker nodes)** 3 lựa chọn:
  - **Managed Node Groups** (EC2): AWS quản lý EC2 + autoscaling.
  - **Self-managed nodes**: bạn tự tạo EC2.
  - **Fargate**: serverless pods.
- **IAM Roles for Service Accounts (IRSA)**: gán IAM role cho K8s ServiceAccount qua OIDC — pod gọi AWS API an toàn.
- **EKS Anywhere**: chạy EKS on-prem.
- **EKS Distro**: distribution K8s open-source của AWS.

---

## 7. Elastic Beanstalk

### 7.1 Khái niệm

- PaaS cho web app — deploy nhanh, không cần quản infra.
- Support: Node.js, Java, Python, Ruby, PHP, Go, .NET, Docker (single/multi-container).
- **Components**: Application → Application Version → Environment.
- Mỗi environment có **tier**:
  - **Web Server tier**: ALB/NLB + ASG + EC2.
  - **Worker tier**: SQS + ASG + EC2.

### 7.2 Deployment options — **THUỘC LÒNG BẢNG NÀY**

| Option | Downtime | Deploy time | No DNS change | Rollback | Ghi chú |
|---|---|---|---|---|---|
| **All at once** | ✅ có | Nhanh nhất | ✅ | Redeploy | Dev only |
| **Rolling** | ❌ | Dài | ✅ | Manual | Giảm capacity tạm thời |
| **Rolling with additional batch** | ❌ | Dài hơn | ✅ | Manual | Giữ nguyên capacity |
| **Immutable** | ❌ | Lâu nhất | ✅ | Nhanh (xóa new ASG) | An toàn nhất in-place |
| **Blue/Green** | ❌ | — | ❌ (swap URL) | Swap lại | Env riêng biệt |
| **Traffic splitting** (canary) | ❌ | — | ✅ | Tự động khi alarm | Test % traffic |

### 7.3 Config files

- Thư mục `.ebextensions/` chứa file `.config` (YAML/JSON).
- `.platform/` (nền tảng Amazon Linux 2) để tùy chỉnh nginx, proc, hooks.

### 7.4 Tips

- **RDS trong Beanstalk environment**: tiện cho dev nhưng khi xóa env → xóa RDS. **Production → tạo RDS bên ngoài** và kết nối qua env var.
- **Cloning environment**: copy cấu hình sang env mới để test.
- **Managed Platform Updates**: auto update minor version.

---

## 8. AWS Batch

- Chạy batch job trên EC2/Fargate/Spot.
- **Khác Lambda**: không giới hạn 15 phút, dùng cho ETL/ML training/HPC.
- Components: Job Definition → Job Queue → Compute Environment.

> 💡 **So sánh nhanh:**
> - **Lambda**: event-driven, < 15 phút, stateless, sync/async.
> - **Batch**: long-running job, Docker-based, dependency giữa job.
> - **ECS/Fargate**: long-running service hoặc task tùy ý.
> - **Step Functions**: orchestrate nhiều step có logic phức tạp.

---

## 9. Lightsail

- VPS đơn giản, giá cố định/tháng.
- Phù hợp: dev học, small website, low-traffic.
- **Không xuất hiện nhiều trong DVA-C02** — chỉ cần biết sơ.

---

## 🎯 Tip ôn thi Phần 1

1. **Lambda là ngôi sao** — đọc kỹ tất cả phần trên, đặc biệt:
   - Giới hạn (memory, timeout, package size).
   - VPC + NAT (bẫy phổ biến).
   - Async retry behavior + DLQ/Destinations.
   - Concurrency: Reserved vs Provisioned.
   - Event source mapping (SQS/Kinesis).

2. **ECS Task Role vs Task Execution Role** — đề rất thích hỏi.

3. **Beanstalk deployment options** — học thuộc bảng. Câu hỏi: "Zero downtime + rollback nhanh nhất?" → **Immutable** hoặc **Blue/Green**.

4. **ASG Launch Template** thay Launch Configuration — câu hỏi nào thấy dùng Launch Config thường là đáp án SAI.

5. **Placement Groups**: Cluster cho HPC, Spread cho HA, Partition cho big data.

6. **IMDSv2 luôn được ưu tiên hơn IMDSv1** vì security.

7. **Spot Strategy**: `capacityOptimized` (ít gián đoạn) hoặc `priceCapacityOptimized` (cost + stability).

8. **CloudFront Functions vs Lambda@Edge**: nếu chỉ cần xử lý header/URL rewrite đơn giản → CloudFront Functions (rẻ hơn, nhanh hơn).

---

**→ Sẵn sàng tiếp tục với Phần 2 (Storage & Database)?**
