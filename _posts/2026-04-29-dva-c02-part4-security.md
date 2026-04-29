---
title: "DVA-C02 Phần 4: Security & Identity"
author: thanhnv1808
date: 2026-04-29 08:30:00 +0700
categories: [AWS, DVA-C02]
tags: [aws, dva-c02, iam, cognito, kms, secrets-manager, ssm, security, certification]
description: "IAM, Cognito, KMS, Secrets Manager, SSM Parameter Store, STS — tất cả security services cần thuộc lòng cho kỳ thi AWS DVA-C02."
pin: false
comments: true
---

> **Chuỗi DVA-C02:**
> [Phần 1: Compute](/posts/dva-c02-part1-compute) → [Phần 2: Storage & DB](/posts/dva-c02-part2-storage-database) → [Phần 3: App Integration](/posts/dva-c02-part3-app-integration) → **Phần 4 (file này)** → [Phần 5: Deploy & Monitor](/posts/dva-c02-part5-deploy-monitor) → [Phần 6: Networking + Tips thi](/posts/dva-c02-part6-networking-tips)
{: .prompt-info }

---

## 1. IAM (Identity & Access Management) ⭐⭐ (Cực kỳ quan trọng)

### 1.1 Các Khái Niệm Cốt Lõi

| Khái niệm | Mô tả |
|---|---|
| **User** | Đại diện người/ứng dụng, có credentials (password, access keys) |
| **Group** | Tập hợp user, áp dụng policy chung. **KHÔNG phải identity**, không chứa user khác được |
| **Role** | Identity **tạm thời**, được **assume** bởi trusted entity. **Không có credentials dài hạn** — dùng STS |
| **Policy** | JSON định nghĩa permissions |

---

### 1.2 Các Loại Policy

| Loại | Mô tả | Use case |
|---|---|---|
| **AWS Managed** | AWS tạo, quản lý (VD: `AmazonS3ReadOnlyAccess`) | Tiện, phổ biến |
| **Customer Managed** | Bạn tạo, quản lý | Custom, reusable |
| **Inline** | Gắn trực tiếp vào user/group/role | One-off, tight coupling |
| **Resource-based** | Gắn vào resource (S3 bucket policy, SQS policy, KMS key policy...) | Cross-account, external access |
| **Permission Boundary** | Giới hạn MAX permission của user/role | Delegation — admin giới hạn quyền cho dev tạo role |
| **SCP** (AWS Organizations) | Giới hạn permission account con | Governance |
| **Session Policy** | Truyền khi AssumeRole → giới hạn session | Fine-grained temp |
| **ABAC** | Policy dùng tag để control | Scale với team lớn |

---

### 1.3 Cấu Trúc IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "OptionalID",
    "Effect": "Allow",
    "Principal": "...",
    "Action": ["s3:GetObject"],
    "Resource": ["arn:aws:s3:::bucket/*"],
    "Condition": {
      "StringEquals": { "aws:username": "John" }
    }
  }]
}
```

`Principal` chỉ dùng trong **resource-based policy**.

---

### 1.4 Evaluation Logic — CỰC KỲ QUAN TRỌNG

```
Request tới IAM →
1. Explicit DENY ở bất kỳ policy nào? → DENY (dừng ngay)
2. Explicit ALLOW ở ít nhất 1 policy? → ALLOW
3. Không có match → Implicit DENY
```

**Cross-account access:**
- Cả bên **source** (IAM policy của principal) VÀ bên **destination** (resource-based policy) đều phải allow.

> **Bẫy:** Explicit Deny **luôn thắng** explicit Allow. **SCP deny → ngay cả root user account con cũng không qua được**.
{: .prompt-danger }

---

### 1.5 IAM Policy Conditions — Hay Xuất Hiện

| Condition Key | Dùng cho |
|---|---|
| `aws:SourceIp` | Giới hạn IP |
| `aws:SourceVpc`, `aws:SourceVpce` | Giới hạn VPC / VPC Endpoint |
| `aws:MultiFactorAuthPresent` | Bắt buộc MFA |
| `aws:RequestedRegion` | Giới hạn region |
| `aws:PrincipalTag/<key>` | Tag của principal (ABAC) |
| `aws:ResourceTag/<key>` | Tag của resource (ABAC) |
| `aws:SecureTransport` | Bắt HTTPS |
| `s3:prefix`, `s3:x-amz-server-side-encryption` | Service-specific |

---

### 1.6 Best Practices

1. **Không dùng root user** cho daily task. Bật MFA cho root.
2. **Principle of least privilege**.
3. **Dùng Role thay vì Access Key** (EC2 Instance Profile, ECS Task Role, Lambda execution role).
4. **Không hardcode credentials** trong code — dùng role / Secrets Manager / SSM.
5. **Rotate access keys** định kỳ.
6. **IAM Access Analyzer**: phát hiện resource chia sẻ với external entity.

---

### 1.7 IAM Roles — Scenarios Hay Hỏi

| Scenario | Role |
|---|---|
| **EC2 Instance Profile** | Gắn role vào EC2 → app chạy trong EC2 dùng role thay access key |
| **ECS Task Role** | Container gọi AWS API |
| **Lambda Execution Role** | Lambda assume để gọi AWS service |
| **Cross-account Role** | Account A tạo role → Account B call `sts:AssumeRole` → nhận temp creds của A |
| **Service-Linked Role** | AWS service tự tạo (VD: ASG, ELB) — không sửa được |

---

### 1.8 EC2 Instance Metadata & Role Credentials

```bash
# Lấy temp credentials qua IMDS
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
```

- Credentials auto-rotate (thường 6 giờ). SDK tự retrieve — developer không cần hardcode.
- Luôn dùng **IMDSv2** (token-based) — v1 có nguy cơ SSRF.

---

## 2. AWS STS (Security Token Service)

### 2.1 Các API Chính

| API | Use case |
|---|---|
| **AssumeRole** | Same/cross-account role assume |
| **AssumeRoleWithSAML** | SAML 2.0 federation (enterprise SSO) |
| **AssumeRoleWithWebIdentity** | Web identity (Google, Facebook) — **thường khuyên dùng Cognito** |
| **GetSessionToken** | MFA session với long-term credentials |
| **DecodeAuthorizationMessage** | Decode encoded error từ IAM |

---

### 2.2 Temporary Credentials

- Trả về `AccessKeyId`, `SecretAccessKey`, `SessionToken`, `Expiration`.
- Duration: **15 phút – 12 giờ** (default 1 giờ; với chain role, max **1 giờ**).
- **Không thể revoke** (đợi expire) — trừ khi edit role policy với condition `aws:TokenIssueTime`.

---

### 2.3 Cross-Account Pattern

```
Account B (source)          Account A (destination)
User/service →              Role (Trust Policy cho Account B)
  sts:AssumeRole →         ← Temporary credentials
  Dùng creds của A để gọi API trong A
```

---

## 3. Amazon Cognito ⭐

### 3.1 User Pools vs Identity Pools — THUỘC LÒNG

| | **User Pool** | **Identity Pool** (Federated Identities) |
|---|---|---|
| Mục đích | **Authentication (Who are you?)** | **Authorization (What can you do?)** |
| Output | **JWT token** (ID, Access, Refresh) | **Temporary AWS credentials** từ IAM Role |
| User directory | Quản lý user (sign-up/sign-in) | Không — nhận identity từ provider khác |
| Federation | Facebook, Google, Apple, SAML, OIDC | User Pool, Cognito, Facebook, Google, SAML, OIDC |
| Integration | API Gateway authorizer, ALB, AppSync | Truy cập S3, DynamoDB... trực tiếp từ mobile/web |

> **Pattern phổ biến:**
> - **User Pool only**: API Gateway authorizer → backend Lambda/ECS.
> - **User Pool + Identity Pool**: user login → token → Identity Pool trả AWS creds → client gọi S3/DynamoDB trực tiếp.
{: .prompt-tip }

---

### 3.2 Cognito User Pool — Tính Năng

- **Sign-up, sign-in**, forgot password, MFA (SMS, TOTP).
- **Hosted UI** (có thể customize).
- **Lambda Triggers**: Pre/Post Sign-up, Pre/Post Authentication, Pre Token Generation, Migrate User, Custom Message...
- **User groups**: gán role ARN → Identity Pool dùng role này khi user trong group.
- **OAuth 2.0 flows**: Authorization Code (khuyến nghị), Implicit (deprecated), Client Credentials.

---

### 3.3 Cognito Identity Pool — Tính Năng

- **Authenticated users**: role "Authenticated".
- **Unauthenticated (guest) users**: role "Unauthenticated" → truy cập hạn chế.
- **Fine-grained role selection**: chọn role dựa trên Cognito group hoặc rule.

---

### 3.4 JWT Tokens

| Token | Mô tả | Default TTL |
|---|---|---|
| **ID Token** | Identity claims (email, name...) — dùng cho authentication | 1 giờ |
| **Access Token** | Scope để gọi API | 1 giờ |
| **Refresh Token** | Đổi lấy ID/Access mới khi hết hạn | **30 ngày** |

---

## 4. AWS KMS (Key Management Service) ⭐

### 4.1 Các Loại Key

| Loại | Mô tả | Chi phí |
|---|---|---|
| **AWS Owned Keys** | AWS quản lý hoàn toàn, không nhìn thấy | **Free** |
| **AWS Managed Keys** | `aws/service-name` — AWS tạo cho service, rotate tự động 1 năm | Không phí key |
| **Customer Managed Keys (CMK)** | Bạn tạo, quản lý | **$1/month/key** |
| **Imported Keys** | BYOK (Bring Your Own Key) | $1/month/key |
| **Custom Key Store** | Dùng CloudHSM | — |

---

### 4.2 Symmetric vs Asymmetric

| | **Symmetric** (AES-256) | **Asymmetric** (RSA, ECC) |
|---|---|---|
| Use case | Encrypt/Decrypt AWS service data | Sign/Verify, Encrypt/Decrypt ở client |
| Key material ra ngoài KMS? | **KHÔNG bao giờ** | Public key ra được, private thì không |
| AWS service integrate | Phần lớn | Ít |
| Auto rotation | ✅ | ❌ |

---

### 4.3 Giới Hạn Quan Trọng

- **Encrypt/Decrypt API: data tối đa 4 KB** → data lớn hơn phải dùng **Envelope Encryption**.
- **Regional**: KMS key không ra khỏi region (trừ Multi-Region Keys).
- **Request quotas**: thường **10,000 req/s** → Throttle → `ThrottlingException`.

---

### 4.4 Envelope Encryption — Pattern Quan Trọng

```
1. GenerateDataKey → plaintext data key + encrypted data key
2. Client dùng plaintext data key encrypt data (AES-256 local)
3. Lưu encrypted data key CÙNG VỚI encrypted data
4. Xóa plaintext data key khỏi memory ngay
5. Decrypt: call Decrypt với encrypted data key → plaintext key → decrypt data
```

**AWS Encryption SDK**, **DynamoDB Encryption Client**, **S3 Encryption Client** đều dùng pattern này.

---

### 4.5 Key Policies & Grants

- **Key Policy**: resource-based policy gắn vào KMS key.
  - **KMS KHÔNG có implicit allow từ IAM admin** nếu key policy không cho phép.
  - **PHẢI edit key policy** nếu cần ai đó access key.
- **Grants**: delegate temporary permission.
- **ViaService condition**: giới hạn key chỉ dùng được qua service cụ thể.

> **Bẫy kinh điển:** "User có AdministratorAccess IAM policy nhưng không dùng được KMS key?" → **Key policy chưa grant** → sửa key policy (IAM admin không bypass được).
{: .prompt-danger }

---

### 4.6 Key Rotation

| Loại Key | Auto Rotation | Interval |
|---|---|---|
| **AWS Managed** | ✅ Auto | **1 năm** (không config được) |
| **Customer Managed Symmetric** | ✅ Bật được | **1 năm** mặc định (hoặc custom 90-2560 ngày) |
| **Imported keys** | ❌ | Manual (tạo key mới) |
| **Asymmetric keys** | ❌ | Manual |

- Rotation giữ nguyên **key ID**, chỉ thay backing key material. Data encrypted bằng old material vẫn decrypt được.

---

### 4.7 Multi-Region Keys

- Replicate key sang region khác với **cùng key ID**.
- Encrypted data ở region A decrypt được ở region B **mà không cần re-encrypt**.

---

## 5. AWS Secrets Manager

### 5.1 Đặc Điểm

- Lưu secret (password, API key, OAuth token...).
- **Auto-rotation** bằng Lambda (built-in cho RDS, DocumentDB, Redshift; custom cho khác).
- Encrypt bằng KMS. Gắn resource-based policy để cross-account.
- **Giá**: **$0.40/secret/tháng + $0.05/10,000 API calls**.

---

### 5.2 Rotation Pattern — 4 Steps

```
1. createSecret    → tạo secret mới
2. setSecret       → apply new secret vào service
3. testSecret      → verify new secret works
4. finishSecret    → mark AWSCURRENT, old thành AWSPREVIOUS
```

---

## 6. AWS Systems Manager Parameter Store vs Secrets Manager — PHẢI PHÂN BIỆT

| | **Secrets Manager** | **SSM Parameter Store** |
|---|---|---|
| Giá | $0.40/secret/tháng | **Miễn phí** (Standard), $0.05/param/tháng (Advanced) |
| Auto rotation | ✅ Built-in | ❌ (tự làm qua Lambda + EventBridge) |
| Size | 64 KB | Standard: 4 KB, Advanced: 8 KB |
| Cross-account | ✅ | ❌ (chỉ IAM) |
| Built-in RDS integration | ✅ | ❌ |
| Hierarchy (`/app/prod/db-url`) | ❌ | ✅ |
| CloudFormation reference | `{{resolve:secretsmanager:...}}` | `{{resolve:ssm:...}}` / `{{resolve:ssm-secure:...}}` |

> **Quyết định nhanh:**
> - Cần **auto-rotation** (VD: DB password) → **Secrets Manager**
> - Config thông thường, không nhạy cảm lắm, tiết kiệm chi phí → **SSM Parameter Store**
> - Secret < 4 KB không rotate → **Parameter Store SecureString** vẫn được
{: .prompt-tip }

---

### 6.1 SSM Parameter Types

| Type | Mô tả |
|---|---|
| **String** | Plain text |
| **StringList** | Comma-separated |
| **SecureString** | Encrypt bằng KMS |

### 6.2 SSM Tiers

| Tier | Params | Size | Phí |
|---|---|---|---|
| **Standard** | 10,000 | 4 KB | Miễn phí |
| **Advanced** | 100,000 | 8 KB | $0.05/param/tháng |

### 6.3 Hierarchical Naming

```
/my-app/dev/db-url
/my-app/dev/db-password
/my-app/prod/db-url
```

`GetParametersByPath` với path `/my-app/dev/` lấy hết param của env dev.

---

## 7. AWS Certificate Manager (ACM)

- **Miễn phí** SSL/TLS certificate cho AWS services (ALB, NLB listener TLS, CloudFront, API Gateway).
- **Public certs**: auto-renew.
- **KHÔNG export** được private key của public cert.

> **Bẫy:** "Cert cho ALB" → **ACM**. "Cert cho EC2 web server tự host Nginx" → **KHÔNG** dùng ACM được (phải đặt sau ALB với ACM).
{: .prompt-warning }

---

## 8. AWS WAF (Web Application Firewall)

- Layer 7 firewall bảo vệ web app khỏi SQLi, XSS, bad bots, DDoS L7.
- Deploy tại: **CloudFront, ALB, API Gateway, AppSync, Cognito User Pool**.
- **KHÔNG** deploy trực tiếp trên EC2/NLB.

**Loại Rules:**
- **Managed Rules**: AWS & Marketplace (Common, Known Bad Inputs, SQLi...).
- **Custom Rules**: match theo IP, country, rate, string, regex, size.
- **Rate-based rule**: giới hạn request rate từ 1 IP.
- **Actions**: Allow, Block, Count, CAPTCHA, Challenge.

---

## 9. AWS Shield

| | **Shield Standard** | **Shield Advanced** |
|---|---|---|
| Giá | **Miễn phí** | **$3,000/tháng** |
| Protection | L3/L4 DDoS (auto-applied) | CloudFront/ALB/NLB/R53/Global Accelerator |
| Support | — | 24/7 DRT support, cost protection |

---

## 10. Các Dịch Vụ Security Khác — Biết Sơ

| Service | Mục đích |
|---|---|
| **Amazon GuardDuty** | Threat detection ML-based từ VPC Flow Logs, CloudTrail, DNS logs |
| **Amazon Inspector** | Vulnerability scan EC2, ECR image, Lambda |
| **Amazon Macie** | Discover PII trong S3 (ML-based) |
| **AWS CloudTrail** | Log mọi API call, audit trail. Management events (default), Data events (opt-in) |
| **AWS Config** | Track config change, compliance rule |
| **AWS Artifact** | Portal download compliance report (SOC, PCI, ISO) |
| **AWS Detective** | Investigate security finding |
| **AWS Security Hub** | Aggregate finding từ GuardDuty, Inspector, Macie, Config |

> **Decision tree "phát hiện X":**
> - Threat real-time (bitcoin mining, compromised instance) → **GuardDuty**
> - Vulnerability trong EC2/ECR/Lambda → **Inspector**
> - PII trong S3 → **Macie**
> - API call audit → **CloudTrail**
> - Config compliance (VD: S3 phải encrypted) → **AWS Config**
{: .prompt-tip }

---

## 11. Signed URLs — S3 vs CloudFront

| | **S3 Pre-signed URL** | **CloudFront Signed URL / Cookie** |
|---|---|---|
| Scope | 1 object cụ thể | 1 file (URL) hoặc nhiều file (cookie) |
| Auth backing | IAM creds của người tạo URL | CloudFront Key Pair |
| Expiration | Tới 7 ngày (SigV4) | Custom, dài hơn |
| Use case | Upload/download tạm thời | Content distribution với expiration, IP restriction |

---

## 🎯 Tip Ôn Thi Phần 4

1. **IAM evaluation logic**: Explicit Deny → Explicit Allow → Implicit Deny. **Deny luôn thắng**.
2. **Cross-account cần 2 bên đều cho phép**: source (IAM policy) + destination (resource-based policy hoặc trust).
3. **EC2 → luôn dùng Instance Profile (role)**, không hardcode access key.
4. **Cognito**: User Pool = authentication (JWT); Identity Pool = authorization (AWS temp creds).
5. **KMS bẫy lớn nhất**: admin IAM không tự động access được CMK. **Key Policy phải cho phép**.
6. **KMS Encrypt API giới hạn 4 KB** → data lớn hơn → **Envelope Encryption** với `GenerateDataKey`.
7. **Secrets Manager vs Parameter Store**: rotation → Secrets Manager; config thường, tiết kiệm → Parameter Store.
8. **ACM chỉ cho AWS-managed services** (ALB, CloudFront, API Gateway...).
9. **WAF tại CloudFront/ALB/API Gateway/AppSync/Cognito** — không trực tiếp EC2/NLB.
10. **STS AssumeRole**: temp credentials 15 phút – 12 giờ (chain role max 1 giờ).
11. **MFA trong IAM policy** → condition `aws:MultiFactorAuthPresent: true`.
12. **KMS key rotation**: AWS Managed 1 năm (auto), Customer Managed 1 năm (bật được), Imported/Asymmetric không auto rotate.
13. **CloudTrail vs Config**: CloudTrail = "ai làm gì khi nào" (API call); Config = "resource có compliant không" (state + history).
14. **GuardDuty** dùng ML trên log có sẵn — không cần deploy agent.

---

> **Tiếp theo:** [Phần 5: Deployment & Monitoring (CI/CD, CloudFormation, SAM, CDK, CloudWatch, X-Ray)](/posts/dva-c02-part5-deploy-monitor)
{: .prompt-info }
