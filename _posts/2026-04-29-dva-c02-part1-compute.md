---
title: "DVA-C02 Phần 1: Compute & Containers"
author: thanhnv1808
date: 2026-04-29 08:00:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, lambda, ec2, ecs, eks, elastic-beanstalk, certification]
description: "Lambda, EC2, ELB, ASG, ECS, EKS, Elastic Beanstalk — tất cả compute services cần thuộc lòng cho kỳ thi AWS DVA-C02."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> **Phần 1 (file này)** → [Phần 2: Storage & DB](/posts/dva-c02-part2-storage-database) → [Phần 3: App Integration](/posts/dva-c02-part3-app-integration) → [Phần 4: Security](/posts/dva-c02-part4-security) → [Phần 5: Deploy & Monitor](/posts/dva-c02-part5-deploy-monitor) → [Phần 6: Networking + Tips thi](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }

---

## 1. AWS Lambda ⭐ (Chiếm nhiều câu nhất)

### 1.1 Giới hạn — PHẢI THUỘC LÒNG

| Thuộc tính | Giới hạn |
|---|---|
| **Memory** | 128 MB → 10,240 MB (10 GB), bước nhảy 1 MB |
| **vCPU** | Tỷ lệ thuận với memory. Đạt **1 vCPU tại 1,769 MB**, tối đa ~6 vCPU ở 10 GB |
| **Timeout tối đa** | **15 phút** (900 giây) |
| **Deployment package — zip trực tiếp** | 50 MB |
| **Deployment package — zip qua S3** | 250 MB (đã giải nén) |
| **Container image** | **10 GB** |
| **/tmp storage** | 512 MB mặc định, có thể cấu hình tới **10 GB** |
| **Environment variables** | Tổng size 4 KB |
| **Concurrent executions (account)** | 1,000 mặc định (soft limit, xin tăng được) |
| **Invocation payload — sync** | 6 MB request + response |
| **Invocation payload — async** | 256 KB |
| **Layers** | Tối đa **5 layers/function**, mỗi layer ≤ 250 MB (đã giải nén, cộng cả function) |

> **Trick:** Đề thường hỏi "function xử lý file 8GB, chạy 2 phút" → KHÔNG dùng Lambda được (do /tmp chỉ 10GB max, hoặc do timeout nếu >15 phút). Dùng **Fargate** hoặc **Batch**.
{: .prompt-tip }

---

### 1.2 Invocation Types

| Loại | Cơ chế | Ví dụ trigger |
|---|---|---|
| **Synchronous** | Caller chờ kết quả. Lỗi → caller tự retry | CLI, API Gateway, ALB, Cognito |
| **Asynchronous** | Lambda **tự retry 2 lần** (tổng 3). Thất bại → DLQ/Destinations | S3, SNS, EventBridge |
| **Event source mapping (Poll-based)** | Lambda tự poll | SQS, Kinesis, DynamoDB Streams, Kafka |

> **Bẫy:** DLQ chỉ áp dụng cho **async invocation**. Với SQS event source → thông điệp quay lại queue → vào **source queue's DLQ** (cấu hình ở SQS, không phải Lambda).
{: .prompt-warning }

---

### 1.3 Destinations vs DLQ

| | DLQ | Destinations |
|---|---|---|
| Xuất hiện | Cũ | Mới hơn, **khuyến nghị** |
| Targets | SQS, SNS | SQS, SNS, Lambda, EventBridge |
| On success | ❌ | ✅ |
| On failure | ✅ | ✅ |
| Metadata đầy đủ | ❌ | ✅ |

---

### 1.4 Concurrency

- **Reserved concurrency**: dành riêng slot cho function → đảm bảo có slot & giới hạn tối đa (throttle bảo vệ downstream như RDS).
- **Provisioned concurrency**: khởi tạo sẵn execution environments → **loại bỏ cold start**. Dùng với alias/version, **KHÔNG dùng với `$LATEST`**.
- **Unreserved**: tổng còn lại phải ≥ **100**.

> **Giải pháp cold start (theo thứ tự ưu tiên):**
> 1. **Provisioned Concurrency** (hiệu quả nhất)
> 2. Tăng memory (khởi tạo nhanh hơn)
> 3. Ngôn ngữ khởi động nhanh: Node.js, Python > Java, .NET
> 4. Giảm kích thước package
> 5. Init SDK clients **bên ngoài handler** (tái dùng giữa các invocation)
{: .prompt-tip }

---

### 1.5 Lambda@Edge vs CloudFront Functions

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript (nhẹ) | Node.js, Python |
| Memory | Rất nhỏ | Tới 10 GB |
| Timeout | **< 1 ms** | 5s (viewer), 30s (origin) |
| Triggers | Viewer request/response | Tất cả 4 trigger CloudFront |
| Use case | Header manipulation, URL rewrite, A/B test nhẹ | Xử lý logic phức tạp, gọi service khác |

---

### 1.6 Lambda Layers & Extensions

- **Layers**: chia sẻ code/dependency giữa nhiều functions. Giải nén vào `/opt`. Max 5 layers.
- **Extensions**: code chạy song song với function, truy cập telemetry. Dùng cho observability (Datadog, Dynatrace...), secrets caching.

---

### 1.7 VPC-enabled Lambda

- Lambda trong VPC cần **NAT Gateway** ở public subnet để ra Internet — **KHÔNG gắn IGW trực tiếp**.
- Từ 2019: Lambda dùng **Hyperplane ENIs** (shared) → không còn cold start do ENI.

> **Bẫy:** Lambda trong VPC mà không có NAT → **KHÔNG gọi được AWS service** (trừ khi dùng **VPC Endpoints**).
{: .prompt-warning }

---

### 1.8 Versions & Aliases

- **Version**: immutable snapshot. `$LATEST` là mutable.
- **Alias**: pointer tới version. Hỗ trợ **weighted routing** (canary: 90% v1, 10% v2).
- Alias **KHÔNG** trỏ được tới alias khác.

---

### 1.9 Event Source Mapping — Batch & Error handling

| Source | Đặc điểm |
|---|---|
| **SQS Standard** | Batch size tối đa 10,000. Batch window (MaximumBatchingWindowInSeconds) |
| **SQS FIFO** | Xử lý theo MessageGroupId, giữ thứ tự trong group |
| **Kinesis / DynamoDB Streams** | ParallelizationFactor (1-10), BisectBatchOnFunctionError, MaximumRetryAttempts, On-failure destination |

---

### 1.10 SnapStart (Java)

- Chỉ cho **Java 11+**. Snapshot execution environment → giảm cold start đáng kể.
- **Không** dùng chung với Provisioned Concurrency.
- **Miễn phí** (snapshot có phí lưu trữ nhỏ).

---

## 2. Amazon EC2

### 2.1 Instance Types

| Họ | Use case |
|---|---|
| **T** (t3, t4g) | Burstable, general purpose, workload không đều |
| **M** (m5, m6i) | General purpose cân đối CPU/RAM |
| **C** (c5, c6i) | Compute-optimized — HPC, batch, gaming |
| **R** (r5, r6i) | Memory-optimized — in-memory DB, caching |
| **X** (x1, x2) | Extreme memory — SAP HANA |
| **I** (i3, i4i) | Storage-optimized — NoSQL, data warehouse (NVMe SSD) |
| **P / G / Inf / Trn** | GPU / ML / inference / training |

Chữ cái cuối: `g` = Graviton (ARM), `n` = enhanced networking, `d` = local NVMe, `a` = AMD EPYC, `i` = Intel.

---

### 2.2 Purchasing Options

| Option | Khi nào dùng | Tiết kiệm |
|---|---|---|
| **On-Demand** | Workload ngắn hạn, không dự đoán | 0% |
| **Reserved (Standard)** | Workload ổn định 1-3 năm | Tới 72% |
| **Reserved (Convertible)** | Cần đổi instance family | Tới 66% |
| **Savings Plans** | Commitment $/h, linh hoạt hơn RI | Tới 72% |
| **Spot** | Fault-tolerant (batch, CI, ML training) | Tới 90% |
| **Dedicated Host** | Compliance, BYOL (Windows, Oracle) | — |
| **Dedicated Instance** | Isolation cấp hardware | — |
| **Capacity Reservations** | Đảm bảo capacity trong AZ, **KHÔNG giảm giá** | — |

---

### 2.3 Spot Instances

- **Spot Fleet**: mix Spot + On-Demand, nhiều instance types/AZ.
- **Interruption notice**: 2 phút warning qua instance metadata.
- **Strategies**: `lowestPrice`, `diversified`, `capacityOptimized` (ít gián đoạn nhất), `priceCapacityOptimized` (mới, tốt nhất).

---

### 2.4 User Data & IMDS

- **User Data**: script chạy **1 lần duy nhất** khi instance khởi động lần đầu.
- **IMDSv1** → có thể bị SSRF attack.
- **IMDSv2** (token-based) → **luôn ưu tiên**.

> **Bẫy thi:** "User data chạy mỗi khi instance reboot" → **SAI**. Chỉ chạy 1 lần đầu.
{: .prompt-warning }

---

### 2.5 Placement Groups

| Type | Mô tả | Use case |
|---|---|---|
| **Cluster** | Gom instance trong 1 rack, cùng AZ — **low latency** | HPC |
| **Spread** | Phân tán qua nhiều rack, max **7 instances/AZ** | Critical app cần isolation |
| **Partition** | Chia thành nhóm rack riêng (tới 7 partitions/AZ) | HDFS, Cassandra, Kafka |

---

### 2.6 EC2 Hibernate

- RAM lưu xuống **EBS root volume** (phải encrypted).
- Start lại từ trạng thái cũ — không mất process.
- Giới hạn: RAM < 150 GB, một số instance types không hỗ trợ.

---

## 3. Elastic Load Balancing (ELB)

### 3.1 Bốn loại ELB

| Loại | Layer | Protocol | Use case |
|---|---|---|---|
| **ALB** | 7 | HTTP/HTTPS/gRPC/WebSocket | Web app, microservice, path/host-based routing |
| **NLB** | 4 | TCP/UDP/TLS | Low-latency, millions req/s, **static IP** |
| **GLB** | 3 | IP | Deploy fleet 3rd-party appliances (firewall, IDS) |
| **CLB** | 4+7 | HTTP/HTTPS/TCP | **Legacy, tránh dùng** |

---

### 3.2 ALB — tính năng quan trọng

- **Routing**: path-based, host-based, header-based, query-string, HTTP method.
- **Target types**: Instance, IP, Lambda, ALB.
- **Sticky session (cookie)**: duration-based, application-based.
- Headers thêm vào: `X-Forwarded-For`, `X-Forwarded-Proto`, `X-Forwarded-Port`.

---

### 3.3 NLB — tính năng quan trọng

- **Static IP per AZ** (hoặc gán **Elastic IP**).
- Preserve client source IP — KHÔNG cần `X-Forwarded-For`.
- Target types: Instance, IP, **ALB**.

---

### 3.4 Health Checks

- **Connection Draining (CLB)** / **Deregistration Delay (ALB/NLB)**: mặc định **300s**, range 0-3600s.

---

## 4. Auto Scaling Groups (ASG)

### 4.1 Scaling Policies

| Policy | Mô tả |
|---|---|
| **Target Tracking** | "Giữ CPU 40%" — AWS tự tính toán (**đơn giản nhất, khuyến nghị**) |
| **Step Scaling** | Scale theo bậc dựa trên CloudWatch alarm |
| **Simple Scaling** | Scale 1 bước rồi chờ cooldown |
| **Scheduled** | Theo thời gian cố định |
| **Predictive** | ML dự đoán → scale trước |

---

### 4.2 Cooldown & Warmup

- **Cooldown period**: mặc định **300s** — ASG tạm dừng sau mỗi scale.
- **Instance warmup**: instance chưa tính metric cho tới khi warmup xong.

---

### 4.3 Lifecycle Hooks

- Delay instance khi `Pending:Wait` hoặc `Terminating:Wait`.
- Trigger tới **SNS/SQS/EventBridge** để run custom script, drain connection, upload log.

---

### 4.4 Termination Policy

- Mặc định: AZ có nhiều instance nhất → **Launch Template/Config cũ nhất** → instance gần billing hour nhất.
- **Instance Protection**: bảo vệ khỏi scale-in.

> **Bẫy:** Termination protection (EC2) ≠ Scale-in protection (ASG). Termination protection KHÔNG ngăn ASG terminate.
{: .prompt-warning }

---

### 4.5 Launch Template vs Launch Configuration

| | Launch Configuration | Launch Template |
|---|---|---|
| Trạng thái | Cũ, immutable, không khuyến nghị | Mới, có versioning |
| Hỗ trợ tính năng mới | ❌ | ✅ (mixed instances, T2/T3 unlimited...) |

---

## 5. Amazon ECS (Elastic Container Service)

### 5.1 Launch Types

| | **EC2 Launch Type** | **Fargate** |
|---|---|---|
| Quản lý server | Bạn quản lý | Serverless — AWS quản |
| Scaling | Scale EC2 + scale task | Chỉ scale task |
| Pricing | Trả cho EC2 | Trả cho vCPU + RAM của task |
| Networking | bridge/host/awsvpc | **awsvpc only** |

---

### 5.2 Task Definition — Khái niệm

| Khái niệm | Mô tả |
|---|---|
| **Task Definition** | JSON blueprint (image, CPU, memory, port, env, IAM role, volume...) |
| **Task** | Instance running của task definition |
| **Service** | Đảm bảo số lượng task chạy, integrate với ELB |
| **Cluster** | Logical grouping |

---

### 5.3 IAM Roles trong ECS — **Bẫy phổ biến**

| Role | Dùng bởi | Mục đích |
|---|---|---|
| **Task Execution Role** | ECS agent | Pull image từ ECR, push log CloudWatch, lấy secrets SSM/Secrets Manager |
| **Task Role** | Container của bạn | Gọi AWS API (S3, DynamoDB...) |

> **Đừng gán quyền ứng dụng vào Task Execution Role!**
{: .prompt-danger }

---

### 5.4 Networking Modes (EC2 launch type)

| Mode | Mô tả |
|---|---|
| **bridge** | NAT. Có thể dynamic host port (port 0) |
| **host** | Share network stack với host → port conflict |
| **awsvpc** | Mỗi task có ENI riêng + Security Group riêng (**bắt buộc với Fargate**) |
| **none** | Không network |

---

### 5.5 ECS Task Placement (chỉ EC2 launch type)

**Placement Strategies:**
- `binpack`: tận dụng tối đa CPU/memory (ít instance nhất) → tiết kiệm.
- `random`: phân phối ngẫu nhiên.
- `spread`: phân tán qua attribute (VD `attribute:ecs.availability-zone`).

**Placement Constraints:**
- `distinctInstance`: mỗi task trên 1 instance khác nhau.
- `memberOf`: theo cluster query language.

---

### 5.6 ECS Rolling Update

- **minimumHealthyPercent** (mặc định 100%): số task tối thiểu phải khỏe trong lúc deploy.
- **maximumPercent** (mặc định 200%): số task tối đa được chạy trong lúc deploy.

---

### 5.7 ECS Deployment với CodeDeploy — Blue/Green

Traffic shifting: `Canary` (2 steps) → `Linear` (đều đặn) → `AllAtOnce`.

---

## 6. Amazon EKS (Elastic Kubernetes Service)

### 6.1 Điểm cần nhớ

- **Control plane**: AWS quản lý, multi-AZ, tính phí $0.10/h per cluster.
- **Data plane** có 3 lựa chọn:

| Option | Mô tả |
|---|---|
| **Managed Node Groups** | AWS quản lý EC2 + autoscaling |
| **Self-managed nodes** | Bạn tự tạo EC2 |
| **Fargate** | Serverless pods |

- **IRSA** (IAM Roles for Service Accounts): gán IAM role cho K8s ServiceAccount qua OIDC — pod gọi AWS API an toàn.
- **EKS Anywhere**: chạy EKS on-prem.

---

## 7. Elastic Beanstalk

### 7.1 Khái niệm

- PaaS cho web app — deploy nhanh, không cần quản infra.
- Hỗ trợ: Node.js, Java, Python, Ruby, PHP, Go, .NET, Docker.
- **Components**: Application → Application Version → Environment.
- **Tiers**: Web Server (ALB/NLB + ASG + EC2) & Worker (SQS + ASG + EC2).

---

### 7.2 Deployment Options — THUỘC LÒNG

| Option | Downtime | Deploy time | Rollback | Ghi chú |
|---|---|---|---|---|
| **All at once** | ✅ có | Nhanh nhất | Redeploy | Dev only |
| **Rolling** | ❌ | Dài | Manual | Giảm capacity tạm thời |
| **Rolling with additional batch** | ❌ | Dài hơn | Manual | Giữ nguyên capacity |
| **Immutable** | ❌ | Lâu nhất | Nhanh (xóa new ASG) | An toàn nhất |
| **Blue/Green** | ❌ | — | Swap URL lại | Env riêng biệt |
| **Traffic splitting** (canary) | ❌ | — | Tự động khi alarm | Test % traffic |

> **Câu hỏi thi:** "Zero downtime + rollback nhanh nhất?" → **Immutable** hoặc **Blue/Green**.
{: .prompt-tip }

---

### 7.3 Tips

- **RDS trong Beanstalk environment**: tiện cho dev nhưng khi xóa env → xóa RDS. **Production → tạo RDS bên ngoài**.
- **Config files**: `.ebextensions/` chứa file `.config` (YAML/JSON).

---

## 8. AWS Batch

- Chạy batch job trên EC2/Fargate/Spot.
- Components: Job Definition → Job Queue → Compute Environment.

| Service | Khi nào dùng |
|---|---|
| **Lambda** | Event-driven, < 15 phút, stateless |
| **Batch** | Long-running job, Docker-based, job dependencies |
| **ECS/Fargate** | Long-running service hoặc task tùy ý |
| **Step Functions** | Orchestrate nhiều step có logic phức tạp |

---

## 🎯 Tip Ôn Thi Phần 1

1. **Lambda là ngôi sao**: giới hạn, VPC + NAT, async retry, concurrency, event source mapping.
2. **ECS Task Role vs Task Execution Role** — đề rất thích hỏi.
3. **Beanstalk deployment options**: học thuộc bảng.
4. **ASG Launch Template** thay Launch Configuration — câu hỏi nào dùng Launch Config thường là đáp án SAI.
5. **Placement Groups**: Cluster → HPC, Spread → HA, Partition → big data.
6. **IMDSv2 luôn ưu tiên** hơn IMDSv1 vì security.
7. **Spot Strategy**: `capacityOptimized` hoặc `priceCapacityOptimized`.
8. **CloudFront Functions vs Lambda@Edge**: xử lý header/URL đơn giản → CloudFront Functions (rẻ hơn, nhanh hơn).

---

> **Tiếp theo:** [Phần 2: Storage & Database (S3, EBS, EFS, DynamoDB, RDS, Aurora, ElastiCache)](/posts/dva-c02-part2-storage-database)
{: .prompt-info }
