---
title: "DVA-C02 Phần 6: Networking, Edge & Tổng Hợp Tips Thi"
author: thanhnv1808
date: 2026-04-29 08:50:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, vpc, route53, cloudfront, global-accelerator, exam-tips, certification]
description: "VPC, Route 53, CloudFront, Global Accelerator + Decision Trees tổng hợp, TOP 30 bẫy kinh điển, chiến thuật làm bài thi DVA-C02."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> [Phần 1: Compute](/posts/dva-c02-part1-compute) → [Phần 2: Storage & DB](/posts/dva-c02-part2-storage-database) → [Phần 3: App Integration](/posts/dva-c02-part3-app-integration) → [Phần 4: Security](/posts/dva-c02-part4-security) → [Phần 5: Deploy & Monitor](/posts/dva-c02-part5-deploy-monitor) → **Phần 6 (file này — KẾT THÚC)**
{: .prompt-info }

---

## 1. Amazon VPC — Biết Đủ Dùng

### 1.1 Các Thành Phần Chính

| Component | Mục đích |
|---|---|
| **VPC** | Virtual network, 1 region, CIDR từ `/16` đến `/28` |
| **Subnet** | Chia VPC theo AZ. Public (có route đến IGW) hoặc Private |
| **Route Table** | Routing rule cho subnet |
| **Internet Gateway (IGW)** | Out-in traffic Internet, gắn 1 VPC |
| **NAT Gateway** | Private subnet → Internet (outbound only). Managed, HA trong AZ |
| **Security Group** | **Stateful**, attach vào ENI. Allow only |
| **NACL** | **Stateless**, attach vào subnet. Allow & Deny, đánh số thứ tự |
| **VPC Peering** | 1-1 giữa 2 VPC, **KHÔNG transitive** |
| **Transit Gateway** | Hub-spoke, nhiều VPC + on-prem |
| **VPC Endpoint (Gateway)** | S3, DynamoDB — qua route table, **miễn phí** |
| **VPC Endpoint (Interface/PrivateLink)** | Hầu hết AWS service — qua ENI + DNS, tốn phí |

---

### 1.2 Security Group vs NACL — BẢNG SO SÁNH KINH ĐIỂN

| | **Security Group** | **NACL** |
|---|---|---|
| Level | ENI/Instance | Subnet |
| State | **Stateful** (return traffic tự động allow) | **Stateless** (phải allow cả in & out) |
| Rules | Only **Allow** | Allow & **Deny** |
| Rule order | Tất cả rules evaluate | Theo **rule number** (ascending, dừng khi match) |
| Default | Allow all outbound, deny all inbound | Custom: deny all |

> **Bẫy kinh điển:** "Ping EC2 không được, Security Group đã mở ICMP in-bound?" → Check **NACL outbound** cho ephemeral ports (1024-65535).
{: .prompt-warning }

---

### 1.3 VPC Endpoints — Khi Nào Dùng

| | **Gateway Endpoint** | **Interface Endpoint (PrivateLink)** |
|---|---|---|
| Services | **S3, DynamoDB** | Hầu hết AWS service (SQS, SNS, KMS...) |
| Chi phí | **Miễn phí** | Tốn tiền (theo giờ + GB) |
| Cơ chế | Qua route table | ENI + DNS private |

> **Tip:** Lambda/EC2 trong private subnet muốn gọi AWS service **mà không cần NAT** → VPC Endpoint.
{: .prompt-tip }

---

### 1.4 Bastion Host vs Systems Manager Session Manager

| | **Bastion Host** | **SSM Session Manager** |
|---|---|---|
| Cần SSH port mở | ✅ | ❌ |
| Cần key pair | ✅ | ❌ |
| Session log | ❌ | ✅ (S3/CloudWatch) |
| Bảo mật | Thấp hơn | **Cao hơn — luôn ưu tiên** |

---

### 1.5 VPC Flow Logs

- Capture metadata network traffic (IP, port, protocol, accept/reject).
- Xuất ra **CloudWatch Logs, S3, Kinesis Firehose**.
- **KHÔNG log payload**, chỉ header.

---

## 2. Amazon Route 53 — Biết Routing Policies

### 2.1 Record Types Cần Biết

| Record | Mô tả |
|---|---|
| `A` | IPv4 |
| `AAAA` | IPv6 |
| `CNAME` | Alias (domain) — **KHÔNG cho root domain** |
| `ALIAS` | AWS-specific, trỏ tới AWS resource. **Cho root domain được. Miễn phí** |
| `MX`, `TXT`, `NS`, `SOA` | Khác |

---

### 2.2 Routing Policies — THUỘC LÒNG

| Policy | Mục đích |
|---|---|
| **Simple** | 1 record, nhiều value → R53 random 1 |
| **Weighted** | % traffic theo weight → canary deployment |
| **Latency-based** | Trả record có latency thấp nhất từ client |
| **Failover** | Primary + Secondary + health check → auto switch |
| **Geolocation** | Theo country/continent của client (không phải latency) |
| **Geoproximity** (qua Traffic Flow) | Theo khoảng cách + bias |
| **Multi-value answer** | Tới 8 healthy records random — poor-man's LB |

---

### 2.3 Health Checks

- Monitor endpoint (HTTP/HTTPS/TCP).
- Monitor CloudWatch alarm.
- Monitor health check khác (calculated health check).
- **Required cho Failover policy**.

---

## 3. Amazon CloudFront ⭐

### 3.1 Khái Niệm

- CDN toàn cầu với 400+ edge locations.
- **Origin types**: S3, ALB, EC2, MediaPackage, HTTP server (on-prem).

---

### 3.2 Caching Behavior

- **Distribution** → **Behaviors** theo path pattern (VD `/static/*` behavior riêng).
- **Cache Policy**: TTL, các key cache (query string, header, cookie).
- **Origin Request Policy**: headers/cookies/query gửi đến origin.

---

### 3.3 Cache Invalidation

- Invalidate theo path (`/images/*`) — tốn phí sau 1000 free/tháng.
- Hoặc đổi **versioning** (VD `image-v2.jpg`) — **khuyến nghị hơn**.

---

### 3.4 OAI vs OAC

| | **OAI** | **OAC** (mới, 2022+) |
|---|---|---|
| Trạng thái | Đang deprecated | **Khuyến nghị dùng** |
| Hỗ trợ SSE-KMS | Hạn chế | ✅ |
| Dynamic request | ❌ | ✅ |

**Cả 2 đều giúp**: Bucket S3 không public → chỉ CloudFront truy cập được.

---

### 3.5 Signed URLs vs Signed Cookies

| | **Signed URL** | **Signed Cookies** |
|---|---|---|
| Scope | 1 file | Nhiều file, domain-wide |
| Use case | Download link cá nhân | Premium content (streaming) |

---

### 3.6 Edge Functions

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Language | JavaScript | Node.js, Python |
| Max timeout | **< 1 ms** | 5s (viewer), 30s (origin) |
| Triggers | Viewer request/response | Tất cả 4 trigger |
| Use case | Header/URL manipulation đơn giản | Logic phức tạp, gọi service |

---

### 3.7 Geo Restriction

- Whitelist/blacklist country access.

---

## 4. AWS Global Accelerator

- **2 static Anycast IP** + Edge network của AWS.
- Route traffic qua **backbone AWS** thay vì Internet → latency thấp, packet loss ít.

> **CloudFront vs Global Accelerator:**
> - Static content, cache-able, HTTP/HTTPS → **CloudFront**
> - Non-HTTP (TCP/UDP), cần static IP, low-latency connection → **Global Accelerator**
{: .prompt-tip }

---

## 5. AWS Direct Connect & VPN (Biết Sơ)

| | **VPN** | **Direct Connect** |
|---|---|---|
| Qua Internet | ✅ (encrypted) | ❌ (dedicated fiber) |
| Latency | Cao hơn | Thấp hơn |
| Bandwidth | Giới hạn | Cao |
| Setup time | Nhanh | Vài tuần |

---

# 🎯 TỔNG HỢP TIPS THI DVA-C02

---

## 6. Cấu Trúc Kỳ Thi DVA-C02

| Thông tin | Chi tiết |
|---|---|
| **Số câu** | 65 câu (~50 tính điểm + ~15 unscored) |
| **Thời gian** | **130 phút** |
| **Pass score** | **720/1000** |
| **Loại câu** | Đa lựa chọn (1 đáp án) & đa phản hồi (2-3 đáp án trong 5-6 options) |

**Tỷ lệ các domain:**

| Domain | Tỷ lệ |
|---|---|
| **Domain 1: Development with AWS Services** | 32% |
| **Domain 2: Security** | 26% |
| **Domain 3: Deployment** | 24% |
| **Domain 4: Troubleshooting & Optimization** | 18% |

---

## 7. Decision Trees Tổng Hợp

### 7.1 Khi Nào Dùng Dịch Vụ Compute?

```
Cần chạy code:
├── Event-driven, < 15 phút, stateless?        → Lambda
├── Long-running container?
│   ├── Quản server                             → ECS (EC2)
│   └── Serverless                             → Fargate / EKS Fargate
├── K8s ecosystem?                              → EKS
├── Batch job dài?                              → AWS Batch
├── Đã có codebase + muốn PaaS?                → Elastic Beanstalk
└── Full control VM?                           → EC2
```

---

### 7.2 Khi Nào Dùng Dịch Vụ Database?

```
Dữ liệu:
├── Quan hệ (SQL)?
│   ├── Full managed + high-perf               → Aurora
│   ├── Standard                               → RDS
│   └── Data warehouse (OLAP)                  → Redshift
├── NoSQL key-value / document?
│   ├── Ultra-low latency                      → DynamoDB (+DAX nếu cần μs)
│   └── MongoDB-compatible                     → DocumentDB
├── Graph?                                     → Neptune
├── Ledger / audit trail?                      → QLDB
├── Time-series?                               → Timestream
├── In-memory cache?
│   ├── Redis ecosystem, HA                    → ElastiCache Redis
│   ├── Simple caching                         → ElastiCache Memcached
│   └── Cache DynamoDB specifically            → DAX
└── Search/log analytics?                      → OpenSearch
```

---

### 7.3 Khi Nào Dùng Dịch Vụ Messaging?

```
Message pattern:
├── Queue (1 sender → 1 consumer)?
│   ├── Cần order chính xác                    → SQS FIFO
│   └── Throughput cao, eventual order         → SQS Standard
├── Pub/sub (1 sender → N consumer)?
│   ├── AWS services + HTTP/Email/SMS          → SNS
│   └── Event routing với filter mạnh, cron   → EventBridge
├── Fan-out (1 event → N xử lý độc lập)?      → SNS → SQS
├── Real-time streaming, retention, replay?
│   ├── Custom processing                      → Kinesis Data Streams
│   └── Chỉ cần load vào S3/Redshift/...      → Kinesis Firehose
├── Kafka-compatible migration?                → MSK
├── AMQP/MQTT/JMS (legacy migration)?         → Amazon MQ
└── Workflow orchestration (multi-step)?
    ├── Long-running, stateful                 → Step Functions Standard
    └── Short, high-volume                    → Step Functions Express
```

---

### 7.4 Khi Nào Dùng Secret/Config?

```
Store sensitive data:
├── Cần auto-rotation (VD: DB password)?       → Secrets Manager
├── Config thông thường (env var, URL)?
│   ├── Feature flag, runtime config           → AppConfig
│   └── Key-value đơn giản, miễn phí          → SSM Parameter Store
└── Encrypted files / arbitrary data > 64 KB? → S3 + KMS encryption
```

---

### 7.5 Khi Nào Dùng Bảo Mật/Audit?

```
Mục đích:
├── Threat detection ML (bitcoin mining, compromise)?    → GuardDuty
├── Vulnerability scan (EC2, ECR, Lambda)?               → Inspector
├── PII trong S3?                                        → Macie
├── API call audit (who did what when)?                  → CloudTrail
├── Resource config compliance?                          → AWS Config
├── DDoS protection?
│   ├── Free L3/L4                                      → Shield Standard
│   └── Advanced L7                                     → Shield Advanced + WAF
├── Layer 7 filter (SQLi, XSS, bot)?                    → WAF
└── Aggregate findings?                                  → Security Hub
```

---

## 8. TOP 30 BẪY KINH ĐIỂN DVA-C02

### Compute & Container

| # | Bẫy |
|---|---|
| 1 | **Lambda timeout 15 phút** — workload > 15 phút → Fargate/Batch |
| 2 | **Lambda trong VPC không có NAT** → không gọi Internet/AWS service (trừ VPC Endpoint) |
| 3 | **Lambda async retry 2 lần** (tổng 3 lần) với exponential backoff |
| 4 | **ECS Task Execution Role** (pull image, log) ≠ **Task Role** (app API calls) |
| 5 | **Beanstalk `Immutable`** cho zero-downtime + rollback nhanh |
| 6 | **User Data chỉ chạy 1 lần đầu** (không phải mỗi reboot) |
| 7 | **IMDSv2 luôn thay IMDSv1** (chống SSRF) |

### Storage & Database

| # | Bẫy |
|---|---|
| 8 | **S3 strong consistency** (từ 12/2020) — đừng bị lừa tài liệu cũ |
| 9 | **S3 Glacier Instant Retrieval** cho archive + truy cập ms |
| 10 | **S3 Bucket Keys** để giảm 99% KMS call |
| 11 | **DynamoDB GSI throttle → main table write throttle** (LSI không vậy) |
| 12 | **DynamoDB FilterExpression & ProjectionExpression KHÔNG giảm RCU** |
| 13 | **DynamoDB Strongly consistent read tốn gấp đôi RCU** |
| 14 | **RDS Multi-AZ KHÔNG scale read** — chỉ HA. Dùng Read Replica |
| 15 | **EBS volume bị khóa AZ** — move AZ → snapshot + restore |

### App Integration

| # | Bẫy |
|---|---|
| 16 | **SQS Visibility Timeout quá ngắn → duplicate processing** |
| 17 | **SQS Long polling (20s) để tiết kiệm cost** |
| 18 | **SQS FIFO: 300 TPS** (3000 với batch). Cao hơn → High Throughput mode |
| 19 | **Kinesis Enhanced Fan-out** khi nhiều consumer đọc cùng stream |
| 20 | **API Gateway timeout 29s cứng** — long-running phải async |
| 21 | **Cognito User Pool Authorizer** chỉ authenticate (không authorize) |

### Security

| # | Bẫy |
|---|---|
| 22 | **IAM: Explicit Deny luôn thắng Explicit Allow** |
| 23 | **KMS Key Policy là bắt buộc** — IAM admin không tự động access được CMK |
| 24 | **KMS Encrypt API giới hạn 4 KB** → data lớn dùng Envelope Encryption |
| 25 | **ACM chỉ cho AWS-managed services** (ALB, CloudFront, API GW). EC2 self-host KHÔNG xài được |
| 26 | **Secrets Manager cần rotation** / **Parameter Store cho config không rotate** |

### Deploy & Monitor

| # | Bẫy |
|---|---|
| 27 | **CloudFormation DeletionPolicy: Retain** để giữ resource khi xóa stack |
| 28 | **CodeDeploy Lambda phải dùng alias**, không deploy vào $LATEST |
| 29 | **CloudWatch Metric Filter KHÔNG retroactive** |
| 30 | **EC2 không có memory/disk metric mặc định** → cài CloudWatch Agent |

---

## 9. Chiến Thuật Làm Bài Thi

### 9.1 Loại Trừ Đáp Án

1. **Loại ngay đáp án nhắc đến dịch vụ deprecated**: CodeStar, Cloud9, CodeCommit (với user mới), OpsWorks.
2. **Loại đáp án sai kỹ thuật** (VD: "Lambda chạy 20 phút" — sai vì max 15 phút).
3. **Loại đáp án bảo mật kém**: hardcode access key, public S3, dùng root user, IMDSv1.
4. **Loại đáp án quá phức tạp** nếu có pattern đơn giản hơn.

---

### 9.2 Từ Khóa Trong Đề

| Từ khóa trong đề | Thường gợi ý |
|---|---|
| "Serverless" | Lambda, Fargate, DynamoDB, API Gateway, S3, Step Functions |
| "Least privilege" | IAM Role specific, không dùng `*` |
| "Cross-account" | IAM Role + sts:AssumeRole hoặc Resource-based policy |
| "Cost-effective" | Spot, Lambda, S3 lifecycle, Parameter Store over Secrets Manager |
| "Low-latency / real-time" | Kinesis Data Streams (<1s), DAX, CloudFront, ElastiCache |
| "Near-real-time" | Kinesis Firehose (60s+) |
| "Long-running" | Step Functions Standard, ECS/Fargate, Batch (không Lambda) |
| "Highly available" | Multi-AZ, ASG, ELB, multi-region |
| "Fault-tolerant" | Dead-letter queue, retry, multi-AZ |
| "Encrypted at rest" | KMS (SSE-KMS cho S3, EBS, RDS) |
| "Encrypted in transit" | TLS/SSL, HTTPS, ACM cert |
| "Rotate credentials" | Secrets Manager (auto) hoặc IAM role (temp creds) |
| "Audit" | CloudTrail |
| "Compliance" | AWS Config, Artifact, SCP |
| "Feature flag" | AppConfig |
| "Canary deployment" | CodeDeploy (Lambda/ECS), Beanstalk Traffic splitting, Route 53 Weighted |
| "Rollback quickly" | Blue/Green hoặc Immutable (Beanstalk) |
| "Fan-out" | SNS → SQS |
| "Replay events" | Kinesis Data Streams, EventBridge Archive |
| "Decouple" | SQS, SNS, EventBridge |

---

### 9.3 Lựa Chọn Giữa 2 Đáp Án Gần Giống

| Ưu tiên | Thay vì |
|---|---|
| **IAM Role** | Hardcoded access key |
| **Managed service** (Parameter Store) | Tự build (lưu trong env var) |
| **AWS-native** | Third-party (nếu không có lý do rõ ràng) |
| **Retry/DLQ** | Không có error handling |
| **HTTP API** | REST API (nếu không cần feature REST-only) |
| **ACM cert** | Tự import cert |
| **VPC Endpoint** | NAT Gateway (nếu chỉ gọi AWS service, tiết kiệm hơn) |

---

### 9.4 Quản Lý Thời Gian

- **65 câu / 130 phút = 2 phút/câu**.
- Đánh dấu câu khó, skip, quay lại sau.
- Đọc kỹ từ khóa: **"MOST cost-effective"**, **"LEAST operational overhead"**, **"FASTEST"**.
- Đáp án dài hơn không có nghĩa là đúng — đọc kỹ constraint.

---

## 10. Checklist Ôn Cuối — 1 Tuần Trước Thi

### Ngày 1-2: Compute & Database

- [ ] Lambda giới hạn (memory, timeout, package, /tmp, payload).
- [ ] Lambda concurrency (reserved vs provisioned) + cold start.
- [ ] Lambda VPC + NAT/Endpoint.
- [ ] DynamoDB RCU/WCU calculation (4 trường hợp).
- [ ] DynamoDB LSI vs GSI + throttle behavior.

### Ngày 3: IAM & Security

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
- [ ] X-Ray integration + Annotations vs Metadata.
- [ ] CloudWatch Agent cho EC2 memory/disk.

### Ngày 6: Practice Exam

- [ ] Làm 1 bộ practice exam đầy đủ (Tutorials Dojo, Stephane Maarek, AWS official).
- [ ] Review câu sai, note topic yếu.
- [ ] Đọc kỹ explanation kể cả câu đúng.

### Ngày 7: Ngày Trước Thi

- [ ] Review note yếu.
- [ ] Ngủ đủ.
- [ ] **Không học mới** — chỉ refresh.

---

## 11. Tài Liệu Khuyến Nghị

| Tài liệu | Mô tả |
|---|---|
| **Stephane Maarek (Udemy)** | Toàn diện nhất cho DVA — khuyến nghị #1 |
| **Neal Davis / Adrian Cantrill** | Alternative nếu thích style khác |
| **Tutorials Dojo practice exams** | Sát đề nhất |
| **AWS Skill Builder** | Official questions, sample questions free |
| **AWS Well-Architected Framework** | Whitepaper quan trọng |

---

## 12. Tổng Kết Chuỗi DVA-C02

| Phần | Nội dung | Tỷ lệ trong đề |
|---|---|---|
| **Phần 1** | Compute & Containers | ~25% |
| **Phần 2** | Storage & Database | ~20% |
| **Phần 3** | App Integration | ~15% |
| **Phần 4** | Security & Identity | ~20% |
| **Phần 5** | Deployment & Monitoring | ~15% |
| **Phần 6** | Networking + Tips | ~5% |

---

> **3 nguyên tắc vàng ôn DVA-C02:**
>
> 1. **Hands-on** — tự làm Lambda, CodePipeline, CloudFormation. Đọc suông không đủ.
> 2. **Biết LIMITS** — con số cụ thể (15 phút Lambda, 29s API Gateway, 256 KB SQS, 4 KB KMS, 400 KB DynamoDB item...).
> 3. **Practice theo thời gian thực** — làm đề full 130 phút để làm quen pressure.
{: .prompt-tip }

---

> **Chúc bạn thi PASS với điểm cao!**
> Nếu cần đào sâu bất kỳ phần nào, review lại từng bài trong chuỗi này.
{: .prompt-info }
