---
title: "DVA-C02 Phần 2: Storage & Database"
author: thanhnv1808
date: 2026-04-29 08:10:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, s3, ebs, efs, dynamodb, rds, aurora, elasticache, certification]
description: "S3, EBS, EFS, DynamoDB, RDS, Aurora, ElastiCache — tất cả storage & database services cần thuộc lòng cho kỳ thi AWS DVA-C02."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> [Phần 1: Compute](/posts/dva-c02-part1-compute) → **Phần 2 (file này)** → [Phần 3: App Integration](/posts/dva-c02-part3-app-integration) → [Phần 4: Security](/posts/dva-c02-part4-security) → [Phần 5: Deploy & Monitor](/posts/dva-c02-part5-deploy-monitor) → [Phần 6: Networking + Tips thi](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }

---

## 1. Amazon S3 ⭐ (Xuất hiện nhiều câu)

### 1.1 Khái niệm cơ bản

- **Bucket** (globally unique name) → **Objects** (key = full path, value = nội dung).
- **Max object size: 5 TB**. Upload 1 request tối đa **5 GB** → trên đó phải **Multipart Upload**.
- **Khuyến nghị Multipart Upload khi file > 100 MB**. Bắt buộc khi > 5 GB.
- **S3 Strong read-after-write consistency** cho mọi operation kể từ Dec 2020.

> **S3 giờ là Strong Consistency** — đừng bị lừa bởi tài liệu cũ nói eventual consistency!
{: .prompt-warning }

---

### 1.2 Storage Classes — THUỘC LÒNG

| Class | Availability | Min storage | Retrieval fee | Use case |
|---|---|---|---|---|
| **S3 Standard** | 99.99% | — | Không | Frequent access |
| **S3 Intelligent-Tiering** | 99.9% | — | Không (phí monitoring) | Unknown/changing access pattern |
| **S3 Standard-IA** | 99.9% | 30 ngày | Có | Infrequent, cần truy cập nhanh |
| **S3 One Zone-IA** | 99.5% | 30 ngày | Có | IA nhưng data tái tạo được (1 AZ) |
| **S3 Glacier Instant Retrieval** | 99.9% | 90 ngày | Có | Archive + truy cập tức thì (ms) |
| **S3 Glacier Flexible Retrieval** | 99.99% | 90 ngày | Có | Archive, retrieval: Expedited (1-5 phút), Standard (3-5h), Bulk (5-12h) |
| **S3 Glacier Deep Archive** | 99.99% | 180 ngày | Có | Archive rẻ nhất, retrieval: Standard (12h), Bulk (48h) |

> **Bẫy thi thường gặp:**
> - "Cheapest, cần lấy trong vài giây?" → **Glacier Instant Retrieval**
> - "Data 1 lần/năm, lấy trong vài phút?" → **Glacier Flexible, Expedited**
> - "Không biết pattern truy cập?" → **Intelligent-Tiering**
{: .prompt-tip }

---

### 1.3 Lifecycle Rules

- **Transition actions**: chuyển class (Standard → IA → Glacier...).
- **Expiration actions**: xóa object/version sau X ngày.
- Áp dụng cho prefix, tag, hoặc cả bucket.
- **Minimum storage duration** phải được tôn trọng — transition sớm hơn sẽ bị charge full period.

---

### 1.4 Versioning

- Bật/tắt ở **bucket level**. Khi tắt → thành `Suspended` (version cũ vẫn còn).
- **Delete marker**: "xóa" ở bucket versioning chỉ thêm delete marker. Xóa vĩnh viễn phải chỉ định `versionId`.
- **MFA Delete**: bảo vệ xóa version, chỉ bucket owner (root account) bật được qua CLI.

---

### 1.5 Encryption — 4 loại

| Loại | Key quản bởi | Header request |
|---|---|---|
| **SSE-S3** (mặc định từ 2023) | AWS (AES-256) | `x-amz-server-side-encryption: AES256` |
| **SSE-KMS** | AWS KMS | `x-amz-server-side-encryption: aws:kms` |
| **SSE-C** | Customer cung cấp key mỗi request | Header kèm key |
| **Client-side** | Customer | Encrypt trước khi upload |

> **SSE-KMS**: mỗi request là 1 call tới KMS → dễ bị throttle. Dùng **S3 Bucket Keys** để giảm KMS call (giảm tới 99% chi phí KMS).
{: .prompt-warning }

---

### 1.6 Bucket Policy vs ACL vs IAM

| Loại | Scope | Use case |
|---|---|---|
| **IAM policy** | User/role | Kiểm soát từ phía principal |
| **Bucket Policy** | Bucket | Cross-account, public access, enforce encryption/HTTPS |
| **ACL** | Object/bucket | **Legacy — AWS khuyến nghị tắt** |
| **Block Public Access** | Account/bucket | Chặn public ngay cả khi policy cho phép |

---

### 1.7 Pre-signed URLs

- Cho phép user tạm truy cập object mà không cần IAM credentials.
- Kế thừa quyền của **người tạo URL** → người tạo cần có quyền tương ứng.
- Default expiration: CLI = 3600s, SDK = 15 phút (tuỳ ngôn ngữ).

> **Pattern phổ biến:** User login → app generate pre-signed URL → user upload lên S3 trực tiếp (tránh proxy qua backend, tiết kiệm).
{: .prompt-tip }

---

### 1.8 Replication

| | **CRR** | **SRR** |
|---|---|---|
| Scope | Cross-Region | Same-Region |
| Use case | DR, compliance, latency | Log aggregation, env sync |

- Yêu cầu: **Versioning bật ở cả source và destination**, IAM role.
- **KHÔNG replicate retroactively** mặc định → bật **S3 Batch Replication** cho data cũ.
- **Delete marker replication** có thể bật/tắt. Xóa vĩnh viễn (với versionId) KHÔNG replicate.

---

### 1.9 Performance

- **Multipart Upload**: parallel, resumable, faster. Dùng `AbortMultipartUpload` để dọn (lifecycle rule nên tự abort sau X ngày).
- **Transfer Acceleration**: upload qua CloudFront edge → S3 bucket.
- **Byte-range fetches**: parallel GET theo range → download lớn nhanh.
- **Prefix sharding**: giới hạn **3,500 PUT/DELETE + 5,500 GET/HEAD per second per prefix**. Chia prefix để tăng throughput.

---

### 1.10 S3 Object Lock

- **WORM** (Write Once Read Many). 2 mode:
  - **Governance**: user có quyền đặc biệt mới override được.
  - **Compliance**: **KHÔNG ai override được**, kể cả root.
- **Legal Hold**: block xóa/sửa đến khi remove hold.

---

### 1.11 S3 Select & Glacier Select

- Query bằng **SQL đơn giản** trên CSV/JSON/Parquet → chỉ tải data cần → tiết kiệm băng thông.

---

### 1.12 Static Website Hosting

- URL format: `bucket-name.s3-website-<region>.amazonaws.com`.
- Phải bật **Block Public Access = OFF** + bucket policy cho phép `s3:GetObject` cho `Principal: *`.
- Dùng **CloudFront** phía trước cho HTTPS, custom domain.

---

## 2. Amazon EBS (Elastic Block Store)

### 2.1 Volume Types

| Type | IOPS | Use case | Ghi chú |
|---|---|---|---|
| **gp3** (mặc định mới) | 3,000-16,000 | General purpose | IOPS & throughput độc lập với size — **gp3 rẻ hơn gp2 ~20%** |
| **gp2** | 3 IOPS/GB, burst 3,000 | General purpose | IOPS gắn với size |
| **io1 / io2** | Tới 64,000 (io2 Block Express: 256,000) | High-performance DB | Multi-Attach |
| **st1** (HDD) | Throughput-optimized | Big data, log | **KHÔNG boot volume** |
| **sc1** (HDD) | Cold HDD | Infrequent access | **KHÔNG boot volume** |

---

### 2.2 Đặc tính quan trọng

- **AZ-specific**: volume bị khóa trong 1 AZ. Move AZ → **snapshot → restore ở AZ khác**.
- **Multi-Attach**: chỉ **io1/io2**, attach tới nhiều EC2 trong cùng AZ (max 16 instances), filesystem phải cluster-aware.
- **Snapshot**: incremental, lưu ở S3 (managed). Copy sang region khác được.
- **Snapshot Archive**: giảm 75% chi phí, restore chậm (24-72h).
- **Recycle Bin**: recover snapshot/AMI đã xóa.
- Encrypt volume cũ: snapshot → copy snapshot với encryption → restore.

---

### 2.3 Instance Store vs EBS

| | Instance Store | EBS |
|---|---|---|
| Persistence | **Ephemeral** — mất khi stop/terminate | Persistent |
| Speed | Cực nhanh (NVMe local) | Network-attached |
| Use case | Cache, buffer, scratch | Production data |

---

## 3. Amazon EFS (Elastic File System)

- **NFS v4 shared file system**, multi-AZ, scale tự động.
- Mount từ nhiều EC2 đồng thời (POSIX).
- **KHÔNG dùng với Windows** (Windows dùng **FSx**).

| Performance Mode | Mô tả |
|---|---|
| **General Purpose** (default) | Low latency |
| **Max I/O** | Higher throughput, higher latency → big data, parallel |

| Throughput Mode | Mô tả |
|---|---|
| **Bursting** (default) | Scale với data size |
| **Provisioned** | Cố định throughput |
| **Elastic** | Auto scale |

---

### So sánh Storage

| | EBS | EFS | FSx for Windows | FSx for Lustre | Instance Store |
|---|---|---|---|---|---|
| Protocol | Block | NFS | SMB | Lustre | Block |
| Access | 1 EC2 (hoặc Multi-Attach io1/io2) | Nhiều Linux EC2 | Nhiều Windows EC2 | HPC | Local only |
| Persistence | Persistent | Persistent | Persistent | Persistent | Ephemeral |

---

## 4. Amazon DynamoDB ⭐ (Cực quan trọng)

### 4.1 Khái niệm cơ bản

- NoSQL key-value + document, **fully managed, serverless**.
- **Primary Key**: Partition Key (PK) duy nhất hoặc Partition Key + Sort Key (composite).

---

### 4.2 Capacity Modes

| Mode | Khi nào dùng |
|---|---|
| **Provisioned** | Traffic đoán được, muốn tiết kiệm với Reserved Capacity |
| **On-Demand** | Traffic không đoán được, spiky, dev |

**Công thức RCU/WCU — hay hỏi trong đề:**

| Operation | Công thức |
|---|---|
| **WCU** | `ceil(item_KB / 1)` per item |
| **Strongly consistent read** | `ceil(item_KB / 4)` per item |
| **Eventually consistent read** | `ceil(item_KB / 4) / 2` per item (làm tròn lên) |
| **Transactional** | **Gấp đôi** WCU/RCU tương ứng |

> **Ví dụ:** Write 10 items/s, mỗi item 4.5 KB → WCU = 10 × ceil(4.5/1) = **10 × 5 = 50 WCU**
>
> Strongly consistent read 10 items/s, mỗi 6 KB → RCU = 10 × ceil(6/4) = **10 × 2 = 20 RCU**
{: .prompt-tip }

---

### 4.3 Các API chính

| API | Mô tả |
|---|---|
| **PutItem** | Insert/replace item |
| **UpdateItem** | Update attribute, hỗ trợ **atomic counter** |
| **DeleteItem** | Delete item |
| **BatchWriteItem** | Tới 25 items, 16 MB |
| **BatchGetItem** | Tới 100 items, 16 MB |
| **GetItem** | Single item by PK |
| **Query** | Theo PK (bắt buộc) + SK (optional). Nhanh |
| **Scan** | Đọc toàn bảng → chậm, tốn RCU |

- **Conditional writes**: `ConditionExpression` → chỉ write khi điều kiện true → chống race condition.

---

### 4.4 Index — LSI vs GSI

| | **LSI** | **GSI** |
|---|---|---|
| Partition Key | Cùng PK với table | PK mới |
| Sort Key | SK khác | SK tuỳ chọn |
| Tạo khi nào | **Khi create table** — không tạo sau được | Tạo/xóa bất cứ lúc nào |
| Số lượng tối đa | 5/table | 20/table |
| Throughput | **Share với table** | **Throughput riêng** |

> **Bẫy thi:** "GSI throttle → chuyện gì xảy ra?" → **Write tới main table CŨNG bị throttle**.
{: .prompt-danger }

---

### 4.5 DynamoDB Streams

- Capture **INSERT/MODIFY/REMOVE** events. Retention **24h**.
- **StreamViewType**: `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, `NEW_AND_OLD_IMAGES`.
- Target: **Lambda**, **Kinesis Data Streams**.

---

### 4.6 TTL (Time To Live)

- Attribute kiểu **Number**, giá trị **epoch seconds**.
- Item tự xóa sau TTL → **KHÔNG tốn WCU**.
- Delete xảy ra trong vòng 48h.

---

### 4.7 Transactions

- `TransactWriteItems` / `TransactGetItems` — ACID.
- **Gấp 2 lần capacity** (1 write trong transaction = 2 WCU).
- Tới 100 items hoặc 4 MB/transaction.

---

### 4.8 DynamoDB Accelerator (DAX)

- **Fully managed in-memory cache** cho DynamoDB.
- Microsecond latency cho read (ms → μs).
- Transparent — không đổi code, chỉ đổi endpoint.
- **KHÁC ElastiCache**: DAX chuyên cho DynamoDB, ElastiCache general purpose.

---

### 4.9 Global Tables

- Multi-region, **active-active** replication.
- Yêu cầu Streams bật. Latency ~1s cross-region.

---

### 4.10 Tips & Bẫy

1. **Hot partition**: PK có distribution kém → dùng **random/composite PK** để phân tán.
2. **Query > Scan** — luôn thiết kế để query qua PK/GSI.
3. **FilterExpression & ProjectionExpression KHÔNG giảm RCU** — chỉ giảm data trả về.
4. **DAX cho read-heavy**, **ElastiCache cho compute caching**.
5. **Pagination**: `LastEvaluatedKey` → `ExclusiveStartKey` cho request tiếp theo.

---

## 5. Amazon RDS

### 5.1 Tính năng chính

- **Engines**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.
- **Multi-AZ**: standby replica ở AZ khác, **sync replication**, tự động failover. **CHỈ để HA, KHÔNG scale read**.
- **Read Replicas**: tới 15, async replication. Dùng để scale read. Cross-region, promote được.
- **Automated Backup**: daily snapshot + transaction log → PITR, retention 0-35 ngày.
- **Storage**: GP2/GP3/IO1/Magnetic. Auto scaling storage có thể bật.

> **Bẫy:** "Multi-AZ giúp scale read?" → **SAI**, chỉ HA. Dùng **Read Replica** để scale read.
{: .prompt-danger }

---

### 5.2 Security

- Encryption at-rest qua KMS (khi create, KHÔNG encrypt sau được → phải snapshot → copy encrypted → restore).
- IAM database authentication (MySQL, PostgreSQL): dùng token 15 phút thay password.

---

### 5.3 RDS Proxy

- Pool connection → giảm failover time, giảm connection overhead cho Lambda.
- Integrate với Secrets Manager / IAM auth.
- **Không publicly accessible**.

---

## 6. Amazon Aurora

### 6.1 Đặc tính

- MySQL & PostgreSQL compatible.
- **5x MySQL, 3x PostgreSQL** về performance.
- Storage auto-scale **10 GB → 128 TB**.
- **6 bản sao data** qua **3 AZ** (4 write, 3 read quorum). Self-healing, continuous backup to S3.

---

### 6.2 Endpoints

| Endpoint | Trỏ tới |
|---|---|
| **Writer endpoint** | Master |
| **Reader endpoint** | Load-balance tới read replicas |
| **Custom endpoint** | Subset of replicas cho workload cụ thể |
| **Instance endpoint** | 1 instance cụ thể |

---

### 6.3 Aurora Replicas

- Tới **15 Aurora Replicas** (vs 5 MySQL replica trên RDS).
- Auto-failover **< 30s**.

---

### 6.4 Aurora Serverless

| Version | Đặc điểm |
|---|---|
| **v1** | Auto scale, cold start, workload không thường xuyên |
| **v2** | Instant auto scale, không cold start (giữ capacity min > 0) |

---

### 6.5 Aurora Global Database

- 1 primary region + tới **5 secondary regions** (read-only).
- Replication latency **< 1s**. **Cross-region failover < 1 phút** (RTO).

---

### 6.6 Aurora Backtrack

- "Rewind" DB tới thời điểm trước (tới 72h) **mà KHÔNG restore** → không cần snapshot, nhanh.
- Chỉ **Aurora MySQL**.

---

## 7. Amazon ElastiCache

### 7.1 Redis vs Memcached — THUỘC LÒNG

| | **Redis** | **Memcached** |
|---|---|---|
| Multi-AZ với auto failover | ✅ | ❌ |
| Replication | ✅ | ❌ |
| Persistence (AOF) | ✅ | ❌ |
| Backup/Restore | ✅ | ❌ |
| Data structures | List, Set, Hash, Sorted Set, Bitmap... | String only |
| Multi-threaded | ❌ (single thread) | ✅ |
| Pub/Sub | ✅ | ❌ |
| Transactions | ✅ | ❌ |

> **Quyết định nhanh:**
> - Cần HA, persistence, advanced data structures → **Redis**
> - Cần cache đơn giản, multi-threaded, tiết kiệm → **Memcached**
{: .prompt-tip }

---

### 7.2 Cache Strategies

| Strategy | Cơ chế | Pros | Cons |
|---|---|---|---|
| **Lazy Loading** (Cache-Aside) | Read → miss → load từ DB → write cache | Chỉ cache data cần | Miss tốn 3 trips, data có thể stale |
| **Write-Through** | Write DB + write cache đồng thời | Data fresh | Cache có data không bao giờ đọc |
| **TTL** | Thường dùng chung với lazy loading | Tự hết hạn | — |

---

## 8. Các Database Khác — Biết Sơ

| Service | Mục đích | Use case |
|---|---|---|
| **Neptune** | Graph DB | Social, fraud, recommendation |
| **DocumentDB** | MongoDB-compatible | Document store |
| **QLDB** | Ledger DB, immutable, cryptographically verifiable | Audit trail (centralized, khác blockchain) |
| **Timestream** | Time-series DB | IoT, monitoring |
| **Keyspaces** | Cassandra-compatible | Wide-column |
| **Redshift** | Data warehouse, columnar, PB-scale | OLAP — khác RDS (OLTP) |

---

## 🎯 Tip Ôn Thi Phần 2

1. **S3 Storage Classes**: học thuộc min storage duration và use case.
2. **S3 Encryption**: SSE-KMS + throttle → bật **S3 Bucket Keys** để giảm 99% KMS call.
3. **S3 Strong Consistency** từ Dec 2020 — đừng để bị lừa.
4. **EBS vs EFS vs Instance Store vs FSx**: 1 EC2/block → EBS; nhiều Linux EC2/shared → EFS; Windows/AD → FSx Windows; HPC → FSx Lustre; Ephemeral → Instance Store.
5. **DynamoDB RCU/WCU tính toán** — chắc chắn có 1-2 câu.
6. **DynamoDB LSI vs GSI**: LSI tạo với table, share throughput; GSI tạo bất kỳ, throughput riêng, throttle ảnh hưởng main table khi write.
7. **DynamoDB FilterExpression & ProjectionExpression KHÔNG giảm RCU**.
8. **DAX**: cache cho DynamoDB, μs latency. Đừng lẫn với ElastiCache.
9. **RDS Multi-AZ KHÔNG scale read** — chỉ HA.
10. **Aurora Global Database** cho DR cross-region, RTO < 1 phút.
11. **ElastiCache**: Redis → HA + persistence; Memcached → simple + multi-thread.

---

> **Tiếp theo:** [Phần 3: Application Integration (SQS, SNS, EventBridge, Step Functions, Kinesis, API Gateway)](/posts/dva-c02-part3-app-integration)
{: .prompt-info }
