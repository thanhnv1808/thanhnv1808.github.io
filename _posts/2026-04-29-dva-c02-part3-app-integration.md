---
title: "DVA-C02 Phần 3: Application Integration"
author: thanhnv1808
date: 2026-04-29 08:20:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, sqs, sns, eventbridge, step-functions, kinesis, api-gateway, certification]
description: "SQS, SNS, EventBridge, Step Functions, Kinesis, API Gateway — tất cả app integration services cho kỳ thi AWS DVA-C02."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> [Phần 1: Compute](/posts/dva-c02-part1-compute) → [Phần 2: Storage & DB](/posts/dva-c02-part2-storage-database) → **Phần 3 (file này)** → [Phần 4: Security](/posts/dva-c02-part4-security) → [Phần 5: Deploy & Monitor](/posts/dva-c02-part5-deploy-monitor) → [Phần 6: Networking + Tips thi](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }

---

## 1. Amazon SQS ⭐ (Cực quan trọng)

### 1.1 Standard vs FIFO — THUỘC LÒNG

| | **Standard** | **FIFO** |
|---|---|---|
| Throughput | **Unlimited** | 300 TPS (3,000 với batching) |
| Ordering | Best-effort | **Strict order trong MessageGroupId** |
| Delivery | **At-least-once** (có thể duplicate) | **Exactly-once** processing |
| Queue name | Bất kỳ | Phải kết thúc **`.fifo`** |
| Deduplication | ❌ | ✅ (5 phút, qua MessageDeduplicationId hoặc content hash) |

---

### 1.2 Thông Số Cần Nhớ

| Thông số | Giá trị |
|---|---|
| **Message size tối đa** | **256 KB** (lớn hơn → dùng **S3 Extended Library**) |
| **Message retention** | 1 phút – **14 ngày** (mặc định 4 ngày) |
| **Visibility Timeout** | 0s – **12 giờ** (mặc định 30s) |
| **Delivery Delay** | 0s – **15 phút** (delay queue) |
| **Long polling (ReceiveMessageWaitTime)** | 0s – **20s** (khuyến nghị dùng) |
| **Batch Receive/Send/Delete** | Tới **10 messages** / request |
| **DLQ maxReceiveCount** | 1 – 1000 |

---

### 1.3 Visibility Timeout — BẪY KINH ĐIỂN

1. Consumer `ReceiveMessage` → message trở thành **invisible** trong Visibility Timeout.
2. Consumer xử lý xong → `DeleteMessage`.
3. **Nếu Visibility Timeout hết mà chưa delete → message quay lại queue → bị xử lý lại → DUPLICATE!**
4. Giải pháp: consumer gọi `ChangeMessageVisibility` để extend nếu xử lý lâu hơn.

> **Câu hỏi thi:** "Consumer xử lý lâu hơn visibility timeout → hậu quả?" → **Message bị xử lý nhiều lần**. Giải pháp: tăng VT hoặc gọi `ChangeMessageVisibility`.
{: .prompt-danger }

---

### 1.4 Dead-Letter Queue (DLQ)

- Queue riêng nhận message không xử lý được sau **maxReceiveCount** lần.
- DLQ của Standard phải là **Standard**, FIFO phải là **FIFO** (matching type).
- **Retention của DLQ nên lớn hơn source** để có thời gian debug.
- **DLQ redrive**: feature redrive message từ DLQ về source queue để retry.

---

### 1.5 Long Polling vs Short Polling

| | Short Polling | Long Polling |
|---|---|---|
| WaitTimeSeconds | 0 (default) | 1-20s (max 20s) |
| Hành vi | Return ngay, có thể empty | Chờ tới khi có message hoặc hết timeout |
| Chi phí | Cao hơn (nhiều API call) | **Thấp hơn** — **khuyến nghị** |

---

### 1.6 FIFO — MessageGroupId & Deduplication

- **MessageGroupId**: messages cùng group → xử lý tuần tự. Group khác → parallel.
- **MessageDeduplicationId**: 2 message cùng DedupId trong 5 phút → AWS chỉ giữ 1.
- Nếu không provide DedupId, SQS dùng hash nội dung (yêu cầu bật **Content-based dedup**).

---

### 1.7 SQS + Lambda

- Lambda **poll** queue (event source mapping).
- **Batch size**: 1–10,000 messages (Standard), 1–10 (FIFO).
- **Batch window**: chờ gom thêm tới 300s.
- **Partial batch response**: Lambda báo 1 số message fail → chỉ message đó quay lại queue.
- **Scaling**: Lambda tự scale theo queue depth.

---

### 1.8 SQS + Auto Scaling (EC2)

- Dùng CloudWatch metric **`ApproximateNumberOfMessagesVisible`** làm trigger cho ASG.
- Hoặc **Backlog per Instance** custom metric để chính xác hơn.

---

## 2. Amazon SNS ⭐

### 2.1 Khái niệm

- **Pub/Sub** service: Topic → nhiều Subscriber nhận bản copy.
- Subscriber types: **SQS, Lambda, HTTP/HTTPS, Email, SMS, Mobile Push, Kinesis Data Firehose, Application endpoint**.
- 1 topic có thể tới **12.5 triệu subscriber**.

---

### 2.2 SNS vs SQS

| | **SNS** | **SQS** |
|---|---|---|
| Pattern | **Push** (pub/sub, fan-out) | **Pull** (queue) |
| 1 message → N consumer | ✅ | ❌ (1 consumer/message) |
| Persistence | Không (phải có subscriber lúc gửi) | Có (tới 14 ngày) |
| Ordering | FIFO topic có | FIFO queue có |

---

### 2.3 Fan-out Pattern — SNS → SQS

```
1 Event
    ↓ SNS Topic
    ├── SQS Queue A → Consumer A (xử lý độc lập)
    ├── SQS Queue B → Consumer B (xử lý độc lập)
    └── SQS Queue C → Consumer C (xử lý độc lập)
```

- Mỗi service có queue riêng → xử lý độc lập, có retention, DLQ riêng.

> **Pattern cực phổ biến trong đề** — khi cần "nhiều service xử lý độc lập cùng 1 event".
{: .prompt-tip }

---

### 2.4 SNS Message Filtering

- Subscription có **FilterPolicy** (JSON) → chỉ subscriber match mới nhận message.
- Filter theo attribute hoặc message body.
- Giảm work cho consumer không cần message đó.

---

### 2.5 SNS FIFO Topic

- Ordering + deduplication như SQS FIFO.
- Chỉ subscribe được từ **SQS FIFO queue**.

---

## 3. Amazon Kinesis

### 3.1 Bốn Dịch Vụ Kinesis — PHÂN BIỆT RÕ

| | **Data Streams** | **Data Firehose** | **Data Analytics** | **Video Streams** |
|---|---|---|---|---|
| Mục đích | Real-time streaming | Load vào destination | Query real-time SQL/Flink | Video streaming |
| Managed | Shard management | Fully managed | Fully managed | Fully managed |
| Retention | **1 – 365 ngày** | Không (chuyển tiếp) | — | Configurable |
| Latency | **~200ms** | **Near real-time** (60s buffer tối thiểu) | Tới giây | — |

---

### 3.2 Kinesis Data Streams (KDS) — Chi Tiết

**Shard throughput:**

| | **Classic** | **Enhanced Fan-Out** |
|---|---|---|
| Write | 1 MB/s hoặc 1,000 records/s per shard | 1 MB/s per shard |
| Read | **2 MB/s shared** giữa consumer | **2 MB/s per consumer** per shard |
| Chi phí | Thấp hơn | Cao hơn |

> **Bẫy:** "Consumer bị throttle khi nhiều app cùng đọc cùng shard?" → dùng **Enhanced Fan-Out** (mỗi consumer có 2 MB/s riêng).
{: .prompt-warning }

**Đặc tính:**
- **Partition Key** quyết định shard → hot partition nếu skew.
- Record size: max **1 MB**.
- **Capacity modes**: Provisioned (tự quản shard) hoặc On-Demand (auto scale).

---

### 3.3 Kinesis Data Firehose

- **Near-real-time**: buffer tối thiểu **60s hoặc 1 MB**, cấu hình đến 900s / 128 MB.
- **Destinations**: S3, Redshift (qua COPY), OpenSearch, Splunk, HTTP endpoints (Datadog, MongoDB...).
- Có thể **transform** data bằng Lambda trước khi ghi.
- Có thể **convert format** (JSON → Parquet/ORC) khi ghi vào S3.
- Compression, encryption built-in.

> **So sánh KDS vs Firehose:**
> - **KDS**: real-time (< 1s), custom consumer logic, retention dài, phải quản shard.
> - **Firehose**: near-real-time (60s+), zero admin, chỉ load vào destination có sẵn.
{: .prompt-tip }

---

### 3.4 Kinesis Client Library (KCL)

- Java library handle shard discovery, load balancing giữa consumer.
- Checkpoint qua **DynamoDB** → KCL yêu cầu DynamoDB table.
- Nếu scale up consumer > shard count → consumer dư sẽ idle.

---

## 4. Amazon EventBridge (CloudWatch Events evolved)

### 4.1 Khái niệm

- **Event bus**: default (AWS services), custom (apps), partner (SaaS như Zendesk).
- Route event tới target qua **Rules** (pattern match hoặc schedule cron/rate).
- Targets: Lambda, SQS, SNS, Step Functions, Kinesis, ECS task, API Gateway, và nhiều hơn nữa.

---

### 4.2 EventBridge vs SNS

| | **EventBridge** | **SNS** |
|---|---|---|
| Filter pattern | **Mạnh, JSON matching** | FilterPolicy đơn giản hơn |
| Schedule | **Cron/rate** | ❌ |
| Target types | Rất nhiều AWS service | Chủ yếu SQS/Lambda/HTTP/Email/SMS |
| Throughput | Thấp hơn SNS | Cao |
| Replay | Có (archive) | ❌ |
| Schema Registry | ✅ | ❌ |

---

### 4.3 EventBridge Pipes (mới)

- Point-to-point: Source → (Filter → Enrich) → Target, không cần Lambda glue.
- Sources: SQS, Kinesis, DynamoDB Streams, Kafka, MSK.

---

## 5. AWS Step Functions

### 5.1 Standard vs Express Workflows — THUỘC LÒNG

| | **Standard** | **Express** |
|---|---|---|
| Duration tối đa | **1 năm** | **5 phút** |
| Execution rate | 2,000 start/s | **100,000 start/s** |
| Execution semantics | **Exactly-once** | At-least-once (Async) hoặc At-most-once (Sync) |
| Pricing | Per transition | Per duration & invocation |
| History | 90 ngày trong console | CloudWatch Logs |
| Use case | Long-running, human approval, ETL | Event processing, IoT, streaming |

---

### 5.2 State Types

| Type | Mô tả |
|---|---|
| **Task** | Thực thi work (Lambda, ECS, SNS...) |
| **Choice** | Branching logic |
| **Parallel** | Chạy nhiều branch đồng thời |
| **Map** | Iterate array, xử lý song song (Distributed Map tới 10k item concurrent) |
| **Wait** | Delay (giây hoặc đến timestamp) |
| **Pass** | Truyền data qua, có thể inject |
| **Succeed / Fail** | Kết thúc |

---

### 5.3 Error Handling

- **Retry** trong state: `ErrorEquals`, `IntervalSeconds`, `MaxAttempts`, `BackoffRate`.
- **Catch**: route tới state khác khi lỗi.

---

### 5.4 Service Integration Patterns

| Pattern | Cơ chế | Use case |
|---|---|---|
| **Request/Response** (default) | Gọi service, lấy ngay response | Lambda sync, SNS publish |
| **Run a Job (.sync)** | Chờ job hoàn tất | ECS task, Batch job |
| **Wait for Callback (.waitForTaskToken)** | Gửi task token, chờ external gọi `SendTaskSuccess/Failure` | Human approval, workflow async |

---

## 6. Amazon API Gateway ⭐

### 6.1 3 Loại API

| | **REST API** | **HTTP API** | **WebSocket API** |
|---|---|---|---|
| Features | Full (API keys, usage plans, request validation, X-Ray...) | Gọn nhẹ, nhanh, **rẻ 71%** | 2-way real-time |
| Use case | Legacy, enterprise | Microservice, serverless mới | Chat, notifications, game |
| Auth | IAM, Lambda Authorizer, Cognito | IAM, JWT (Cognito/OIDC), Lambda Authorizer | IAM, Lambda Authorizer |

> **Default mới:** HTTP API — trừ khi cần feature chỉ có ở REST (usage plans, API keys, request/response transformation mạnh, ...).
{: .prompt-tip }

---

### 6.2 Endpoint Types

| Type | Mô tả |
|---|---|
| **Edge-optimized** | Qua CloudFront (default) — global low latency |
| **Regional** | Trong 1 region, client cùng region |
| **Private** | Chỉ accessible trong VPC qua VPC Endpoint (interface) |

---

### 6.3 Stages & Deployments

- **Stage** (dev, prod): deployment snapshot của API.
- **Stage Variables**: biến gắn stage → trỏ Lambda alias khác nhau.
- **Canary Deployments**: split traffic % giữa canary & stable trong cùng stage.

---

### 6.4 Throttling & Usage Plans

- **Account-level throttling**: mặc định **10,000 req/s**, burst **5,000**.
- **Usage Plans**: gắn **API Keys** → rate limit per key.
- **Throttling error**: `429 Too Many Requests`.

---

### 6.5 Caching

- Cache ở **stage level** (cache size 0.5 GB – 237 GB).
- TTL mặc định **300s** (0-3600s).
- Invalidate với header `Cache-Control: max-age=0` (cần IAM permission `InvalidateCache`).

---

### 6.6 Authentication

| Loại | Cơ chế | Use case |
|---|---|---|
| **IAM** | SigV4 | Service-to-service trong AWS |
| **Lambda Authorizer** | Lambda trả IAM policy. Cache policy (mặc định 300s) | Custom auth logic, token-based |
| **Cognito User Pools** | JWT từ Cognito, API GW verify tự động. **Chỉ authenticate, không authorize** | Web/mobile apps |
| **Resource Policy** | Ai gọi được API (IP, account, VPC endpoint) | IP whitelist, cross-account |

---

### 6.7 Giới Hạn Quan Trọng

| Giới hạn | Giá trị |
|---|---|
| **Integration timeout** | **29 giây** (hard limit) |
| **Max payload** | 10 MB (REST & HTTP API) |
| **WebSocket idle timeout** | 10 phút |

> **Bẫy kinh điển:** "Lambda chạy 5 phút → gọi qua API Gateway" → **LỖI timeout 29s**. Giải pháp: async pattern (API GW → SQS → Lambda hoặc trả 202, poll status).
{: .prompt-danger }

---

### 6.8 Monitoring

- **CloudWatch Metrics**: `4XXError`, `5XXError`, `IntegrationLatency`, `Latency`, `CacheHitCount`, `CacheMissCount`, `Count`.
- **X-Ray**: distributed tracing.

---

## 7. AWS AppSync (GraphQL)

- Managed **GraphQL & Pub/Sub** API.
- Data sources: DynamoDB, RDS (Aurora Serverless), Lambda, HTTP, OpenSearch.
- **Real-time subscriptions** qua WebSocket.
- Offline sync với **Amplify DataStore**.
- Auth: API Key, IAM, Cognito User Pools, OIDC, Lambda.

---

## 8. Amazon MQ

- Managed **Apache ActiveMQ / RabbitMQ**.
- Dùng khi migration từ on-prem cần **protocol chuẩn**: AMQP, MQTT, OpenWire, STOMP, WSS, JMS/NMS.

> **Quyết định:** Hệ thống mới → **SQS + SNS**. Migration cần AMQP/MQTT/JMS → **Amazon MQ**.
{: .prompt-tip }

---

## 9. Amazon MSK (Managed Streaming for Apache Kafka)

- Managed **Kafka** (cluster với brokers).
- **MSK Serverless**: không quản broker.
- So với KDS: Kafka mạnh hơn cho workload cực cao & ecosystem.

---

## 🎯 Tip Ôn Thi Phần 3

1. **SQS Visibility Timeout**: VT quá ngắn + consumer xử lý chậm → **duplicate processing**. Giải pháp: tăng VT hoặc gọi `ChangeMessageVisibility`.
2. **SQS Long Polling luôn khuyên dùng** — giảm empty response, giảm cost.
3. **DLQ retention > source retention** để có thời gian debug.
4. **SQS FIFO: 300 TPS** (3,000 với batching). Cao hơn → bật **High Throughput** mode.
5. **Fan-out pattern**: SNS topic → nhiều SQS queue → nhiều consumer xử lý độc lập.
6. **SNS Filter Policy** — tránh Lambda fan-out rồi filter bằng code.
7. **Kinesis Enhanced Fan-Out** khi nhiều app đọc cùng stream → tránh throttle 2 MB/s shared.
8. **Step Functions Standard vs Express**: Long-running, stateful → **Standard** (1 năm); Short, high-volume → **Express** (5 phút).
9. **Step Functions .waitForTaskToken** — pattern cho human approval hoặc callback từ external.
10. **API Gateway timeout 29s cứng** — long-running phải async.
11. **API Gateway caching**: invalidate với `Cache-Control: max-age=0` header.
12. **Cognito User Pool Authorizer** chỉ authenticate, không authorize.
13. **HTTP API rẻ hơn REST API 71%** — default chọn HTTP API.
14. **EventBridge cho cron/schedule**, không dùng SNS/SQS.
15. **Amazon MQ** chỉ khi migration cần AMQP/MQTT/JMS.

---

> **Tiếp theo:** [Phần 4: Security & Identity (IAM, Cognito, KMS, Secrets Manager, SSM)](/posts/dva-c02-part4-security)
{: .prompt-info }
