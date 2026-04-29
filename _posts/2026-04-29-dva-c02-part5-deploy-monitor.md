---
title: "DVA-C02 Phần 5: Deployment & Monitoring"
author: thanhnv1808
date: 2026-04-29 08:40:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, cloudformation, sam, cdk, codebuild, codedeploy, codepipeline, cloudwatch, xray, certification]
description: "CloudFormation, SAM, CDK, CodeBuild, CodeDeploy, CodePipeline, CloudWatch, X-Ray — phần đặc trưng nhất của DVA-C02 (~30-35% tổng điểm)."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> [Phần 1: Compute](/posts/dva-c02-part1-compute) → [Phần 2: Storage & DB](/posts/dva-c02-part2-storage-database) → [Phần 3: App Integration](/posts/dva-c02-part3-app-integration) → [Phần 4: Security](/posts/dva-c02-part4-security) → **Phần 5 (file này)** → [Phần 6: Networking + Tips thi](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }

> **Đây là phần đặc trưng nhất của DVA-C02** — khác với Solutions Architect. Chiếm khoảng **30-35% tổng điểm thi**. Đọc kỹ CloudFormation, CodeDeploy hooks, CloudWatch.
{: .prompt-warning }

---

## 1. AWS CloudFormation ⭐ (Cực quan trọng)

### 1.1 Khái Niệm Cốt Lõi

- **IaC** (Infrastructure as Code) — khai báo resource bằng YAML/JSON template.
- **Stack**: 1 template được triển khai → thành tập resource.
- **Change Set**: preview thay đổi trước khi apply.
- **Drift Detection**: phát hiện resource bị thay đổi ngoài CloudFormation.

---

### 1.2 Các Section của Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: ...
Metadata: ...
Parameters:      # Input từ user khi deploy
Mappings:        # Static lookup table (region → AMI)
Conditions:      # Logic if/else
Transform:       # Macro (VD: AWS::Serverless cho SAM)
Resources:       # BẮT BUỘC — định nghĩa resource
Outputs:         # Export giá trị ra ngoài stack
```

---

### 1.3 Intrinsic Functions — THUỘC LÒNG

| Function | Mục đích | Ví dụ |
|---|---|---|
| **`!Ref`** | Tham chiếu parameter hoặc resource | `!Ref MyVPC` → VPC ID |
| **`!GetAtt`** | Lấy attribute của resource | `!GetAtt MyEC2.PublicIp` |
| **`!Sub`** | String substitution | `!Sub "arn:aws:s3:::${BucketName}"` |
| **`!Join`** | Nối string | `!Join [",", ["a", "b"]]` → `"a,b"` |
| **`!Split`** | Tách string | `!Split [",", "a,b,c"]` |
| **`!Select`** | Chọn phần tử array | `!Select [0, !GetAZs ""]` |
| **`!FindInMap`** | Lookup Mappings | `!FindInMap [RegionMap, !Ref AWS::Region, AMI]` |
| **`!If`, `!And`, `!Or`, `!Not`, `!Equals`** | Conditions | — |
| **`!ImportValue`** | Import Output từ stack khác | Cross-stack reference |
| **`!GetAZs`** | Danh sách AZ của region | — |
| **`!Base64`** | Encode (VD: UserData) | — |

---

### 1.4 Pseudo Parameters (built-in, không cần khai báo)

`AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::StackId`, `AWS::NoValue` (xóa property).

---

### 1.5 Parameters — Validation

```yaml
Parameters:
  DBPassword:
    Type: String
    NoEcho: true        # Không in ra console/CLI — BẮT BUỘC cho password
    MinLength: 8
    MaxLength: 41
    AllowedPattern: "[a-zA-Z0-9]*"
```

**Dynamic reference (không hardcode secret):**
```yaml
Password: '{{resolve:ssm-secure:MyParam:1}}'
DBPassword: '{{resolve:secretsmanager:mySecret:SecretString:password}}'
```

---

### 1.6 Cross-stack References

```yaml
# Stack A — Export
Outputs:
  VPCId:
    Value: !Ref MyVPC
    Export:
      Name: MyVPCId

# Stack B — Import
Resources:
  MyInstance:
    Properties:
      VpcId: !ImportValue MyVPCId
```

**KHÔNG xóa Stack A nếu output đang được import**.

---

### 1.7 Nested Stacks vs Cross-Stack

| | **Nested Stacks** | **Cross-Stack References** |
|---|---|---|
| Quản lý | Stack con là 1 phần của stack cha | Các stack độc lập |
| Share value | Internal | Qua Export/Import |
| Use case | Tái sử dụng component (VPC, IAM template) | Share value giữa team/stack độc lập |

---

### 1.8 StackSets

- Deploy cùng template lên **nhiều account + nhiều region** (qua AWS Organizations).
- Admin account tạo → Target accounts nhận.

---

### 1.9 DeletionPolicy & UpdateReplacePolicy — BẪY QUAN TRỌNG

| Policy | Giá trị | Hành vi |
|---|---|---|
| **DeletionPolicy** | `Delete` (default) | Xóa resource khi stack bị xóa |
| | `Retain` | **Giữ resource** (VD: S3 bucket, RDS) |
| | `Snapshot` | Tạo snapshot trước khi xóa (EBS, RDS, ElastiCache...) |
| **UpdateReplacePolicy** | Tương tự | Áp dụng khi resource bị replace |

> **Bẫy:** "Stack xóa nhưng DB phải giữ" → `DeletionPolicy: Retain` cho RDS resource.
{: .prompt-danger }

---

### 1.10 CreationPolicy & cfn-signal

```yaml
# Trong Resource definition
CreationPolicy:
  ResourceSignal:
    Count: 1
    Timeout: PT15M
```

```bash
# Trong UserData của EC2
/opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyEC2 --region ${AWS::Region}
/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource MyEC2 --region ${AWS::Region}
```

**CloudFormation Helper Scripts:**
- `cfn-init`: execute metadata `AWS::CloudFormation::Init`.
- `cfn-signal`: signal lên stack.
- `cfn-hup`: daemon detect thay đổi metadata, chạy action.

---

### 1.11 Custom Resources

- `AWS::CloudFormation::CustomResource` hoặc `Custom::XYZ` → **Lambda/SNS** xử lý create/update/delete.
- Dùng cho: third-party resource types, logic tùy chỉnh khi deploy.

---

## 2. AWS SAM (Serverless Application Model)

### 2.1 Khái Niệm

- **Extension của CloudFormation** chuyên cho serverless.
- `Transform: AWS::Serverless-2016-10-31` ở đầu template.
- Template ngắn gọn hơn nhiều so với CloudFormation thuần.

---

### 2.2 Resource Types Chính

| SAM | CloudFormation tương đương |
|---|---|
| `AWS::Serverless::Function` | `AWS::Lambda::Function` + Role + Permissions |
| `AWS::Serverless::Api` | `AWS::ApiGateway::RestApi` + liên quan |
| `AWS::Serverless::HttpApi` | API Gateway v2 |
| `AWS::Serverless::SimpleTable` | DynamoDB simple |
| `AWS::Serverless::StateMachine` | Step Functions |
| `AWS::Serverless::LayerVersion` | Lambda Layer |

---

### 2.3 SAM CLI Commands — THUỘC LÒNG

| Command | Mục đích |
|---|---|
| `sam init` | Tạo project mới từ template |
| `sam build` | Build code + dependencies |
| `sam local invoke` | Chạy Lambda **local** |
| `sam local start-api` | Chạy API Gateway local |
| `sam package` | Upload code lên S3 + tạo packaged template |
| `sam deploy --guided` | Deploy lên AWS |
| `sam sync` | **Sync nhanh** thay đổi code (không qua CloudFormation) — dev mode |
| `sam logs` | Stream CloudWatch logs của Lambda |
| `sam traces` | X-Ray traces |

---

### 2.4 SAM Deploy Preferences (Canary/Linear)

```yaml
# Trong SAM template
AutoPublishAlias: live
DeploymentPreference:
  Type: Canary10Percent5Minutes
  Alarms:
    - !Ref MyAlarm
  Hooks:
    PreTraffic: !Ref PreTrafficLambda
    PostTraffic: !Ref PostTrafficLambda
```

**Các preset**: `Canary10Percent5Minutes`, `Canary10Percent30Minutes`, `Linear10PercentEvery1Minute`, `AllAtOnce`...

---

## 3. AWS CDK (Cloud Development Kit)

### 3.1 Khái Niệm

- Code-first IaC — viết bằng **TypeScript, Python, Java, C#, Go**.
- Compile ra CloudFormation template.

| Construct Level | Mô tả | Ví dụ |
|---|---|---|
| **L1 (CFN)** | 1-1 với CloudFormation resource | `CfnBucket` |
| **L2** | Opinionated, sensible defaults | `Bucket` |
| **L3 (Pattern)** | High-level pattern | `ApplicationLoadBalancedFargateService` |

---

### 3.2 CDK CLI

| Command | Mục đích |
|---|---|
| `cdk init` | Tạo project |
| `cdk synth` | Compile → CloudFormation YAML |
| `cdk bootstrap` | Tạo resource cần thiết để CDK deploy |
| `cdk deploy` | Deploy stack |
| `cdk diff` | Compare local vs deployed |
| `cdk destroy` | Xóa stack |

---

### 3.3 CloudFormation vs SAM vs CDK

| | CloudFormation | SAM | CDK |
|---|---|---|---|
| Format | YAML/JSON | YAML (extend CF) | TypeScript/Python/Java/C#/Go |
| Use case | Full control, mọi người dùng được | Serverless, local testing Lambda | Complex app, dev thích code |
| Local testing | ❌ | ✅ `sam local` | Qua SAM |

---

## 4. AWS CodeBuild ⭐

### 4.1 Khái Niệm

- Managed build service — compile, test, produce artifact.
- Sources: CodeCommit, GitHub, GitHub Enterprise, Bitbucket, S3.
- Environment: Docker image (AWS managed hoặc custom từ ECR).

---

### 4.2 buildspec.yml — THUỘC LÒNG CẤU TRÚC

```yaml
version: 0.2

env:
  variables:
    KEY: "value"
  parameter-store:
    DB_PASSWORD: "/app/db/password"       # Từ SSM Parameter Store
  secrets-manager:
    API_KEY: "my-secret:apiKey"           # Từ Secrets Manager

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install -g yarn
  pre_build:
    commands:
      - echo "Before build"
  build:
    commands:
      - npm run build
      - docker build -t my-app .
  post_build:                             # Vẫn chạy kể cả khi build fail
    commands:
      - echo "After build"
      - docker push ...

artifacts:
  files:
    - '**/*'
  base-directory: dist

cache:
  paths:
    - 'node_modules/**/*'                 # Cache giữa các build

reports:
  TestReports:
    files:
      - 'reports/**/*'
    file-format: JUNITXML
```

> **buildspec phases: install → pre_build → build → post_build**. **`post_build` vẫn chạy kể cả khi `build` fail** → dùng để cleanup.
{: .prompt-warning }

---

### 4.3 CodeBuild đặc tính

- **Timeout**: 5 phút – 8 giờ (default 60 phút).
- **VPC config**: chạy build trong VPC để access RDS, ElastiCache, service private.
- **Local build**: `codebuild-agent` trên máy dev để debug buildspec.
- Logs trong **CloudWatch Logs**.

---

## 5. AWS CodeDeploy ⭐⭐ (Nhiều câu hỏi nhất phần CI/CD)

### 5.1 Compute Platforms

| Platform | Deployment types |
|---|---|
| **EC2/On-Premises** | In-place, Blue/Green |
| **Lambda** | Canary, Linear, All-at-once |
| **ECS** | Blue/Green |

---

### 5.2 appspec.yml — EC2/On-Premises

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html

hooks:
  ApplicationStop:         # Revision cũ stop
    - location: scripts/stop.sh
      timeout: 300
  BeforeInstall:           # Trước khi copy file
    - location: scripts/pre-install.sh
  AfterInstall:            # Sau khi copy file
    - location: scripts/after-install.sh
  ApplicationStart:        # Start app
    - location: scripts/start.sh
  ValidateService:         # Health check
    - location: scripts/validate.sh
```

### Thứ Tự Hooks EC2 — ĐỀ RẤT THÍCH HỎI

```
1. ApplicationStop          (script có thể run)
2. DownloadBundle           (không hook được)
3. BeforeInstall            (script có thể run)
4. Install                  (không hook được)
5. AfterInstall             (script có thể run)
6. ApplicationStart         (script có thể run)
7. ValidateService          (script có thể run)
```

Với ELB: thêm `BeforeBlockTraffic` (trước ApplicationStop) và `AfterAllowTraffic` (sau ValidateService).

---

### 5.3 appspec.yml — Lambda

```yaml
version: 0.0
Resources:
  - myFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: my-function
        Alias: live
        CurrentVersion: "1"
        TargetVersion: "2"

Hooks:
  - BeforeAllowTraffic: !Ref PreTrafficHook
  - AfterAllowTraffic: !Ref PostTrafficHook
```

- **Deployment configs Lambda**: `LambdaCanary10Percent5Minutes`, `LambdaLinear10PercentEvery1Minute`, `LambdaAllAtOnce`.

> **Bẫy:** Lambda deployment với CodeDeploy **PHẢI dùng alias** — không thể deploy trực tiếp vào `$LATEST`.
{: .prompt-danger }

---

### 5.4 appspec.yml — ECS

```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:...task-definition/..."
        LoadBalancerInfo:
          ContainerName: "web"
          ContainerPort: 80

Hooks:
  - BeforeInstall:          # Trước khi tạo replacement task set
  - AfterInstall:           # Sau khi replacement task set tạo xong
  - AfterAllowTestTraffic:  # Sau khi route test traffic
  - BeforeAllowTraffic:     # Trước khi route production traffic
  - AfterAllowTraffic:      # Sau khi replacement ổn định
```

---

### 5.5 Deployment Strategies

| Strategy | In-place? | Downtime? | Rollback time |
|---|---|---|---|
| **In-place All-at-once** | ✅ | ✅ | Slow (redeploy cũ) |
| **In-place Half-at-a-time** | ✅ | Partial | Medium |
| **In-place One-at-a-time** | ✅ | ❌ (1 instance 1 lúc) | Medium |
| **Blue/Green** | ❌ | ❌ | **Nhanh** (shift traffic ngược) |

---

### 5.6 Traffic Routing (Lambda/ECS)

| Type | Mô tả |
|---|---|
| **Canary** | 2 bước (VD 10% → wait 5 min → 100%) |
| **Linear** | Đều đặn (VD 10% mỗi phút) |
| **All-at-once** | Shift 100% ngay |

---

### 5.7 Rollback

- **Automatic rollback**: khi deployment fail hoặc khi CloudWatch alarm trigger.
- **Manual rollback**: tạo deployment mới với revision cũ.

---

## 6. AWS CodePipeline ⭐

### 6.1 Action Types

| Type | Providers |
|---|---|
| **Source** | CodeCommit, GitHub (via connection), Bitbucket, S3, ECR |
| **Build** | CodeBuild, Jenkins |
| **Test** | CodeBuild, 3rd-party |
| **Deploy** | CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3 |
| **Approval** | Manual (SNS notification) |
| **Invoke** | Lambda, Step Functions |

---

### 6.2 Artifacts

- Pipeline lưu **artifact trong S3 bucket** (tự tạo hoặc chỉ định).
- Mỗi action có input artifacts và output artifacts. Encrypt bằng KMS.

---

### 6.3 Pipeline Failure Debugging

- Stage fail → pipeline dừng tại stage đó.
- **Retry failed action**: không cần start lại từ đầu.
- **Cross-account**: artifact S3 cần KMS key share.

---

## 7. AWS CodeArtifact

- Managed artifact repo (npm, pip, Maven, NuGet, generic).
- Thay Nexus/Artifactory. Tích hợp với IAM, KMS.

---

## 8. AWS CodeGuru

| | Mô tả |
|---|---|
| **Reviewer** | AI review code (Java, Python) — bug, security issue |
| **Profiler** | Phát hiện hot CPU line trong production |

---

## 9. Amazon CloudWatch ⭐ (Cực quan trọng)

### 9.1 Metrics

- **Namespace**: container (VD `AWS/EC2`, `AWS/Lambda`).
- **Metric**: tên + dimensions.
- **Resolution**: Standard (1 phút) hoặc High resolution (1/5/10/30 giây).

**Retention:**

| Granularity | Retention |
|---|---|
| < 60s data point | 3 giờ |
| 1 phút | 15 ngày |
| 5 phút | 63 ngày |
| 1 giờ | 15 tháng |

---

### 9.2 Default vs Custom Metrics

**EC2 Default** (5 phút miễn phí, bật Detailed Monitoring → 1 phút):
- CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed.

> **KHÔNG có default metric cho:**
> - **Memory utilization** → phải cài **CloudWatch Agent** và push custom metric.
> - **Disk space** → tương tự.
{: .prompt-danger }

**Custom metric API**: `PutMetricData`.

---

### 9.3 CloudWatch Logs

- **Log Group** → **Log Stream** → **Log Events**.
- **Retention**: 1 ngày – 10 năm, **hoặc Never Expire (default — TỐN TIỀN!)**.
- **Subscriptions**: stream real-time sang Kinesis, Firehose, Lambda, OpenSearch.
- **Metric Filters**: extract metric từ log. **KHÔNG retroactive**.
- **Logs Insights**: query logs bằng SQL-like syntax.

> **Bẫy:** "Metric Filter đếm số ERROR từ log history 1 tháng trước" → **KHÔNG retroactive**, chỉ apply từ lúc tạo filter.
{: .prompt-warning }

---

### 9.4 CloudWatch Alarms

- States: `OK`, `ALARM`, `INSUFFICIENT_DATA`.
- Actions: SNS, EC2 (stop/terminate/reboot/recover), ASG (scaling policy).
- **Composite Alarms**: kết hợp nhiều alarm với AND/OR.

---

### 9.5 CloudWatch Agent

- Thu thập logs + metrics + traces.
- Config qua **SSM Parameter Store** hoặc file local.
- Monitor: **memory, disk, swap**, custom processes (những gì EC2 không có default).

---

### 9.6 CloudWatch Synthetics

- Canary — chạy script (Node.js/Python) định kỳ để monitor endpoint/user journey.
- Use case: health check, broken link check, E2E transaction.

---

### 9.7 Embedded Metric Format (EMF)

- Log JSON format đặc biệt → CloudWatch tự extract metric từ log.
- Không cần gọi `PutMetricData` — tiết kiệm, latency thấp.

---

## 10. AWS X-Ray ⭐

### 10.1 Khái Niệm

| Khái niệm | Mô tả |
|---|---|
| **Segment** | 1 service xử lý request |
| **Subsegment** | Tác vụ con (SDK call, DB query, HTTP call) |
| **Trace** | Full flow của 1 request |
| **Service Graph** | Visualization dependency |

---

### 10.2 Integration

| Service | Cách bật |
|---|---|
| **Lambda** | Bật trong configuration |
| **API Gateway** | Bật trong stage |
| **EC2/ECS/on-prem** | Chạy **X-Ray daemon** (UDP port 2000), instrument bằng SDK |
| **Elastic Beanstalk** | Bật qua option |

---

### 10.3 IAM Permissions

- **Write** (instrumented app): `AWSXRayDaemonWriteAccess` → `xray:PutTraceSegments`, `xray:PutTelemetryRecords`.
- **Read** (dashboard): `AWSXRayReadOnlyAccess`.

---

### 10.4 Sampling Rules

- Default: **1 request/s + 5% còn lại**.
- Custom rule để giảm cost, focus endpoint quan trọng.

---

### 10.5 Annotations vs Metadata — QUAN TRỌNG

| | **Annotations** | **Metadata** |
|---|---|---|
| Indexed | ✅ | ❌ |
| Filter trace được | ✅ | ❌ |
| Data type | Key-value đơn giản | Key-value phức tạp hơn |
| Use case | Search/filter traces | Debug info chi tiết |

---

### 10.6 Trace Header

HTTP header `X-Amzn-Trace-Id` → truyền giữa service để chain trace.

---

## 11. AWS AppConfig

- Manage & deploy **feature flag, config** runtime (không cần redeploy).
- Validate config (JSON schema, Lambda).
- Deploy strategies: all-at-once, linear, exponential.
- Tích hợp với Lambda Extension — cache config.

---

## 🎯 Tip Ôn Thi Phần 5

1. **CloudFormation intrinsic functions**: `!Ref`, `!GetAtt`, `!Sub`, `!ImportValue` phải thuộc.
2. **DeletionPolicy: Retain** — khi stack xóa nhưng giữ RDS/S3.
3. **NoEcho: true** cho password parameter.
4. **Dynamic references** — `{{resolve:ssm:...}}`, `{{resolve:secretsmanager:...}}` để không hardcode secret.
5. **cfn-init + cfn-signal**: UserData chạy cfn-init → cài package → cfn-signal → báo hoàn thành.
6. **CodeDeploy hooks EC2 theo thứ tự**: ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService. Với ELB: thêm BeforeBlockTraffic đầu, AfterAllowTraffic cuối.
7. **CodeDeploy Lambda phải dùng alias** — không deploy vào $LATEST.
8. **CodeBuild buildspec phases**: install → pre_build → build → **post_build** (vẫn chạy kể cả khi build fail).
9. **CodeBuild env variables từ Secrets Manager / SSM** — không hardcode trong buildspec.
10. **CloudWatch: EC2 không có memory/disk metric mặc định** → cài **CloudWatch Agent**.
11. **CloudWatch Logs Metric Filter KHÔNG retroactive**.
12. **CloudWatch Log retention mặc định: Never Expire** → Đặt retention khi tạo Log Group.
13. **X-Ray daemon UDP port 2000**. IAM: `xray:PutTraceSegments`.
14. **X-Ray Annotations (indexed, filter được) vs Metadata (không indexed)**.
15. **SAM Deploy Preferences**: canary/linear cho Lambda, tích hợp CodeDeploy.
16. **CDK Construct levels**: L1 (Cfn*) = raw, L2 = sensible default, L3 = pattern.
17. **CodePipeline artifacts lưu S3** — cross-account cần KMS key share.
18. **AppConfig** — feature flag & config runtime, không cần redeploy app.

---

> **Tiếp theo:** [Phần 6: Networking, Edge & Tổng hợp Tips Thi (VPC, Route 53, CloudFront, Decision Trees, TOP 30 bẫy)](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }
