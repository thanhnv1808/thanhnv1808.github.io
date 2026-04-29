# DVA-C02 — Phần 3: Application Integration

> **Chuỗi tài liệu:** Phần 1 (Compute) → Phần 2 (Storage & DB) → **Phần 3 (App Integration)** → Phần 4 (Security) → Phần 5 (Deploy & Monitor) → Phần 6 (Networking + Tips).

---

## 1. Amazon SQS (⭐ CỰC QUAN TRỌNG)

### 1.1 Standard vs FIFO — THUỘC LÒNG

| | **Standard** | **FIFO** |
|---|---|---|
| Throughput | **Unlimited** | 300 TPS (3,000 với batching), 3,000 TPS không batch với High Throughput mode |
| Ordering | Best-effort | **Strict order trong MessageGroupId** |
| Delivery | **At-least-once** (có thể duplicate) | **Exactly-once** processing |
| Queue name | Bất kỳ | Phải kết thúc **`.fifo`** |
| Deduplication | ❌ | ✅ (5 phút, qua MessageDeduplicationId hoặc content hash) |

### 1.2 Các thông số cần nhớ

| Thông số | Giá trị |
|---|---|
| **Message size tối đa** | **256 KB** (lớn hơn → dùng **S3 Extended Library**) |
| **Message retention** | 1 phút – **14 ngày** (mặc định 4 ngày) |
| **Visibility Timeout** | 0s – **12 giờ** (mặc định 30s) |
| **Delivery Delay** | 0s – **15 phút** (delay queue) |
| **Long polling (ReceiveMessageWaitTime)** | 0s – **20s** (khuyến nghị dùng để giảm empty response & cost) |
| **Batch Receive/Send/Delete** | Tới **10 messages** / request |
| **DLQ maxReceiveCount** | 1 – 1000 |

### 1.3 Visibility Timeout — **BẪY KINH ĐIỂN**

- Khi consumer `ReceiveMessage` → message trở thành **invisible** trong Visibility Timeout.
- Consumer xử lý xong → `DeleteMessage`.
- **Nếu Visibility Timeout hết mà chưa delete → message quay lại queue** → bị xử lý lại → duplicate!
- Giải pháp: consumer gọi `ChangeMessageVisibility` để extend nếu xử lý lâu hơn.

> 💡 **Câu hỏi thi:** "Consumer xử lý lâu hơn visibility timeout → hậu quả?" → **Message bị xử lý nhiều lần**. Giải pháp: tăng VT hoặc gọi ChangeMessageVisibility.

### 1.4 Dead-Letter Queue (DLQ)

- Queue riêng nhận message không xử lý được sau **maxReceiveCount** lần.
- DLQ của Standard phải là **Standard**, FIFO phải là **FIFO** (matching type).
- **Retention của DLQ nên lớn hơn source** để debug.
- **DLQ redrive**: có feature redrive message từ DLQ về source queue để retry.

### 1.5 Long Polling vs Short Polling

- **Short polling** (default, WaitTimeSeconds=0): return ngay, có thể empty → tốn API call.
- **Long polling** (WaitTimeSeconds > 0, max 20s): chờ tới khi có message hoặc hết timeout → **giảm cost & latency**.
- Có thể set ở queue level hoặc per `ReceiveMessage` call.

### 1.6 FIFO — MessageGroupId & Deduplication

- **MessageGroupId**: messages cùng group → xử lý tuần tự. Group khác → parallel.
- **MessageDeduplicationId**: nếu 2 message cùng DedupId gửi trong 5 phút → AWS chỉ giữ 1. Nếu không provide, SQS dùng hash nội dung (yêu cầu bật Content-based dedup).

### 1.7 SQS Extended Client (Java)

- Message > 256 KB → body lưu vào **S3**, SQS chỉ giữ reference.
- Tự động handle put/get.

### 1.8 SQS + Lambda

- Lambda **poll** queue (event source mapping).
- **Batch size**: 1–10,000 messages (Standard), 1–10 (FIFO).
- **Batch window**: chờ gom thêm tới 300s.
- **Partial batch response**: Lambda có thể báo 1 số message fail → chỉ message đó quay lại queue (tránh toàn batch retry).
- **Scaling**: Lambda tự scale theo queue depth (5 concurrent instances / batch window mỗi phút, tối đa 1000 concurrent consumers).

### 1.9 SQS + Auto Scaling (EC2)

- Dùng CloudWatch metric **`ApproximateNumberOfMessagesVisible`** làm trigger cho ASG.
- Hoặc **Backlog per Instance** custom metric để chính xác hơn.

### 1.10 SQS Access Policy

- **Resource-based policy** trên queue (ngoài IAM user/role).
- Use case: cross-account, SNS gửi tới SQS (cần grant permission cho SNS principal).

---

## 2. Amazon SNS (⭐ QUAN TRỌNG)

### 2.1 Khái niệm

- **Pub/Sub** service: Topic → nhiều Subscriber nhận bản copy.
- Subscriber types: **SQS, Lambda, HTTP/HTTPS, Email, SMS, Mobile Push, Kinesis Data Firehose, Application endpoint**.
- 1 topic có thể tới **12.5 triệu subscriber**; tài khoản có thể có 100,000 topics.

### 2.2 SNS vs SQS — Pattern chính

| | **SNS** | **SQS** |
|---|---|---|
| Pattern | Push (pub/sub, fan-out) | Pull (queue) |
| 1 message → N consumer | ✅ | ❌ (1 consumer/message) |
| Persistence | Không (phải có subscriber tại thời điểm gửi) | Có (tới 14 ngày) |
| Ordering | FIFO topic có | FIFO queue có |

### 2.3 Fan-out Pattern — SNS → SQS

- SNS publish 1 lần → nhiều SQS queue cùng nhận copy.
- Mỗi service có queue riêng → xử lý độc lập, có retention, DLQ riêng.
- **Rất phổ biến trong đề thi** — khi cần "nhiều service xử lý độc lập cùng 1 event".

### 2.4 SNS Message Filtering

- Subscription có **FilterPolicy** (JSON) → chỉ subscriber match mới nhận message.
- Filter theo attribute hoặc message body.
- Giảm work cho consumer không cần message đó.

### 2.5 SNS FIFO Topic

- Ordering + deduplication như SQS FIFO.
- Chỉ subscribe được từ **SQS FIFO queue**.

### 2.6 SNS Message Size & Retries

- Max **256 KB** (giống SQS).
- **Retry policy cho HTTP/S**: tới 3 lần default, configurable. Có DLQ cho SNS (SQS).

### 2.7 Mobile Push

- **FCM** (Android), **APNS** (iOS), **ADM** (Amazon), **Baidu**, **WNS** (Windows).

---

## 3. Amazon Kinesis

### 3.1 Bốn dịch vụ Kinesis — PHÂN BIỆT RÕ

| | **Data Streams** | **Data Firehose** | **Data Analytics** | **Video Streams** |
|---|---|---|---|---|
| Mục đích | Real-time data streaming | Load vào destination | Query real-time với SQL/Flink | Video streaming |
| Managed | Shard management tùy chọn | Fully managed | Fully managed | Fully managed |
| Retention | 1 – 365 ngày | Không (chuyển tiếp) | — | Configurable |
| Latency | ~200ms | **Near real-time** (60s buffer tối thiểu) | Tới giây | — |

### 3.2 Kinesis Data Streams (KDS) — chi tiết

- **Shard**: đơn vị throughput.
  - **Write**: 1 MB/s hoặc 1,000 records/s per shard.
  - **Read (Classic)**: 2 MB/s total share giữa consumer.
  - **Read (Enhanced Fan-out)**: 2 MB/s per consumer per shard.
- **Capacity modes**:
  - **Provisioned**: tự quản shard count.
  - **On-Demand**: AWS tự scale (2022+).
- **Producer**: PutRecord/PutRecords API, KPL, Kinesis Agent.
- **Consumer**: SDK (`GetRecords`), KCL, Lambda, Firehose, Data Analytics.
- **Partition Key** quyết định shard → hot partition nếu skew.
- **Record size**: max **1 MB**.
- **Retention**: 1-365 ngày.

> ⚠️ **Bẫy:** "Consumer bị throttle khi nhiều app cùng đọc cùng shard?" → dùng **Enhanced Fan-Out** (mỗi consumer có 2 MB/s riêng, nhưng tốn tiền hơn).

### 3.3 Kinesis Data Firehose

- **Near-real-time** (buffer tối thiểu **60s hoặc 1 MB**, cấu hình đến 900s / 128 MB).
- Destinations: **S3, Redshift (qua COPY), OpenSearch, Splunk, HTTP endpoints** (Datadog, MongoDB, New Relic...).
- Có thể **transform** data bằng Lambda trước khi ghi.
- Có thể **convert format** (JSON → Parquet/ORC) khi ghi vào S3 (cho Redshift/Athena hiệu quả hơn).
- **Compression, encryption** built-in.

> 💡 **So sánh KDS vs Firehose:**
> - KDS: real-time (< 1s), custom consumer logic, retention dài, phải quản shard.
> - Firehose: near-real-time, zero admin, chỉ load vào destination có sẵn.

### 3.4 Kinesis Data Analytics

- **SQL** (cũ) hoặc **Flink (Managed Service for Apache Flink)** — query stream real-time.
- Input từ KDS/Firehose, output sang KDS/Firehose/Lambda.

### 3.5 Kinesis Client Library (KCL)

- Java library handle:
  - Shard discovery, load balancing giữa consumer (1 consumer per shard).
  - Checkpoint qua **DynamoDB** → KCL yêu cầu DynamoDB table.
- Nếu scale up consumer > shard count → các consumer dư sẽ idle.

---

## 4. Amazon EventBridge (CloudWatch Events evolved)

### 4.1 Khái niệm

- **Event bus**: default (AWS services) hoặc custom (apps) hoặc partner (SaaS như Zendesk).
- Route event đến target qua **Rules** (pattern match hoặc schedule cron/rate).
- Targets: Lambda, SQS, SNS, Step Functions, Kinesis, ECS task, API Gateway, API destination (HTTP), nhiều dịch vụ khác.

### 4.2 EventBridge vs SNS

| | **EventBridge** | **SNS** |
|---|---|---|
| Filter pattern | **Mạnh, JSON matching** | FilterPolicy đơn giản hơn |
| Schedule | **Cron/rate** | ❌ |
| Target types | Rất nhiều AWS service | Chủ yếu SQS/Lambda/HTTP/Email/SMS |
| Throughput | Thấp hơn SNS | Cao |
| Replay | Có (archive) | ❌ |
| Schema Registry | ✅ | ❌ |

### 4.3 EventBridge Schema Registry

- Tự discover schema từ events, code binding cho Java/Python/TypeScript.

### 4.4 EventBridge Scheduler (mới)

- Thay CloudWatch Events cron — scalable, one-time hoặc recurring, **tới 14 ngày delay**.

### 4.5 EventBridge Pipes

- Point-to-point: Source → (Filter → Enrich) → Target, không cần Lambda glue.
- Sources: SQS, Kinesis, DynamoDB Streams, Kafka, MSK.

---

## 5. AWS Step Functions

### 5.1 Khái niệm

- **Serverless workflow** với state machine (JSON: **Amazon States Language**).
- Visual workflow trong console.

### 5.2 Standard vs Express Workflows

| | **Standard** | **Express** |
|---|---|---|
| Duration tối đa | **1 năm** | **5 phút** |
| Execution rate | 2,000 start/s | 100,000 start/s |
| Execution semantics | Exactly-once | At-least-once (Async) hoặc At-most-once (Sync) |
| Pricing | Per transition | Per duration & invocation |
| History retention | 90 ngày trong console | CloudWatch Logs |
| Use case | Long-running, human approval, ETL | Event processing, IoT, streaming |

### 5.3 State types

| Type | Mô tả |
|---|---|
| **Task** | Thực thi work (Lambda, ECS, SNS, ...) |
| **Choice** | Branching logic |
| **Parallel** | Chạy nhiều branch đồng thời |
| **Map** | Iterate array, xử lý song song (Inline hoặc Distributed Map tới 10k item concurrent) |
| **Wait** | Delay (giây hoặc đến timestamp) |
| **Pass** | Truyền data qua, có thể inject |
| **Succeed / Fail** | Kết thúc |

### 5.4 Error Handling

- **Retry** trong state: `ErrorEquals`, `IntervalSeconds`, `MaxAttempts`, `BackoffRate`.
- **Catch**: route tới state khác khi lỗi.

### 5.5 Service Integration Patterns

1. **Request/Response** (default): gọi service, lấy ngay response.
2. **Run a Job (.sync)**: chờ job hoàn tất (ECS task, Batch job...).
3. **Wait for Callback (.waitForTaskToken)**: gửi task token, chờ external system call `SendTaskSuccess/Failure` → dùng cho human approval, workflow async với service ngoài.

### 5.6 Activities

- Custom worker code, dùng `GetActivityTask`. Ít dùng so với Lambda.

---

## 6. Amazon API Gateway (⭐ QUAN TRỌNG)

### 6.1 3 loại API

| | **REST API** | **HTTP API** | **WebSocket API** |
|---|---|---|---|
| Features | Full (API keys, usage plans, request validation, X-Ray...) | Gọn nhẹ, nhanh, **rẻ 71%** | 2-way real-time |
| Use case | Legacy, enterprise | Microservice, serverless mới | Chat, notifications, game |
| Latency | Cao hơn | Thấp hơn | — |
| Auth | IAM, Lambda Authorizer, Cognito | IAM, JWT (Cognito/OIDC), Lambda Authorizer | IAM, Lambda Authorizer |

> 💡 **Default mới:** HTTP API trừ khi cần feature chỉ có ở REST (usage plans, API keys, private integration với VPC Link legacy, request/response transformation mạnh, ...).

### 6.2 Endpoint Types

- **Edge-optimized**: qua CloudFront (default) — global low latency.
- **Regional**: trong 1 region, client cùng region hoặc tự dùng CloudFront.
- **Private**: chỉ accessible trong VPC qua VPC Endpoint (interface).

### 6.3 Integration Types

- **Lambda proxy** (AWS_PROXY): Lambda nhận full request, trả full response.
- **Lambda non-proxy**: cấu hình mapping template.
- **HTTP proxy** / **HTTP**.
- **AWS Service** (VD: gọi SQS, DynamoDB trực tiếp).
- **Mock**: trả static response.
- **VPC Link**: REST API → ALB/NLB trong VPC private. HTTP API → tới VPC via ALB/NLB/Cloud Map.

### 6.4 Stages & Deployments

- **Stage** (dev, prod): deployment snapshot của API.
- **Stage Variables**: biến gắn stage → trỏ Lambda alias khác nhau, config khác nhau.
- **Canary Deployments**: split traffic % giữa canary & stable trong cùng stage.

### 6.5 Throttling & Usage Plans

- **Account-level throttling**: mặc định 10,000 req/s, burst 5,000.
- **Stage/Method-level throttling**.
- **Usage Plans**: gắn **API Keys** → rate limit per key.
- **Throttling error**: `429 Too Many Requests`.

> ⚠️ **Bẫy:** "Tại sao user bị 429?" → account-level limit, usage plan, hoặc stage throttling đang giới hạn.

### 6.6 Caching

- Cache ở **stage level** (cache size 0.5 GB – 237 GB).
- TTL mặc định 300s (0-3600s).
- Có thể invalidate với header `Cache-Control: max-age=0` (cần IAM permission `InvalidateCache`).
- Cache per-key (path, query, header).

### 6.7 Authentication

1. **IAM**: SigV4. Cho service-to-service trong AWS.
2. **Lambda Authorizer** (cũ: Custom Authorizer): Lambda trả IAM policy. Token-based hoặc Request-based. **Có thể cache policy** (mặc định 300s).
3. **Cognito User Pools**: dùng JWT từ Cognito, API Gateway verify tự động. **Không kiểm tra authorization (chỉ authentication)** — logic phân quyền ở Lambda.
4. **Resource Policy**: ai (IP, account, VPC endpoint) gọi được API.

### 6.8 Request/Response transformations

- **Mapping Templates** (Velocity VTL) — chỉ REST API:
  - Transform REST → SOAP.
  - Rename/filter body fields.
  - Tách request thành param cho backend.

### 6.9 CORS

- Browser OPTIONS preflight request → API Gateway trả header `Access-Control-Allow-Origin/-Methods/-Headers`.
- REST API: enable trong console per resource.
- HTTP API: CORS config cấp API.

### 6.10 Monitoring

- **CloudWatch Metrics**: `4XXError`, `5XXError`, `IntegrationLatency`, `Latency`, `CacheHitCount`, `CacheMissCount`, `Count`.
- **CloudWatch Logs**: execution logs (per-request), access logs (log format tùy chỉnh).
- **X-Ray**: distributed tracing.

### 6.11 API Gateway limits

- Timeout integration: **29 giây** (hard limit) — Lambda/HTTP backend phải trả trong 29s.
- Max payload: **10 MB** (REST), **10 MB** (HTTP API).
- **WebSocket**: idle connection timeout 10 phút.

> ⚠️ **Bẫy kinh điển:** "Lambda chạy 5 phút cho long-running job → gọi qua API Gateway" → **LỖI timeout 29s**. Giải pháp: async invocation pattern (API Gateway → SQS → Lambda hoặc trả 202, poll status).

---

## 7. AWS AppSync (GraphQL)

### 7.1 Khái niệm

- Managed **GraphQL & Pub/Sub** API.
- Data sources: DynamoDB, RDS (Aurora Serverless), Lambda, HTTP, OpenSearch.
- **Real-time subscriptions** qua WebSocket.
- Offline sync với **Amplify DataStore**.

### 7.2 Resolvers

- **Unit resolvers**: single data source.
- **Pipeline resolvers**: chain nhiều function.
- Viết bằng **VTL** hoặc **JavaScript** (mới).

### 7.3 Authentication

- API Key, IAM, Cognito User Pools, OIDC, Lambda.

---

## 8. Amazon MQ

- Managed **Apache ActiveMQ / RabbitMQ**.
- Dùng khi migration từ hệ thống on-prem cần **protocol chuẩn**: AMQP, MQTT, OpenWire, STOMP, WSS, JMS/NMS.
- **Không scale bằng SQS/SNS** — chỉ dùng khi bắt buộc vì protocol compatibility.

> 💡 **Quyết định:** Hệ thống mới → **SQS + SNS**. Migration cần AMQP/MQTT/JMS → **Amazon MQ**.

---

## 9. Amazon MSK (Managed Streaming for Apache Kafka)

- Managed **Kafka** (cluster với brokers).
- **MSK Serverless**: không quản broker.
- So với KDS: Kafka mạnh hơn cho workload cực cao & ecosystem, KDS đơn giản hơn.

---

## 🎯 Tip ôn thi Phần 3

1. **SQS Visibility Timeout bẫy #1**:
   - VT quá ngắn + consumer xử lý chậm → **duplicate processing**.
   - Giải pháp: tăng VT, hoặc gọi `ChangeMessageVisibility` dynamic.

2. **SQS Long Polling luôn khuyên dùng** — giảm empty response, giảm cost.

3. **DLQ retention > source retention** để có thời gian debug.

4. **SQS FIFO: 300 TPS** (3,000 với batching). Cần cao hơn → bật **High Throughput** mode (3,000 base, 30,000 batch).

5. **Fan-out pattern**: SNS topic → nhiều SQS queue → nhiều consumer xử lý độc lập. **Cực phổ biến trong đề**.

6. **SNS Filter Policy** — để tránh Lambda fan-out rồi filter bằng code (lãng phí).

7. **Kinesis Data Streams vs Firehose**:
   - **KDS**: real-time <1s, custom consumer, retention, quản shard.
   - **Firehose**: near-real-time 60s+, zero admin, chỉ load vào destination.

8. **Kinesis Enhanced Fan-Out** khi nhiều app đọc cùng stream → tránh throttle 2 MB/s shared.

9. **Step Functions Standard vs Express**:
   - Long-running, stateful → **Standard** (1 năm).
   - Short, high-volume → **Express** (5 phút, 100k/s).

10. **Step Functions .waitForTaskToken** — pattern cho human approval hoặc callback từ external.

11. **API Gateway timeout 29s cứng** — long-running phải async.

12. **API Gateway caching** — invalidate với `Cache-Control: max-age=0` header (cần IAM permission).

13. **Cognito User Pool Authorizer** chỉ authenticate, không authorize → logic quyền ở Lambda.

14. **HTTP API rẻ hơn REST API 71%** — default chọn HTTP API trừ khi cần feature REST-only.

15. **EventBridge cho cron/schedule**, không dùng SNS/SQS.

16. **Amazon MQ** chỉ khi migration cần AMQP/MQTT/JMS. Workload mới luôn SQS/SNS/EventBridge.

---

**→ Sẵn sàng tiếp tục với Phần 4 (Security & Identity)?**
