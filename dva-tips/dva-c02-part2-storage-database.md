# DVA-C02 — Phần 2: Storage & Database

> **Chuỗi tài liệu:** Phần 1 (Compute) → **Phần 2 (Storage & DB)** → Phần 3 (App Integration) → Phần 4 (Security) → Phần 5 (Deploy & Monitor) → Phần 6 (Networking + Tips).

---

## 1. Amazon S3 (⭐ CỰC QUAN TRỌNG — xuất hiện nhiều câu)

### 1.1 Khái niệm cơ bản

- **Bucket** (globally unique name) → **Objects** (key = full path, value = nội dung).
- **Max object size: 5 TB**. Upload 1 request tối đa **5 GB** → trên đó phải **Multipart Upload**.
- **Khuyến nghị Multipart Upload khi file > 100 MB**. Bắt buộc khi > 5 GB.
- **Eventual consistency đã bị loại bỏ** (từ Dec 2020) → S3 giờ **Strong read-after-write consistency** cho mọi operation (PUT/GET/DELETE/LIST).

### 1.2 Storage Classes — THUỘC LÒNG BẢNG NÀY

| Class | Availability | Durability | Min storage | Retrieval fee | Use case |
|---|---|---|---|---|---|
| **S3 Standard** | 99.99% | 11 nines | — | Không | Frequent access |
| **S3 Intelligent-Tiering** | 99.9% | 11 nines | — | Không (phí monitoring) | Unknown/changing access pattern |
| **S3 Standard-IA** | 99.9% | 11 nines | 30 ngày | Có (per GB) | Infrequent, cần truy cập nhanh |
| **S3 One Zone-IA** | 99.5% | 11 nines | 30 ngày | Có | IA nhưng data tái tạo được (1 AZ only) |
| **S3 Glacier Instant Retrieval** | 99.9% | 11 nines | 90 ngày | Có | Archive + truy cập tức thì (ms) |
| **S3 Glacier Flexible Retrieval** | 99.99% | 11 nines | 90 ngày | Có | Archive, retrieval: Expedited (1-5ph), Standard (3-5h), Bulk (5-12h) |
| **S3 Glacier Deep Archive** | 99.99% | 11 nines | 180 ngày | Có | Archive rẻ nhất, retrieval: Standard (12h), Bulk (48h) |

> 💡 **Bẫy thi thường gặp:**
> - "Cheapest cho data hiếm khi truy cập nhưng cần lấy trong vài giây?" → **Glacier Instant Retrieval**.
> - "Data cần 1 lần/năm, lấy trong vài phút?" → **Glacier Flexible, Expedited**.
> - "Không biết pattern truy cập?" → **Intelligent-Tiering**.

### 1.3 Lifecycle Rules

- **Transition actions**: chuyển class (Standard → IA → Glacier...).
- **Expiration actions**: xóa object/version sau X ngày.
- Áp dụng cho prefix, tag, hoặc cả bucket.
- **Minimum storage duration** phải được tôn trọng — transition sớm hơn sẽ bị charge full period.

### 1.4 Versioning

- Bật/tắt ở **bucket level**. Khi tắt → thành `Suspended` (version cũ vẫn còn).
- **Delete marker**: "xóa" ở bucket versioning chỉ thêm delete marker. Xóa vĩnh viễn phải chỉ định `versionId`.
- **MFA Delete**: bảo vệ xóa version, chỉ bucket owner (root account) bật được qua CLI.

### 1.5 Encryption — 4 loại

| Loại | Key quản bởi | Header request |
|---|---|---|
| **SSE-S3** (mặc định từ 2023) | AWS (AES-256) | `x-amz-server-side-encryption: AES256` |
| **SSE-KMS** | AWS KMS | `x-amz-server-side-encryption: aws:kms` |
| **SSE-C** | Customer cung cấp key mỗi request | Header kèm key |
| **Client-side** | Customer | Encrypt trước khi upload |

> ⚠️ **SSE-KMS**: mỗi request là 1 call tới KMS → dễ bị throttle. Dùng **S3 Bucket Keys** để giảm KMS call (tạo data key ở bucket level, giảm 99% chi phí KMS).

- **Bucket Policy enforce encryption**: deny `PutObject` nếu không có header `x-amz-server-side-encryption`.

### 1.6 Bucket Policy vs ACL vs IAM

- **IAM policy**: attach vào user/role → kiểm soát từ phía principal.
- **Bucket Policy**: JSON gắn vào bucket → cross-account access, public access, enforce encryption/HTTPS.
- **ACL**: cũ, legacy. AWS khuyến nghị tắt ACL (Object Ownership = **Bucket Owner Enforced**).
- **Block Public Access**: settings cấp account/bucket để chặn public ngay cả khi policy cho phép.

### 1.7 Pre-signed URLs

- Cho phép user tạm truy cập object mà không cần IAM credentials.
- Tạo bằng SDK hoặc CLI — URL có **SignatureVersion, expiration**.
- Kế thừa quyền của **người tạo URL** → user tạo cần có quyền tương ứng.
- Default expiration: CLI = 3600s, SDK = 15 phút (tuỳ ngôn ngữ).

### 1.8 CORS

- Khi web app ở domain A gọi S3 bucket ở domain B → cần CORS config trên bucket.
- Config XML/JSON với `AllowedOrigins`, `AllowedMethods`, `AllowedHeaders`.

### 1.9 Replication

- **CRR** (Cross-Region Replication): cho DR, compliance, latency.
- **SRR** (Same-Region Replication): log aggregation, env sync.
- Yêu cầu: **Versioning bật ở cả source và destination**, IAM role.
- **KHÔNG replicate retroactively** (default) — có thể bật **S3 Batch Replication** cho data cũ.
- **Delete marker replication** có thể bật/tắt. Xóa vĩnh viễn (với versionId) KHÔNG replicate.
- Replicate chỉ 1 hop: A→B→C, object từ A KHÔNG tự tới C (trừ khi bật multi-destination rules).

### 1.10 Event Notifications

- Events: `s3:ObjectCreated:*`, `ObjectRemoved:*`, `ObjectRestore:*`, `Replication:*`.
- Targets: **SNS, SQS, Lambda, EventBridge**.
- EventBridge là targeting mới nhất, filter mạnh hơn.

### 1.11 Performance — multipart, Transfer Acceleration, Byte-range fetches

- **Multipart Upload**: parallel, resumable, faster. Dùng `CompleteMultipartUpload` khi xong, `AbortMultipartUpload` để dọn (lifecycle rule nên tự abort sau X ngày để tiết kiệm).
- **Transfer Acceleration**: upload qua CloudFront edge → S3 bucket (dùng URL `bucket.s3-accelerate.amazonaws.com`).
- **Byte-range fetches**: parallel GET theo range → download lớn nhanh.
- **Prefix sharding**: S3 tự scale, giới hạn **3,500 PUT/COPY/POST/DELETE + 5,500 GET/HEAD per second per prefix**. Chia prefix để tăng throughput.

### 1.12 S3 Object Lock & Glacier Vault Lock

- **Object Lock**: WORM (Write Once Read Many). 2 mode:
  - **Governance**: user có quyền đặc biệt mới override được.
  - **Compliance**: KHÔNG ai override được, kể cả root.
- **Legal Hold**: block xóa/sửa đến khi remove hold.
- **Glacier Vault Lock**: áp dụng policy với S3 Glacier vault (legacy API).

### 1.13 S3 Select & Glacier Select

- Query bằng **SQL đơn giản** trên CSV/JSON/Parquet → chỉ tải data cần → tiết kiệm băng thông.
- Thay cho việc download toàn bộ file.

### 1.14 Access Points & Multi-Region Access Points

- **Access Points**: DNS endpoint riêng cho từng use case/team, policy riêng.
- **Multi-Region Access Points**: 1 endpoint global, route tới bucket nhanh nhất, có failover.

### 1.15 Static Website Hosting

- Bật ở bucket, set index/error document.
- **Phải bật Block Public Access = OFF + bucket policy cho phép `s3:GetObject` cho `Principal: *`**.
- URL format: `bucket-name.s3-website-<region>.amazonaws.com`.
- Dùng CloudFront phía trước cho HTTPS, custom domain.

---

## 2. Amazon EBS (Elastic Block Store)

### 2.1 Volume Types

| Type | IOPS | Use case | Ghi chú |
|---|---|---|---|
| **gp3** (mặc định mới) | 3,000-16,000 | General purpose | IOPS & throughput độc lập với size → **gp3 rẻ hơn gp2 ~20%** |
| **gp2** | 3 IOPS/GB, burst 3,000 | General purpose | IOPS gắn với size |
| **io1 / io2** | Tới 64,000 | High-performance DB | io2 Block Express tới 256,000 IOPS |
| **st1** (HDD) | Throughput-optimized | Big data, log | **KHÔNG boot volume** |
| **sc1** (HDD) | Cold HDD | Infrequent access | **KHÔNG boot volume** |

### 2.2 Đặc tính quan trọng

- **AZ-specific**: volume bị khóa trong 1 AZ. Muốn move AZ → **snapshot → restore ở AZ khác**.
- **Multi-Attach**: chỉ **io1/io2** hỗ trợ attach tới nhiều EC2 trong cùng AZ (max 16), chỉ Nitro instances, filesystem phải là cluster-aware (không phải ext4/xfs thường).
- **Snapshot**: incremental, lưu ở S3 (managed). Có thể copy sang region khác.
- **Snapshot Archive**: giảm 75% chi phí, restore chậm (24-72h).
- **Recycle Bin**: recover snapshot/AMI đã xóa (retention period).
- **EBS Encryption**: encrypt at-rest, in-transit giữa volume và instance. Snapshot của encrypted volume cũng encrypted. Encrypt volume cũ: snapshot → copy snapshot với encryption → restore.

### 2.3 Instance Store vs EBS

- **Instance Store**: ephemeral, physical disk attach với host. **Mất data khi stop/terminate**. Cực nhanh (NVMe local). Dùng cho cache, buffer, scratch.
- **EBS**: persistent, network-attached. Mặc định cho production.

### 2.4 Tips

- **Optimize cost**: gp3 thay gp2 khi có thể.
- **Resize volume**: dùng `modify-volume`, sau đó extend filesystem trong OS. **KHÔNG downtime**.
- **Snapshot cross-region**: copy sang region khác để DR.

---

## 3. Amazon EFS (Elastic File System)

- **NFS v4 shared file system**, multi-AZ, scale tự động.
- **Mount từ nhiều EC2 đồng thời** (POSIX).
- **KHÔNG dùng với Windows** (Windows dùng **FSx**).
- **Performance modes**:
  - **General Purpose** (default): low latency.
  - **Max I/O**: higher throughput, higher latency → big data, parallel.
- **Throughput modes**: Bursting (default), Provisioned, Elastic.
- **Storage classes**:
  - **Standard** (multi-AZ).
  - **One Zone** (rẻ hơn, 1 AZ).
  - Lifecycle management tự chuyển sang **IA** / **Archive** sau X ngày không truy cập.

> 💡 **So sánh:**
> - **EBS**: 1 EC2 attach (trừ Multi-Attach), per-AZ, block storage.
> - **EFS**: nhiều EC2, multi-AZ, Linux NFS.
> - **FSx for Windows**: SMB, Active Directory — thay file server Windows.
> - **FSx for Lustre**: HPC, ML training, tốc độ cực cao.
> - **Instance Store**: ephemeral, local.

---

## 4. Amazon DynamoDB (⭐ CỰC QUAN TRỌNG)

### 4.1 Khái niệm cơ bản

- NoSQL key-value + document, **fully managed, serverless**.
- Latency ms, **multi-AZ** mặc định.
- **Table** → **Items** → **Attributes**.
- **Primary Key**: 2 lựa chọn:
  - **Partition Key (PK)** duy nhất.
  - **Partition Key + Sort Key** (composite).

### 4.2 Capacity modes

| Mode | Khi nào dùng |
|---|---|
| **Provisioned** | Traffic đoán được, muốn tiết kiệm với Reserved Capacity |
| **On-Demand** | Traffic không đoán được, spiky, dev |

- **RCU/WCU** (Read/Write Capacity Units):
  - 1 **WCU** = 1 KB/s write.
  - 1 **RCU** = 1 strongly consistent read 4 KB/s HOẶC 2 eventually consistent read 4 KB/s HOẶC 0.5 transactional read 4 KB/s.
- **Auto Scaling** với Provisioned: target utilization %.

> 💡 **Công thức thi hay hỏi:**
> - Write 10 items/s, mỗi item 4.5 KB → WCU = 10 × ceil(4.5/1) = **10 × 5 = 50 WCU**.
> - Strongly consistent read 10 items/s, mỗi item 6 KB → RCU = 10 × ceil(6/4) = **10 × 2 = 20 RCU**.
> - Eventually consistent cùng case → **10 RCU**.
> - Transactional read cùng case → **40 RCU**.

### 4.3 Các API chính

- **PutItem**: insert/replace.
- **UpdateItem**: update attribute, hỗ trợ **atomic counter** (`ADD`).
- **DeleteItem**: delete.
- **Conditional writes**: `ConditionExpression` → chỉ write khi điều kiện true → chống race condition.
- **BatchWriteItem**: tới 25 items, 16 MB.
- **BatchGetItem**: tới 100 items, 16 MB.
- **GetItem**: single item by PK.
- **Query**: theo PK (bắt buộc) + SK (optional). Nhanh.
- **Scan**: đọc toàn bảng → chậm, tốn RCU. Dùng **Parallel Scan** để chia segment.

### 4.4 Index

- **Local Secondary Index (LSI)**:
  - Cùng PK, **SK khác**.
  - **Tạo khi create table** — không tạo sau được.
  - Max 5 LSI/table.
  - Share RCU/WCU với table.
- **Global Secondary Index (GSI)**:
  - PK mới (+ SK tuỳ chọn).
  - Tạo/xóa bất cứ lúc nào.
  - Max 20 GSI/table.
  - **Có RCU/WCU riêng** → nếu GSI bị throttle, **main table cũng bị throttle write**!

> ⚠️ **Bẫy thi:** "GSI throttle → chuyện gì xảy ra với main table?" → **Write tới main table cũng bị throttle**.

### 4.5 DynamoDB Streams

- Capture **INSERT/MODIFY/REMOVE** events.
- Retention **24h**.
- **StreamViewType**:
  - `KEYS_ONLY`
  - `NEW_IMAGE`
  - `OLD_IMAGE`
  - `NEW_AND_OLD_IMAGES`
- Target: **Lambda**, **Kinesis Data Streams** (via Kinesis Adapter).

### 4.6 TTL (Time To Live)

- Attribute kiểu **Number**, giá trị **epoch seconds**.
- Item tự xóa sau TTL → KHÔNG tốn WCU.
- Delete xảy ra trong vòng 48h.

### 4.7 Transactions

- `TransactWriteItems` / `TransactGetItems` — ACID.
- **Gấp 2 lần capacity** (1 write trong transaction = 2 WCU).
- Tới 100 items hoặc 4 MB/transaction.

### 4.8 DynamoDB Accelerator (DAX)

- **Fully managed in-memory cache** cho DynamoDB.
- Microsecond latency cho read (ms → μs).
- Transparent — không đổi code, chỉ đổi endpoint.
- Cluster trong VPC, 1 primary + replicas.
- **KHÁC ElastiCache**: DAX chuyên cho DynamoDB, ElastiCache general purpose.

### 4.9 Global Tables

- Multi-region, active-active replication.
- Yêu cầu Streams bật.
- Latency ~1s cross-region.

### 4.10 Backup & Restore

- **On-demand backup**: full backup, không ảnh hưởng performance.
- **PITR (Point-In-Time Recovery)**: restore về bất kỳ thời điểm nào **35 ngày gần nhất**.

### 4.11 Operators & Optimistic Locking

- **Optimistic Locking**: dùng `ConditionExpression` check version attribute trước update.
- **Pagination**: `LastEvaluatedKey` → `ExclusiveStartKey` cho request tiếp theo.
- **FilterExpression**: apply **sau Query/Scan** → KHÔNG giảm RCU tiêu thụ.
- **ProjectionExpression**: giảm data trả về nhưng cũng KHÔNG giảm RCU.

### 4.12 Tips & bẫy thường gặp

1. **Hot partition**: PK có distribution kém → bottleneck. Dùng **random/composite PK** để phân tán.
2. **Query > Scan** — luôn thiết kế để query qua PK/GSI.
3. **Strongly consistent read tốn gấp đôi RCU** so với eventually consistent.
4. **DAX cho read-heavy**, **ElastiCache cho compute caching**.
5. **Conditional write** để tránh lost update trong concurrent write.

---

## 5. Amazon RDS (Relational Database Service)

### 5.1 Các engine hỗ trợ

MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, **Aurora** (AWS proprietary).

### 5.2 Tính năng chính

- **Multi-AZ**: standby replica ở AZ khác, **sync replication**, **tự động failover**. CHỈ để HA, **KHÔNG scale read**.
- **Read Replicas**: tới 15, async replication. Dùng để scale read. Có thể cross-region, promote thành standalone.
- **Automated Backup**: daily snapshot + transaction log → PITR, retention 0-35 ngày.
- **Manual Snapshot**: giữ forever, share cross-account được.
- **Storage**: GP2/GP3/IO1/Magnetic. Auto scaling storage có thể bật.

> ⚠️ **Bẫy:** "Multi-AZ giúp scale read?" → **SAI**, chỉ HA. Dùng **Read Replica**.

### 5.3 Security

- Encryption at-rest qua KMS (khi create, KHÔNG encrypt sau được → phải snapshot → copy encrypted → restore).
- In-transit: SSL/TLS.
- IAM database authentication (MySQL, PostgreSQL): dùng token 15 phút thay password.

### 5.4 RDS Proxy

- Pool connection → giảm failover time, giảm connection overhead cho Lambda.
- Integrate với Secrets Manager / IAM auth.
- Không publicly accessible.

### 5.5 Parameter Groups & Option Groups

- **Parameter Group**: config engine (VD `max_connections`).
- **Option Group**: chức năng bổ sung (VD Oracle Native Network Encryption).

---

## 6. Amazon Aurora

### 6.1 Đặc tính

- MySQL & PostgreSQL compatible.
- **5x MySQL, 3x PostgreSQL** về performance.
- Storage auto-scale **10 GB → 128 TB**.
- 6 bản sao data qua **3 AZ** (4 write, 3 read quorum).
- **Self-healing**, continuous backup to S3.

### 6.2 Endpoints

- **Writer endpoint** (cluster endpoint): trỏ tới master.
- **Reader endpoint**: load-balance tới read replicas.
- **Custom endpoint**: subset of replicas cho workload cụ thể.
- **Instance endpoint**: trỏ 1 instance cụ thể.

### 6.3 Replica

- Tới **15 Aurora Replicas** (vs 5 MySQL replica trên RDS).
- Auto-failover **< 30s** thường.
- Cross-Region replicas cho DR.

### 6.4 Aurora Serverless

- **v1**: auto scale, phù hợp workload không thường xuyên. Cold start.
- **v2**: instant auto scale, giữ capacity min > 0 → không cold start (thực chất "warm").

### 6.5 Aurora Global Database

- 1 primary region + tới **5 secondary regions** (read-only).
- Replication latency **< 1s**.
- **Cross-region failover < 1 phút** (RTO).
- Dùng cho DR, low-latency read toàn cầu.

### 6.6 Aurora ML

- Tích hợp SageMaker, Comprehend trực tiếp từ SQL query.

### 6.7 Backtrack

- "Rewind" DB tới thời điểm trước (tới 72h) **mà KHÔNG restore** → KHÔNG cần snapshot, nhanh.
- Chỉ Aurora MySQL.

---

## 7. Amazon ElastiCache

### 7.1 Redis vs Memcached — THUỘC LÒNG

| | **Redis** | **Memcached** |
|---|---|---|
| Multi-AZ với auto failover | ✅ | ❌ |
| Replication | ✅ | ❌ |
| Persistence (AOF) | ✅ | ❌ |
| Backup/Restore | ✅ | ❌ |
| Data structures | List, Set, Hash, Sorted Set, Bitmap, ... | String only |
| Multi-threaded | ❌ (single thread) | ✅ |
| Pub/Sub | ✅ | ❌ |
| Transactions (MULTI/EXEC) | ✅ | ❌ |

> 💡 **Chọn:**
> - Cần HA, persistence, advanced data structures → **Redis**.
> - Cần cache đơn giản, multi-threaded, tiết kiệm → **Memcached**.

### 7.2 Cache strategies

1. **Lazy Loading** (Cache-Aside):
   - Read: check cache → miss → load từ DB → write cache.
   - Pros: chỉ cache data thực sự cần.
   - Cons: cache miss tốn 3 trips, data có thể stale.
2. **Write-Through**:
   - Write: write DB + write cache đồng thời.
   - Pros: data fresh.
   - Cons: cache có data không bao giờ đọc.
3. **TTL**: thường dùng chung với lazy loading.

### 7.3 Redis Auth & Encryption

- **Redis AUTH**: password token.
- **Encryption in-transit & at-rest**: hỗ trợ.
- **IAM Auth** cho Redis 7+.

---

## 8. Amazon Neptune, DocumentDB, QLDB, Timestream, Keyspaces — biết sơ

- **Neptune**: Graph DB (social, fraud, recommendation).
- **DocumentDB**: MongoDB-compatible.
- **QLDB**: ledger DB, immutable, cryptographically verifiable. **KHÁC blockchain** ở chỗ centralized.
- **Timestream**: time-series DB (IoT, monitoring).
- **Keyspaces**: Cassandra-compatible.

> DVA-C02 ít hỏi sâu, chủ yếu nhận biết use case.

---

## 9. Amazon Redshift

- Data warehouse, columnar, PB-scale.
- **Khác RDS**: OLAP không phải OLTP.
- **Redshift Spectrum**: query S3 trực tiếp không load.
- **COPY** để load data nhanh từ S3.
- KHÔNG multi-AZ (cho tới gần đây, hỏi ít trong DVA).

---

## 🎯 Tip ôn thi Phần 2

1. **S3 Storage Classes** — học thuộc min storage duration và use case. Glacier Instant Retrieval hay bị confuse với Standard-IA.

2. **S3 Encryption** — khi nào dùng SSE-KMS: cần audit & rotation, nhưng coi chừng **KMS throttling** → bật **S3 Bucket Keys**.

3. **S3 Strong Consistency** — từ 2020, mọi read-after-write đều strong. Đừng để bị lừa bởi tài liệu cũ.

4. **EBS vs EFS vs Instance Store vs FSx**:
   - 1 EC2 / block → EBS.
   - Nhiều Linux EC2 / shared → EFS.
   - Windows share / AD → FSx for Windows.
   - HPC ultra-fast → FSx for Lustre.
   - Ephemeral, ultra-fast local → Instance Store.

5. **DynamoDB RCU/WCU tính toán** — gần như chắc chắn có 1-2 câu. Nhớ:
   - Write: `ceil(size_KB)` per item.
   - Strongly consistent read: `ceil(size_KB / 4)`.
   - Eventually consistent: `ceil(size_KB / 4) / 2` → làm tròn lên.
   - Transactional: gấp đôi.

6. **DynamoDB LSI vs GSI**:
   - LSI: create với table, share throughput.
   - GSI: tạo bất kỳ, **throughput riêng**, throttle ảnh hưởng main table trên write.

7. **DynamoDB Streams + Lambda** — pattern hay bị hỏi để trigger action khi data thay đổi.

8. **DAX**: cache cho DynamoDB, μs latency, transparent. Đừng lẫn với ElastiCache.

9. **RDS Multi-AZ KHÔNG phải để scale read** — chỉ HA. Read Replica mới scale.

10. **Aurora Global Database** cho DR cross-region, RTO < 1 phút.

11. **ElastiCache**: Redis cho HA + persistence; Memcached cho simple + multi-thread.

12. **Pre-signed URL** — user tải/upload mà không cần IAM. Có thời hạn. Kế thừa quyền của creator.

---

**→ Sẵn sàng tiếp tục với Phần 3 (Application Integration)?**
