# DVA-C02 — Phần 5: Deployment & Monitoring

> **Chuỗi tài liệu:** Phần 1 (Compute) → Phần 2 (Storage & DB) → Phần 3 (App Integration) → Phần 4 (Security) → **Phần 5 (Deploy & Monitor)** → Phần 6 (Networking + Tips).
>
> **Lưu ý:** Đây là phần **đặc trưng nhất của DVA-C02** (khác với Solutions Architect). Chiếm khoảng 30-35% tổng điểm thi. Đọc kỹ CloudFormation, CodeDeploy hooks, CloudWatch.

---

## 1. AWS CloudFormation — ⭐ CỰC QUAN TRỌNG

### 1.1 Khái niệm cốt lõi

- **IaC (Infrastructure as Code)** — khai báo resource bằng YAML/JSON template.
- **Stack**: 1 template được triển khai → thành tập resource.
- **Change Set**: preview thay đổi trước khi apply.
- **Drift Detection**: phát hiện resource bị thay đổi ngoài CloudFormation.

### 1.2 Các section của template

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
| **`!If`, `!And`, `!Or`, `!Not`, `!Equals`** | Conditions |
| **`!ImportValue`** | Import Output từ stack khác | Cross-stack reference |
| **`!GetAZs`** | Danh sách AZ của region |
| **`!Cidr`** | Tạo subnet CIDR từ block lớn |
| **`!Base64`** | Encode (VD: UserData) |
| **`!Transform`** | Gọi macro |

### 1.4 Pseudo Parameters — built-in, không cần khai báo

- `AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::StackId`, `AWS::NotificationARNs`, `AWS::NoValue` (xóa property).

### 1.5 Parameters — validation

```yaml
Parameters:
  DBPassword:
    Type: String
    NoEcho: true        # Không in ra console/CLI
    MinLength: 8
    MaxLength: 41
    AllowedPattern: "[a-zA-Z0-9]*"
    ConstraintDescription: "Alphanumeric, 8-41 chars"
```

- **`NoEcho: true`** — cực quan trọng cho password, API key.
- Types: `String`, `Number`, `CommaDelimitedList`, `List<Number>`, AWS-specific (`AWS::EC2::KeyPair::KeyName`, `AWS::EC2::VPC::Id`...).
- **Dynamic reference to SSM/Secrets Manager**:
  ```yaml
  Password: '{{resolve:ssm-secure:MyParam:1}}'
  DBPassword: '{{resolve:secretsmanager:mySecret:SecretString:password}}'
  ```

### 1.6 Cross-stack References — `Outputs` + `!ImportValue`

- Stack A output với `Export: Name: MyVPCId`.
- Stack B import với `!ImportValue MyVPCId`.
- **KHÔNG xóa stack A nếu output đang được import**.

### 1.7 Nested Stacks

- Stack con được reference trong stack cha qua `AWS::CloudFormation::Stack` resource.
- Template con upload lên S3.
- Use case: tái sử dụng component (VPC template, IAM template).

> 💡 **Cross-stack vs Nested stack:**
> - **Cross-stack**: các stack độc lập, share value qua Export/Import.
> - **Nested**: stack con là 1 phần của stack cha, được quản lý chung.

### 1.8 StackSets

- Deploy cùng template lên **nhiều account + nhiều region** (qua AWS Organizations).
- Admin account tạo → Target accounts nhận.

### 1.9 DeletionPolicy & UpdateReplacePolicy — **BẪY QUAN TRỌNG**

| Policy | Giá trị | Hành vi |
|---|---|---|
| **DeletionPolicy** | `Delete` (default) | Xóa resource khi stack bị xóa |
| | `Retain` | Giữ resource (VD: S3 bucket, RDS) |
| | `Snapshot` | Tạo snapshot trước (chỉ EBS, RDS, ElastiCache...) |
| **UpdateReplacePolicy** | Tương tự | Áp dụng khi resource bị replace (do property change bắt buộc replace) |

> ⚠️ **Bẫy:** "Stack xóa nhưng DB phải giữ" → `DeletionPolicy: Retain` cho RDS resource.

### 1.10 CreationPolicy & UpdatePolicy

- **CreationPolicy** (EC2, ASG, WaitCondition): chờ signal trước khi coi resource là CREATE_COMPLETE.
  - `cfn-signal` script chạy trong UserData → signal lên CloudFormation.
- **UpdatePolicy** (ASG): điều khiển update ASG.
  - `AutoScalingRollingUpdate`, `AutoScalingReplacingUpdate`, `AutoScalingScheduledAction`.

### 1.11 Helper scripts (cfn-helper)

- `cfn-init`: execute metadata `AWS::CloudFormation::Init` (cài package, copy file, services...).
- `cfn-signal`: signal lên stack.
- `cfn-get-metadata`: đọc metadata.
- `cfn-hup`: daemon detect thay đổi metadata, chạy action.

Pattern dùng trong UserData:
```bash
/opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyEC2 --region ${AWS::Region}
/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource MyEC2 --region ${AWS::Region}
```

### 1.12 Stack Policies

- JSON policy **bảo vệ resource** khỏi update nhầm trong stack update.
- Mặc định: mọi resource có thể update.

### 1.13 Change Sets

- Preview thay đổi **trước khi apply**.
- Biết resource nào sẽ bị Replace (dẫn tới downtime).

### 1.14 Rollback & Stack Failure

- Default: **rollback khi lỗi** (`ROLLBACK_IN_PROGRESS` → `ROLLBACK_COMPLETE`).
- Có thể disable rollback để debug (`--on-failure DO_NOTHING` hoặc `DELETE`).
- **Continue Rollback**: retry sau khi fix manual issue.

### 1.15 CloudFormation Registry & Third-party Resource Types

- Resource types ngoài AWS (Datadog, MongoDB Atlas...).
- Custom resource: `AWS::CloudFormation::CustomResource` hoặc `Custom::XYZ` → Lambda/SNS xử lý create/update/delete.

---

## 2. AWS SAM (Serverless Application Model)

### 2.1 Khái niệm

- **Extension của CloudFormation** chuyên cho serverless (Lambda, API Gateway, DynamoDB, Step Functions, ...).
- Template ngắn gọn hơn CloudFormation.
- `Transform: AWS::Serverless-2016-10-31` ở đầu template.

### 2.2 Resource types chính

| SAM | CloudFormation tương đương |
|---|---|
| `AWS::Serverless::Function` | `AWS::Lambda::Function` + Role + Permissions |
| `AWS::Serverless::Api` | `AWS::ApiGateway::RestApi` + các resource liên quan |
| `AWS::Serverless::HttpApi` | API Gateway v2 |
| `AWS::Serverless::SimpleTable` | DynamoDB simple |
| `AWS::Serverless::StateMachine` | Step Functions |
| `AWS::Serverless::LayerVersion` | Lambda Layer |

### 2.3 SAM CLI commands — THUỘC LÒNG

| Command | Mục đích |
|---|---|
| `sam init` | Tạo project mới từ template |
| `sam build` | Build code + dependencies |
| `sam local invoke` | Chạy Lambda local |
| `sam local start-api` | Chạy API Gateway local |
| `sam package` | Upload code lên S3 + tạo packaged template |
| `sam deploy --guided` | Deploy lên AWS |
| `sam sync` | **Sync nhanh** thay đổi code trực tiếp (không qua CloudFormation) — dev mode |
| `sam logs` | Stream CloudWatch logs của Lambda |
| `sam traces` | X-Ray traces |

### 2.4 SAM Policy Templates

- Shortcut thay vì viết IAM policy:
  ```yaml
  Policies:
    - S3ReadPolicy:
        BucketName: my-bucket
    - DynamoDBCrudPolicy:
        TableName: !Ref MyTable
  ```

### 2.5 SAM Deploy Preferences (Canary/Linear)

- Built-in hỗ trợ CodeDeploy:
  ```yaml
  AutoPublishAlias: live
  DeploymentPreference:
    Type: Canary10Percent5Minutes
    Alarms:
      - !Ref MyAlarm
    Hooks:
      PreTraffic: !Ref PreTrafficLambda
      PostTraffic: !Ref PostTrafficLambda
  ```
- Các preset: `Canary10Percent5Minutes`, `Canary10Percent30Minutes`, `Linear10PercentEvery1Minute`, `AllAtOnce`...

---

## 3. AWS CDK (Cloud Development Kit)

### 3.1 Khái niệm

- Code-first IaC — viết bằng **TypeScript, Python, Java, C#, Go**.
- Compile ra CloudFormation template.
- Khái niệm chính:
  - **App** → **Stacks** → **Constructs**.
  - **Construct levels**:
    - **L1 (CFN)**: 1-1 với CloudFormation resource (`CfnBucket`).
    - **L2**: opinionated, sensible defaults (`Bucket`).
    - **L3 (Pattern)**: high-level pattern (VD: `ApplicationLoadBalancedFargateService`).

### 3.2 CDK CLI

| Command | Mục đích |
|---|---|
| `cdk init` | Tạo project |
| `cdk synth` | Compile TypeScript/Python → CloudFormation YAML |
| `cdk bootstrap` | Tạo resource cần thiết để CDK deploy (S3 bucket cho asset) |
| `cdk deploy` | Deploy stack |
| `cdk diff` | Compare local vs deployed |
| `cdk destroy` | Xóa stack |

### 3.3 Khi nào CDK vs SAM vs CloudFormation

- **CloudFormation**: công cụ cơ bản, ai cũng dùng được, full control.
- **SAM**: đặc tả serverless, local testing mạnh, dev Lambda nhanh.
- **CDK**: complex app, dev thích code hơn YAML, dùng loop/logic.

---

## 4. AWS CodeCommit (deprecated nhưng vẫn có trong DVA cũ)

- Git repo managed bởi AWS.
- **Từ tháng 7/2024**: AWS không còn accept customer mới → đề thi mới hạn chế hỏi.
- Biết sơ: IAM để access, HTTPS hoặc SSH, mirror được.

---

## 5. AWS CodeBuild — ⭐ QUAN TRỌNG

### 5.1 Khái niệm

- Managed build service — compile, test, produce artifact.
- Sources: CodeCommit, GitHub, GitHub Enterprise, Bitbucket, S3.
- Environment: Docker image (AWS managed hoặc custom từ ECR).

### 5.2 buildspec.yml — THUỘC LÒNG CẤU TRÚC

```yaml
version: 0.2

env:
  variables:
    KEY: "value"
  parameter-store:
    DB_PASSWORD: "/app/db/password"
  secrets-manager:
    API_KEY: "my-secret:apiKey"

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install -g yarn
  pre_build:
    commands:
      - echo "Before build"
      - aws ecr get-login-password | docker login ...
  build:
    commands:
      - npm run build
      - docker build -t my-app .
  post_build:
    commands:
      - echo "After build"
      - docker push ...

artifacts:
  files:
    - '**/*'
  base-directory: dist
  discard-paths: no

cache:
  paths:
    - 'node_modules/**/*'

reports:
  TestReports:
    files:
      - 'reports/**/*'
    file-format: JUNITXML
```

### 5.3 Môi trường & đặc tính

- **Compute types**: `BUILD_GENERAL1_SMALL/MEDIUM/LARGE/2XLARGE`.
- **Timeout**: 5 phút – 8 giờ (default 60 phút).
- **Artifacts** lưu vào S3, có thể encrypt.
- **VPC config**: chạy build trong VPC để access RDS, ElastiCache, service private.
- **Local build**: `codebuild-agent` trên máy dev để debug buildspec.
- **Validate buildspec trước**: `aws codebuild create-project --generate-cli-skeleton`.

### 5.4 buildspec.yml — các chỗ đặt file

- Mặc định ở **root của source**.
- Đổi vị trí qua **`buildspecOverride`** trong `start-build` API.

### 5.5 Cache

- **S3 cache**: lưu artifact giữa build.
- **Local cache** (faster): `DOCKER_LAYER_CACHE`, `SOURCE_CACHE`, `CUSTOM_CACHE`.

### 5.6 Troubleshooting

- Logs trong **CloudWatch Logs**.
- **Phase errors**: build dừng tại phase đó, chuyển sang phase tiếp theo là `post_build` và `FINALIZING`. **`post_build` vẫn chạy** ngay cả khi `build` fail → chỗ cleanup.

---

## 6. AWS CodeDeploy — ⭐ CỰC KỲ QUAN TRỌNG (NHIỀU CÂU HỎI)

### 6.1 Compute platforms hỗ trợ

| Platform | Deployment types |
|---|---|
| **EC2/On-Premises** | In-place, Blue/Green |
| **Lambda** | Canary, Linear, All-at-once |
| **ECS** | Blue/Green |

### 6.2 appspec.yml — **THUỘC LÒNG CẤU TRÚC THEO PLATFORM**

#### EC2/On-Premises — `appspec.yml`

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html

hooks:
  ApplicationStop:         # Trước tất cả (revision cũ stop)
    - location: scripts/stop.sh
      timeout: 300
      runas: root
  BeforeInstall:           # Trước khi copy file
    - location: scripts/pre-install.sh
  AfterInstall:            # Sau khi copy file
    - location: scripts/after-install.sh
  ApplicationStart:        # Start app
    - location: scripts/start.sh
  ValidateService:         # Health check
    - location: scripts/validate.sh

  # Nếu dùng Load Balancer (Blue/Green hoặc in-place với ELB):
  BeforeBlockTraffic:      # Trước khi rút instance ra khỏi ELB
  AfterBlockTraffic:
  BeforeAllowTraffic:      # Trước khi đưa instance vào ELB
  AfterAllowTraffic:
```

> ⚠️ **Thứ tự hooks EC2 deployment** — đề rất thích hỏi:
> 1. ApplicationStop
> 2. DownloadBundle *(không hook được)*
> 3. BeforeInstall
> 4. Install *(không hook được)*
> 5. AfterInstall
> 6. ApplicationStart
> 7. ValidateService

> Với ELB: thêm `BeforeBlockTraffic` (trước ApplicationStop) và `AfterAllowTraffic` (sau ValidateService).

#### Lambda — `appspec.yml`

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
  - BeforeAllowTraffic: !Ref PreTrafficHook     # Lambda chạy TEST
  - AfterAllowTraffic: !Ref PostTrafficHook     # Lambda chạy TEST
```

- PreTrafficHook & PostTrafficHook là Lambda function gọi `PutLifecycleEventHookExecutionStatus` để báo `Succeeded`/`Failed`.
- **Deployment configs**:
  - `CodeDeployDefault.LambdaCanary10Percent5Minutes`
  - `CodeDeployDefault.LambdaLinear10PercentEvery1Minute`
  - `CodeDeployDefault.LambdaAllAtOnce`

#### ECS — `appspec.yml`

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
  - BeforeInstall:         # Trước khi tạo replacement task set
  - AfterInstall:          # Sau khi replacement task set tạo xong
  - AfterAllowTestTraffic: # Sau khi route test traffic sang replacement
  - BeforeAllowTraffic:    # Trước khi route production traffic
  - AfterAllowTraffic:     # Sau khi replacement ổn định
```

### 6.3 Deployment strategies chi tiết

| Strategy | In-place? | Downtime? | Rollback time |
|---|---|---|---|
| **In-place All-at-once** | ✅ | ✅ | Slow (redeploy cũ) |
| **In-place Half-at-a-time** | ✅ | Partial | Medium |
| **In-place One-at-a-time** | ✅ | ❌ (1 instance 1 lúc) | Medium |
| **Blue/Green** | ❌ | ❌ | **Nhanh** (shift traffic ngược) |

### 6.4 CodeDeploy Agent (EC2/On-prem)

- Phải cài trên EC2/on-prem để CodeDeploy deploy được.
- Cần IAM instance profile với `AWSCodeDeployRole`/CodeDeploy permissions.
- Tag EC2 hoặc dùng ASG để nhắm target.

### 6.5 Rollback

- **Automatic rollback**:
  - Khi deployment fail.
  - Khi CloudWatch alarm trigger trong deployment.
- **Manual rollback**: tạo deployment mới với revision cũ → chạy lại từ đầu.

### 6.6 Traffic Routing (Lambda/ECS)

- **Canary**: 2 bước (VD 10% → wait 5 min → 100%).
- **Linear**: đều đặn (VD 10% mỗi phút).
- **All-at-once**: shift 100% ngay.

> 💡 **Bẫy:** Lambda deployment với CodeDeploy **PHẢI dùng alias** — không thể deploy trực tiếp vào `$LATEST`.

---

## 7. AWS CodePipeline — ⭐ QUAN TRỌNG

### 7.1 Khái niệm

- Orchestrator CI/CD — chain các stage (Source → Build → Test → Deploy).
- **Stages** → **Actions** → **Action Types**.

### 7.2 Action types

| Type | Providers |
|---|---|
| **Source** | CodeCommit, GitHub (via connection), Bitbucket, S3, ECR |
| **Build** | CodeBuild, Jenkins |
| **Test** | CodeBuild, 3rd-party |
| **Deploy** | CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3, OpsWorks, Service Catalog, AppConfig |
| **Approval** | Manual (SNS notification) |
| **Invoke** | Lambda, Step Functions |

### 7.3 Artifacts

- Pipeline lưu **artifact trong S3 bucket** (tự tạo hoặc chỉ định).
- Mỗi action có **input artifacts** và **output artifacts**.
- Encrypt bằng KMS.

### 7.4 CloudWatch Events / EventBridge triggers

- Trigger pipeline: CodeCommit push, ECR image push, S3 object change, schedule (EventBridge rule).

### 7.5 Pipeline failure debugging

- Stage fail → pipeline dừng tại stage đó.
- Logs: CodeBuild logs trong CloudWatch; CodeDeploy events trong console.
- **Retry failed action**: không cần start lại từ đầu.

### 7.6 Cross-region / Cross-account

- **Cross-region**: pipeline trong region A, action deploy ở region B.
- **Cross-account**: IAM role + KMS key cross-account permissions cho artifact S3.

---

## 8. AWS CodeStar (deprecated July 2024)

- Dashboard tích hợp CodeCommit + Build + Deploy + Pipeline.
- **Sẽ không xuất hiện nhiều trong đề thi mới**.

---

## 9. AWS CodeArtifact

- Managed artifact repo (npm, pip, Maven, NuGet, generic).
- Thay Nexus/Artifactory.
- Tích hợp với IAM, KMS.

---

## 10. AWS CodeGuru

- **Reviewer**: AI review code (Java, Python) — bug, security issue.
- **Profiler**: phát hiện hot CPU line trong production.

---

## 11. Amazon CloudWatch — ⭐ CỰC QUAN TRỌNG

### 11.1 CloudWatch Metrics

- **Namespace**: container (VD `AWS/EC2`, `AWS/Lambda`).
- **Metric**: tên + dimensions (VD `CPUUtilization` + `InstanceId=i-xxx`).
- **Resolution**:
  - **Standard**: 1 phút granularity.
  - **High resolution**: 1/5/10/30 giây.
- **Retention**:
  - < 60s data point: **3 giờ**.
  - 1 phút: **15 ngày**.
  - 5 phút: **63 ngày**.
  - 1 giờ: **15 tháng**.

### 11.2 Default metrics vs Custom metrics

- **Default (EC2)**: CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed. **5 phút** miễn phí. Bật **Detailed Monitoring** → **1 phút** (tốn phí).
- **KHÔNG có default** cho:
  - **Memory utilization** — phải cài **CloudWatch Agent** và push custom metric.
  - **Disk space** — tương tự.
- **Custom metric API**: `PutMetricData`. Có thể gửi batch.

### 11.3 CloudWatch Logs

- **Log Group** → **Log Stream** → **Log Events**.
- Sources: EC2 (qua CloudWatch Agent), Lambda, ECS, VPC Flow Logs, API Gateway, Route 53, CloudTrail...
- **Retention**: 1 ngày – 10 năm, **hoặc Never Expire (default — tốn tiền!)**.
- **Subscriptions**: stream real-time sang Kinesis, Firehose, Lambda, OpenSearch.
- **Metric Filters**: extract metric từ log (VD đếm số "ERROR"). KHÔNG retroactive.
- **Logs Insights**: query logs bằng SQL-like syntax.
- **Log Group-level subscription** cho cross-account.

> ⚠️ **Bẫy:** "Metric Filter đếm số ERROR từ log history 1 tháng trước" → **KHÔNG retroactive**, chỉ apply từ lúc tạo filter.

### 11.4 CloudWatch Alarms

- States: `OK`, `ALARM`, `INSUFFICIENT_DATA`.
- Actions: SNS, EC2 (stop/terminate/reboot/recover), ASG (scaling policy), Systems Manager.
- **Composite Alarms**: kết hợp nhiều alarm với AND/OR.

### 11.5 CloudWatch Agent vs Unified Agent

- **CloudWatch Agent** (mới, unified): thu thập logs + metrics + traces. Config JSON.
- **Cấu hình** qua SSM Parameter Store hoặc file local.
- Monitor: memory, disk, swap, custom processes.

### 11.6 CloudWatch Events → EventBridge

- EventBridge là kế thừa của CloudWatch Events — đã đề cập Phần 3.

### 11.7 CloudWatch Synthetics

- Canary — chạy script (Node.js/Python) định kỳ để monitor endpoint/user journey.
- Use case: health check, broken link check, E2E transaction.

### 11.8 CloudWatch RUM & Evidently

- **RUM (Real User Monitoring)**: collect client-side perf data.
- **Evidently**: feature flags + A/B testing.

### 11.9 CloudWatch Contributor Insights

- Top-N analysis từ log/metric (VD: "top IP theo request").

### 11.10 Embedded Metric Format (EMF)

- Log JSON format đặc biệt → CloudWatch tự extract metric từ log.
- Không cần gọi `PutMetricData` — tiết kiệm, latency thấp.

---

## 12. AWS X-Ray — ⭐ QUAN TRỌNG

### 12.1 Khái niệm

- Distributed tracing — theo dõi request qua nhiều service.
- **Segment**: 1 service xử lý request.
- **Subsegment**: tác vụ con (VD: SDK call, DB query, HTTP call).
- **Trace**: full flow của 1 request.
- **Service Graph**: visualization dependency.

### 12.2 Integration

- **Lambda**: bật trong configuration → tự trace.
- **API Gateway**: bật trong stage.
- **EC2/ECS/on-prem**: chạy **X-Ray daemon** (UDP port 2000). App instrument bằng SDK, gửi tới daemon → daemon batch gửi API.
- **Beanstalk**: bật qua option.

### 12.3 IAM permissions

- **Write** (instrumented app): `AWSXRayDaemonWriteAccess` → `xray:PutTraceSegments`, `xray:PutTelemetryRecords`.
- **Read** (dashboard): `AWSXRayReadOnlyAccess`.

### 12.4 Sampling Rules

- Default: **1 request/s + 5% còn lại**.
- Custom rule để giảm cost, focus endpoint quan trọng.

### 12.5 Annotations vs Metadata

- **Annotations**: key-value, **indexed** → filter trace được.
- **Metadata**: key-value phức tạp hơn, **không indexed** → chỉ xem trong trace.

### 12.6 Trace header

- HTTP header `X-Amzn-Trace-Id` → truyền giữa service để chain trace.

### 12.7 ADOT (AWS Distro for OpenTelemetry)

- Distribution của OpenTelemetry, ship metric/trace tới X-Ray, CloudWatch, partner tools.
- Đang dần thay X-Ray SDK cũ.

---

## 13. AWS AppConfig

- Manage & deploy **feature flag, config** runtime (không cần redeploy).
- Validate config (JSON schema, Lambda).
- Deploy strategies: all-at-once, linear, exponential.
- Tích hợp với Lambda Extension — cache config.

---

## 14. AWS Cloud9 (deprecated Jul 2024)

- Web IDE, shared env.
- Không còn trong DVA mới.

---

## 🎯 Tip ôn thi Phần 5

1. **CloudFormation intrinsic functions** — `!Ref`, `!GetAtt`, `!Sub`, `!ImportValue` là phải thuộc.

2. **DeletionPolicy: Retain** — khi stack xóa nhưng giữ RDS/S3.

3. **NoEcho: true** cho password parameter.

4. **Dynamic references** — `{{resolve:ssm:...}}`, `{{resolve:secretsmanager:...}}` để không hardcode secret trong template.

5. **cfn-init + cfn-signal** — bootstrap EC2 từ CloudFormation. Nhớ thứ tự:
   - UserData chạy cfn-init → cài package.
   - cfn-signal → báo hoàn thành → CreationPolicy pass.

6. **CodeDeploy hooks EC2 theo thứ tự**:
   ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService.
   Với ELB: thêm BeforeBlockTraffic đầu, AfterAllowTraffic cuối.

7. **CodeDeploy Lambda** phải dùng **alias** — không deploy vào $LATEST.

8. **CodeDeploy deployment configs**:
   - Lambda: `LambdaCanary10Percent5Minutes`, `LambdaLinear10PercentEvery1Minute`, `LambdaAllAtOnce`.
   - EC2: `OneAtATime`, `HalfAtATime`, `AllAtOnce`, custom.

9. **CodeBuild buildspec phases**: install → pre_build → build → post_build. **post_build vẫn chạy kể cả khi build fail**.

10. **CodeBuild env variables từ Secrets Manager / SSM**:
    ```yaml
    env:
      parameter-store:
        DB_PASS: /app/db
      secrets-manager:
        API_KEY: prod/api:apiKey
    ```

11. **CloudWatch: EC2 không có memory/disk metric mặc định** → cài **CloudWatch Agent**.

12. **CloudWatch Logs Metric Filter KHÔNG retroactive**.

13. **CloudWatch Log retention mặc định: Never Expire** → đốt tiền! Đặt retention khi tạo Log Group.

14. **X-Ray daemon UDP port 2000**. IAM permission: `xray:PutTraceSegments`, `xray:PutTelemetryRecords`.

15. **X-Ray Annotations (indexed, filter được) vs Metadata (không indexed)**.

16. **EMF (Embedded Metric Format)** — tiết kiệm API call, log JSON để CloudWatch auto extract metric.

17. **SAM Deploy Preferences** — canary/linear cho Lambda, không cần viết CodeDeploy config riêng.

18. **CDK Construct levels**: L1 (Cfn*) = raw, L2 = sensible default, L3 = pattern.

19. **CodePipeline artifacts lưu S3** — cross-account cần KMS key share.

20. **AppConfig** — feature flag & config runtime, không cần redeploy app.

---

**→ Sẵn sàng tiếp tục với Phần 6 (Networking + Tổng hợp tips thi)?**
