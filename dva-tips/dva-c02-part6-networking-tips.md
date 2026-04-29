# DVA-C02 — Phần 6: Networking, Edge & Tổng hợp tips thi (KẾT THÚC)

> **Chuỗi tài liệu:** Phần 1 (Compute) → Phần 2 (Storage & DB) → Phần 3 (App Integration) → Phần 4 (Security) → Phần 5 (Deploy & Monitor) → **Phần 6 (Networking + Tips thi)**.
>
> Phần này gộp: Networking/Edge (ít nặng trong DVA so với SAA, nhưng vẫn có câu), **decision trees tổng hợp**, **bẫy thường gặp xuyên các phần**, **thủ thuật làm bài**, và **checklist ôn cuối cùng**.

---

## 1. Amazon VPC — biết đủ dùng

### 1.1 Các thành phần chính

| Component | Mục đích |
|---|---|
| **VPC** | Virtual network, 1 region, CIDR từ `/16` đến `/28` |
| **Subnet** | Chia VPC theo AZ. Public (có route đến IGW) hoặc Private |
| **Route Table** | Routing rule cho subnet |
| **Internet Gateway (IGW)** | Out-in traffic Internet, gắn 1 VPC |
| **NAT Gateway** | Private subnet → Internet (outbound only). Managed, HA trong AZ |
| **NAT Instance** | Legacy, tự quản EC2. Không còn khuyến nghị |
| **Security Group** | **Stateful**, attach vào ENI. Allow only |
| **NACL** | **Stateless**, attach vào subnet. Allow & Deny, đánh số thứ tự |
| **VPC Peering** | 1-1 giữa 2 VPC, KHÔNG transitive |
| **Transit Gateway** | Hub-spoke, nhiều VPC + on-prem |
| **VPC Endpoint (Gateway)** | S3, DynamoDB — qua route table, miễn phí |
| **VPC Endpoint (Interface/PrivateLink)** | Hầu hết AWS service — qua ENI + DNS, tốn phí |

### 1.2 Security Group vs NACL — **BẢNG SO SÁNH KINH ĐIỂN**

| | **Security Group** | **NACL** |
|---|---|---|
| Level | ENI/Instance | Subnet |
| State | **Stateful** (return traffic tự động allow) | **Stateless** (phải allow cả in & out) |
| Rules | Only **Allow** | Allow & Deny |
| Rule order | Tất cả rules evaluate | Theo **rule number** (ascending) |
| Default | Allow all outbound, deny all inbound | Default VPC: allow all. Custom: deny all |

> ⚠️ **Bẫy kinh điển:** "Ping EC2 không được, Security Group đã mở ICMP in-bound?" → Check **NACL outbound** cho ephemeral ports (1024-65535).

### 1.3 VPC Endpoints — khi nào dùng

- **Gateway Endpoint (S3, DynamoDB)**: miễn phí, qua route table. Lambda trong VPC → S3/DynamoDB mà không qua NAT.
- **Interface Endpoint (PrivateLink)**: ENI + DNS private, tốn tiền theo giờ + GB. Dùng cho hầu hết service khác (SQS, SNS, KMS, CloudWatch...).

> 💡 **Tip:** Lambda/EC2 trong private subnet muốn gọi AWS service **mà không cần NAT** → VPC Endpoint.

### 1.4 Bastion Host & Systems Manager Session Manager

- **Bastion Host**: EC2 ở public subnet, SSH/RDP qua đó.
- **SSM Session Manager (mới hơn)**: không cần bastion, không cần SSH port mở, không cần key pair. Session log vào S3/CloudWatch. **Luôn ưu tiên** over bastion.

### 1.5 VPC Flow Logs

- Capture metadata network traffic (IP, port, protocol, accept/reject).
- Xuất ra **CloudWatch Logs, S3, Kinesis Firehose**.
- KHÔNG log payload, chỉ header.

---

## 2. Amazon Route 53 — ⭐ biết các routing policy

### 2.1 Record types cần biết

- `A` (IPv4), `AAAA` (IPv6), `CNAME`, `ALIAS`, `MX`, `TXT`, `NS`, `SOA`.
- **CNAME không cho root domain** (`example.com`) → dùng **ALIAS** (AWS-specific, trỏ tới AWS resource).
- **ALIAS miễn phí** (không tính per-query), resolve nhanh.

### 2.2 Routing Policies — THUỘC LÒNG

| Policy | Mục đích |
|---|---|
| **Simple** | 1 record, nhiều value → R53 random 1 |
| **Weighted** | % traffic chia theo weight (VD: 70% → server A, 30% → B) — canary |
| **Latency-based** | Trả record có latency thấp nhất từ client |
| **Failover** | Primary + Secondary + health check → auto switch |
| **Geolocation** | Theo country/continent của client (không phải latency) |
| **Geoproximity** (qua Traffic Flow) | Theo khoảng cách + bias |
| **Multi-value answer** | Tới 8 healthy records random — poor-man's LB |

### 2.3 Health Checks

- Monitor endpoint (HTTP/HTTPS/TCP).
- Monitor CloudWatch alarm.
- Monitor health check khác (calculated health check).
- Required cho **Failover policy**.

### 2.4 TTL

- Thời gian cache record ở resolver (DNS, client).
- Thấp (VD 60s) → update nhanh, cost cao hơn.
- Cao (VD 24h) → tiết kiệm, update chậm.
- **ALIAS record có TTL cố định theo target**.

---

## 3. Amazon CloudFront — ⭐ QUAN TRỌNG

### 3.1 Khái niệm

- CDN toàn cầu với 400+ edge locations.
- Origin types: **S3, ALB, EC2, MediaPackage, HTTP server (on-prem)**.

### 3.2 Caching behavior

- **Distribution** → **Behaviors** theo path pattern (VD `/static/*` behavior riêng).
- **Cache Policy**: TTL, các key cache (query string, header, cookie).
- **Origin Request Policy**: headers/cookies/query gửi đến origin.
- **Response Headers Policy**: custom response headers (CORS, security).

### 3.3 Cache Invalidation

- Invalidate theo path (`/images/*`) — tốn phí sau 1000 free/tháng.
- Hoặc đổi versioning (VD `image-v2.jpg`) — khuyến nghị hơn.

### 3.4 Origin Access Identity (OAI) & Origin Access Control (OAC)

- Bucket S3 không public → chỉ CloudFront truy cập được.
- **OAC (mới, 2022+)**: SigV4, tốt hơn OAI (SSE-KMS, dynamic request...).
- OAI đang deprecated — chọn OAC cho project mới.

### 3.5 Signed URLs vs Signed Cookies

| | **Signed URL** | **Signed Cookies** |
|---|---|---|
| Scope | 1 file | Nhiều file, domain-wide |
| Use case | Download link cá nhân | Premium content (streaming) |

### 3.6 Functions tại Edge

- **CloudFront Functions** (JS, <1ms, viewer request/response only).
- **Lambda@Edge** (Node/Python, 5-30s, 4 trigger — viewer req/res, origin req/res).
- Đã so sánh ở Phần 1.

### 3.7 Geo Restriction

- Whitelist/blacklist country access.

### 3.8 Field-level encryption

- Encrypt 1 số field (VD: credit card) tại edge → chỉ service backend có private key mới decrypt được.

---

## 4. AWS Global Accelerator

- 2 **static Anycast IP** + Edge network của AWS.
- Route traffic qua backbone AWS thay vì Internet → latency thấp, packet loss ít.
- **Khác CloudFront**: Global Accelerator không cache, tốt cho non-HTTP (TCP/UDP), static IP, gaming/IoT.

> 💡 **Decision:**
> - Static content, cache-able, HTTP/HTTPS → **CloudFront**.
> - Non-HTTP (TCP/UDP), cần static IP, low-latency connection → **Global Accelerator**.
> - Cả 2 đều giảm latency, có thể dùng song song (GA trước ALB).

---

## 5. AWS Direct Connect & VPN (biết sơ cho DVA)

- **VPN**: IPSec, qua Internet, encrypted.
- **Direct Connect**: dedicated fiber, on-prem → AWS. Không qua Internet, thấp latency, cao bandwidth, vài tuần setup.
- **DVA-C02 hầu như không hỏi sâu** — biết sơ khái niệm.

---

# 🎯 PHẦN II: TỔNG HỢP TIPS THI DVA-C02

---

## 6. Cấu trúc kỳ thi DVA-C02

- **65 câu**: ~50 câu tính điểm + ~15 câu không tính (unscored, để test).
- **130 phút**.
- **Pass score: 720/1000**.
- **Đa lựa chọn** (1 đáp án đúng) và **đa phản hồi** (2-3 đáp án đúng trong 5-6 options).

### Tỷ lệ các domain (theo AWS official):

| Domain | Tỷ lệ |
|---|---|
| **Domain 1: Development with AWS Services** | 32% |
| **Domain 2: Security** | 26% |
| **Domain 3: Deployment** | 24% |
| **Domain 4: Troubleshooting & Optimization** | 18% |

---

## 7. Decision Trees tổng hợp — "Nếu đề hỏi X, chọn Y"

### 7.1 Khi nào dùng dịch vụ compute nào?

```
Cần chạy code:
├── Event-driven, < 15 phút, stateless?
│   └── Lambda
├── Long-running container?
│   ├── Quản server → ECS (EC2)
│   └── Serverless → Fargate hoặc EKS (Fargate)
├── K8s ecosystem?
│   └── EKS
├── Batch job dài?
│   └── AWS Batch
├── Đã có codebase + muốn PaaS?
│   └── Elastic Beanstalk
└── Full control VM?
    └── EC2
```

### 7.2 Khi nào dùng dịch vụ database nào?

```
Dữ liệu:
├── Quan hệ (SQL)?
│   ├── Full managed + high-perf → Aurora
│   ├── Standard → RDS
│   └── Data warehouse (OLAP, analytics) → Redshift
├── NoSQL key-value / document?
│   ├── Ultra-low latency → DynamoDB (+ DAX nếu cần μs)
│   └── MongoDB-compatible → DocumentDB
├── Graph?
│   └── Neptune
├── Ledger / audit trail?
│   └── QLDB
├── Time-series?
│   └── Timestream
├── In-memory cache?
│   ├── Redis ecosystem, HA → ElastiCache Redis
│   ├── Simple caching → ElastiCache Memcached
│   └── Cache DynamoDB specifically → DAX
└── Search/log analytics?
    └── OpenSearch
```

### 7.3 Khi nào dùng dịch vụ messaging nào?

```
Message pattern:
├── Queue (1 sender → 1 consumer)?
│   ├── Cần order chính xác → SQS FIFO
│   └── Throughput cao, eventual order → SQS Standard
├── Pub/sub (1 sender → N consumer)?
│   ├── AWS services + HTTP/Email/SMS → SNS
│   └── Event routing với filter mạnh, cron → EventBridge
├── Fan-out (1 event → N xử lý độc lập)?
│   └── SNS → SQS (1 queue/consumer)
├── Real-time streaming, retention, replay?
│   ├── Custom processing → Kinesis Data Streams
│   └── Chỉ cần load vào S3/Redshift/... → Kinesis Firehose
├── Kafka-compatible migration?
│   └── MSK
├── AMQP/MQTT/JMS (legacy migration)?
│   └── Amazon MQ
└── Workflow orchestration (multi-step logic)?
    ├── Long-running, stateful → Step Functions Standard
    └── Short, high-volume → Step Functions Express
```

### 7.4 Khi nào dùng dịch vụ secret/config nào?

```
Store sensitive data:
├── Cần auto-rotation (VD: DB password)?
│   └── Secrets Manager
├── Config thông thường (env var, URL)?
│   ├── Feature flag, runtime config → AppConfig
│   └── Key-value đơn giản, miễn phí → SSM Parameter Store
└── Encrypted files / arbitrary data > 64 KB?
    └── S3 + KMS encryption
```

### 7.5 Khi nào dùng dịch vụ bảo mật/audit nào?

```
Mục đích:
├── Threat detection ML-based (bitcoin mining, compromise)?
│   └── GuardDuty
├── Vulnerability scan (EC2, ECR, Lambda)?
│   └── Inspector
├── PII trong S3?
│   └── Macie
├── API call audit (who did what when)?
│   └── CloudTrail
├── Resource config compliance (VD: S3 phải encrypted)?
│   └── AWS Config
├── DDoS protection?
│   ├── Free L3/L4 → Shield Standard
│   └── Advanced L7 → Shield Advanced + WAF
├── Layer 7 filter (SQLi, XSS, bot)?
│   └── WAF (tại CloudFront/ALB/API GW/AppSync)
└── Aggregate findings?
    └── Security Hub
```

---

## 8. TOP 30 BẪY KINH ĐIỂN TRONG DVA-C02

### Compute & Container

1. **Lambda timeout 15 phút** — workload > 15 phút → Fargate/Batch.
2. **Lambda trong VPC không có NAT** → không gọi Internet/AWS service (trừ khi có VPC Endpoint).
3. **Lambda async retry 2 lần** (tổng 3 lần) với exponential backoff.
4. **ECS Task Execution Role** (pull image, log) ≠ **Task Role** (app API calls).
5. **Beanstalk `Immutable` deployment** cho zero-downtime + rollback nhanh.
6. **User Data chỉ chạy 1 lần đầu** (không phải mỗi reboot).
7. **IMDSv2 luôn thay IMDSv1** (chống SSRF).

### Storage & Database

8. **S3 strong consistency** (từ 12/2020) — đừng bị lừa bởi tài liệu cũ nói eventual.
9. **S3 Glacier Instant Retrieval** cho archive + truy cập ms.
10. **S3 Bucket Keys** để giảm 99% KMS call (chống throttle).
11. **DynamoDB GSI throttle → main table write throttle** (LSI không vậy).
12. **DynamoDB FilterExpression & ProjectionExpression KHÔNG giảm RCU**.
13. **DynamoDB Strongly consistent read tốn gấp đôi RCU** so với eventually.
14. **RDS Multi-AZ KHÔNG scale read** — chỉ HA. Dùng Read Replica.
15. **EBS volume bị khóa AZ** — move AZ → snapshot + restore.

### App Integration

16. **SQS Visibility Timeout quá ngắn → duplicate processing**.
17. **SQS Long polling (20s) để tiết kiệm cost** và giảm empty response.
18. **SQS FIFO: 300 TPS** (3000 với batch). Cần cao hơn → High Throughput mode.
19. **Kinesis Enhanced Fan-out** khi nhiều consumer đọc cùng stream.
20. **API Gateway timeout 29s cứng** — long-running phải async.
21. **Cognito User Pool Authorizer** chỉ authenticate (không authorize) — logic quyền ở Lambda.

### Security

22. **IAM: Explicit Deny luôn thắng Explicit Allow**.
23. **KMS Key Policy là bắt buộc** — IAM admin không tự động access được CMK.
24. **KMS Encrypt API giới hạn 4 KB** → data lớn dùng Envelope Encryption + GenerateDataKey.
25. **ACM chỉ dùng cho AWS-managed services** (ALB, CloudFront, API GW). EC2 Nginx self-host KHÔNG xài được ACM.
26. **Secrets Manager cần rotation** / **Parameter Store cho config không rotate**.

### Deploy & Monitor

27. **CloudFormation DeletionPolicy: Retain** để giữ resource khi xóa stack.
28. **CodeDeploy Lambda phải dùng alias**, không deploy vào $LATEST.
29. **CloudWatch Metric Filter KHÔNG retroactive** — chỉ apply từ lúc tạo.
30. **EC2 không có memory/disk metric mặc định** → cài CloudWatch Agent.

---

## 9. Chiến thuật làm bài thi

### 9.1 Loại trừ đáp án — thứ tự

1. **Loại ngay đáp án nhắc đến dịch vụ đã deprecated**: CodeStar, Cloud9, CodeCommit (với user mới), OpsWorks.
2. **Loại đáp án sai kỹ thuật** (VD: "Lambda chạy 20 phút" — sai vì max 15 phút).
3. **Loại đáp án bảo mật kém**: hardcode access key, public S3, dùng root user, IMDSv1.
4. **Loại đáp án quá phức tạp** nếu có pattern đơn giản hơn (VD: tự build retry logic khi SQS đã có).

### 9.2 Dấu hiệu từ khóa trong đề

| Từ khóa trong đề | Thường gợi ý |
|---|---|
| "Serverless" | Lambda, Fargate, DynamoDB, API Gateway, S3, Step Functions |
| "Least privilege" | IAM Role specific, không dùng `*` |
| "Cross-account" | IAM Role + sts:AssumeRole hoặc Resource-based policy |
| "Cost-effective" | Spot, Lambda, S3 lifecycle, Parameter Store over Secrets Manager |
| "Low-latency" / "real-time" | Kinesis Data Streams (< 1s), DAX, CloudFront, ElastiCache |
| "Near-real-time" | Kinesis Firehose (60s+) |
| "Long-running" | Step Functions Standard, ECS/Fargate, Batch (không Lambda) |
| "Highly available" | Multi-AZ, ASG, ELB, multi-region |
| "Fault-tolerant" | Dead-letter queue, retry, multi-AZ |
| "Encrypted at rest" | KMS (SSE-KMS cho S3, EBS encryption, RDS encryption) |
| "Encrypted in transit" | TLS/SSL, HTTPS, ACM cert |
| "Rotate credentials" | Secrets Manager (auto) hoặc IAM role (temp creds) |
| "Audit" | CloudTrail |
| "Compliance" | AWS Config, Artifact, SCP |
| "Feature flag" | AppConfig |
| "Canary deployment" | CodeDeploy (Lambda/ECS), Beanstalk Traffic splitting, Route 53 Weighted |
| "Rollback quickly" | Blue/Green hoặc Immutable (Beanstalk) |
| "Dynamic config runtime" | AppConfig, SSM Parameter Store với cfn-hup |
| "Fan-out" | SNS → SQS |
| "Replay events" | Kinesis Data Streams, EventBridge Archive |
| "Decouple" | SQS, SNS, EventBridge |

### 9.3 Lựa chọn giữa 2 đáp án gần giống

- **Đáp án có IAM Role** > đáp án có **hardcoded access key**.
- **Đáp án dùng managed service** (Parameter Store) > **tự build** (lưu trong env var).
- **Đáp án AWS-native** > third-party nếu không có lý do rõ ràng.
- **Đáp án có retry/DLQ** > không có error handling.
- **Đáp án native HTTP API** > REST API nếu không cần feature REST-only.
- **Đáp án ACM cert** > tự import cert.
- **Đáp án VPC Endpoint** > NAT Gateway (nếu chỉ gọi AWS service, tiết kiệm hơn).

### 9.4 Quản lý thời gian

- 65 câu / 130 phút = **2 phút/câu**.
- Đánh dấu câu khó, skip, quay lại sau.
- Đọc kỹ từ khóa: "MOST cost-effective", "LEAST operational overhead", "FASTEST".
- Đáp án dài hơn không có nghĩa là đúng — đọc kỹ constraint.

---

## 10. Checklist ôn cuối cùng — 1 tuần trước thi

### Ngày 1-2: Review hard topics

- [ ] Lambda giới hạn (memory, timeout, package, /tmp, payload).
- [ ] Lambda concurrency (reserved vs provisioned) + cold start.
- [ ] Lambda VPC + NAT/Endpoint.
- [ ] DynamoDB RCU/WCU calculation (4 trường hợp).
- [ ] DynamoDB LSI vs GSI + throttle behavior.

### Ngày 3: IAM deep-dive

- [ ] Policy evaluation logic (Deny > Allow > Implicit Deny).
- [ ] Resource-based vs Identity-based policy.
- [ ] Cross-account: AssumeRole + trust policy.
- [ ] KMS Key Policy (bắt buộc, không bypass được bằng IAM admin).
- [ ] STS AssumeRole duration (max chain 1 giờ).

### Ngày 4: CI/CD

- [ ] CodeBuild buildspec 4 phases.
- [ ] CodeDeploy hooks EC2 theo thứ tự.
- [ ] CodeDeploy Lambda/ECS appspec.
- [ ] CloudFormation intrinsic functions + DeletionPolicy.
- [ ] SAM CLI + Deploy Preferences.

### Ngày 5: Integration & Monitoring

- [ ] SQS Visibility Timeout + DLQ behavior.
- [ ] SNS fan-out pattern.
- [ ] Kinesis KDS vs Firehose.
- [ ] Step Functions Standard vs Express + `.waitForTaskToken`.
- [ ] X-Ray integration với các compute service.
- [ ] CloudWatch Agent cho EC2 memory/disk.

### Ngày 6: Practice exam

- [ ] Làm 1 bộ practice exam đầy đủ (Tutorials Dojo, Stephane Maarek, AWS official).
- [ ] Review câu sai, note topic yếu.
- [ ] Đọc kỹ explanation kể cả câu đúng.

### Ngày 7: Ngày trước thi

- [ ] Review note yếu.
- [ ] Ngủ đủ.
- [ ] **Không học mới** — chỉ refresh.

---

## 11. Tài liệu & practice test khuyến nghị

- **Stephane Maarek course** (Udemy) — toàn diện nhất cho DVA.
- **Neal Davis / Adrian Cantrill** — alternative nếu thích style khác.
- **Tutorials Dojo practice exams** — sát đề nhất.
- **AWS Skill Builder** — official questions, sample questions free.
- **AWS Whitepapers**:
  - [Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
  - [Lambda Operator Guide](https://docs.aws.amazon.com/lambda/latest/operatorguide/intro.html)
  - [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- **AWS Documentation**: đọc "Developer Guide" của các service trọng tâm (Lambda, DynamoDB, S3, IAM).

---

## 12. Lời kết

Chuỗi 6 phần này đã phủ gần như toàn bộ syllabus DVA-C02:

| Phần | Nội dung | Tỷ lệ trong đề |
|---|---|---|
| **Phần 1** | Compute & Containers | ~25% |
| **Phần 2** | Storage & Database | ~20% |
| **Phần 3** | App Integration | ~15% |
| **Phần 4** | Security & Identity | ~20% |
| **Phần 5** | Deployment & Monitoring | ~15% |
| **Phần 6** | Networking + Tips | ~5% |

**3 nguyên tắc vàng ôn DVA-C02:**

1. **Hands-on** — tự làm Lambda, CodePipeline, CloudFormation. Đọc suông không đủ.
2. **Biết LIMITS** — con số cụ thể (15 phút Lambda, 29s API Gateway, 256 KB SQS, 4 KB KMS, 400 KB DynamoDB item...).
3. **Practice theo thời gian thực** — làm đề full 130 phút để làm quen pressure.

Chúc bạn thi **PASS với điểm cao**! 🎯

---

**Hết chuỗi tài liệu DVA-C02. Nếu cần đào sâu bất kỳ phần nào, hoặc cần thêm:**
- Lab hands-on cụ thể theo service
- Giải thích từng câu hỏi mẫu
- Ôn 1 topic riêng chi tiết hơn
- So sánh DVA-C02 vs các chứng chỉ khác (SAA, SysOps)

→ Hỏi lại bất cứ lúc nào!
