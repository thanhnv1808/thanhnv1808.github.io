# DVA-C02 — Phần 4: Security & Identity

> **Chuỗi tài liệu:** Phần 1 (Compute) → Phần 2 (Storage & DB) → Phần 3 (App Integration) → **Phần 4 (Security)** → Phần 5 (Deploy & Monitor) → Phần 6 (Networking + Tips).

---

## 1. IAM (Identity & Access Management) — ⭐ CỰC CỰC QUAN TRỌNG

### 1.1 Các khái niệm cốt lõi

- **User**: đại diện người/ứng dụng, có credentials (password, access keys).
- **Group**: tập hợp user, áp dụng policy chung. **KHÔNG phải identity**, không chứa user khác được.
- **Role**: identity **tạm thời** với permission, được **assume** bởi trusted entity (user, service, account khác). **Không có credentials dài hạn** — dùng STS sinh temporary credentials.
- **Policy**: JSON định nghĩa permissions.

### 1.2 Các loại Policy

| Loại | Mô tả | Use case |
|---|---|---|
| **AWS Managed** | AWS tạo, quản lý (VD: `AmazonS3ReadOnlyAccess`) | Tiện, phổ biến |
| **Customer Managed** | Bạn tạo, quản lý | Custom, reusable |
| **Inline** | Gắn trực tiếp vào user/group/role | One-off, tight coupling |
| **Resource-based** | Gắn vào resource (S3 bucket policy, SQS policy, KMS key policy...) | Cross-account, external access |
| **Permission Boundary** | Giới hạn MAX permission của user/role | Delegation — admin giới hạn quyền cho dev tạo role |
| **SCP** (AWS Organizations) | Giới hạn permission account con | Governance |
| **Session Policy** | Truyền khi AssumeRole → giới hạn session | Fine-grained temp |
| **ABAC** (Attribute-based) | Policy dùng tag để control | Scale với team lớn |

### 1.3 Cấu trúc IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "OptionalID",
    "Effect": "Allow" | "Deny",
    "Principal": "...",   // chỉ resource-based
    "Action": ["s3:GetObject"],
    "Resource": ["arn:aws:s3:::bucket/*"],
    "Condition": { "StringEquals": { "aws:username": "John" } }
  }]
}
```

### 1.4 Evaluation Logic — **CỰC KỲ QUAN TRỌNG**

Thứ tự đánh giá khi 1 request tới:

1. **Explicit Deny** ở bất kỳ policy nào → **DENY**.
2. **Explicit Allow** ở ít nhất 1 policy áp dụng → **ALLOW**.
3. Không có match → **Implicit Deny**.

Với cross-account access:
- **Cả bên source VÀ bên destination đều phải allow**.
- Bên source: IAM policy của principal.
- Bên destination: resource-based policy HOẶC bên source account trust.

> ⚠️ **Bẫy phổ biến:** Explicit Deny **luôn thắng** explicit Allow. SCP deny → ngay cả root user account con cũng không qua được.

### 1.5 IAM Policy Conditions — hay xuất hiện

| Condition Key | Dùng cho |
|---|---|
| `aws:SourceIp` | Giới hạn IP |
| `aws:SourceVpc`, `aws:SourceVpce` | Giới hạn VPC / VPC Endpoint |
| `aws:MultiFactorAuthPresent` | Bắt buộc MFA |
| `aws:RequestedRegion` | Giới hạn region |
| `aws:PrincipalTag/<key>` | Tag của principal (ABAC) |
| `aws:ResourceTag/<key>` | Tag của resource (ABAC) |
| `s3:prefix`, `s3:x-amz-server-side-encryption` | Service-specific |
| `aws:SecureTransport` | Bắt HTTPS |

### 1.6 Best Practices cho DVA-C02

1. **Không dùng root user** cho daily task. Bật MFA cho root.
2. **Principle of least privilege**.
3. **Dùng Role thay vì Access Key** khi có thể (EC2 Instance Profile, ECS Task Role, Lambda execution role).
4. **Không hardcode credentials** trong code — dùng role / Secrets Manager / SSM Parameter Store.
5. **Rotate access keys** định kỳ.
6. **IAM Access Analyzer**: phát hiện resource chia sẻ với external entity.
7. **IAM Credential Report / Access Advisor**: audit.

### 1.7 IAM Roles — các scenario hay hỏi

- **EC2 Instance Profile**: gắn role vào EC2 → app chạy trong EC2 dùng role thay access key.
- **ECS Task Role**: như đã nói Phần 1.
- **Lambda Execution Role**: Lambda assume để gọi AWS service.
- **Cross-account Role**: account A tạo role cho account B assume. B gọi `sts:AssumeRole` → temporary credentials của A.
- **Service-Linked Role**: AWS service tự tạo (VD: ASG, ELB) — không sửa được.

### 1.8 EC2 Instance Metadata & Role Credentials

- EC2 lấy temp credentials qua **IMDS**:
  ```
  curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
  ```
- Credentials auto-rotate (thường 6 giờ).
- SDK tự retrieve — developer không cần hardcode gì.

> ⚠️ **Nhớ:** Luôn dùng **IMDSv2** (token-based) — v1 có nguy cơ SSRF.

---

## 2. AWS STS (Security Token Service)

### 2.1 Các API chính

| API | Use case |
|---|---|
| **AssumeRole** | Same/cross-account role assume |
| **AssumeRoleWithSAML** | SAML 2.0 federation (enterprise SSO) |
| **AssumeRoleWithWebIdentity** | Web identity (Google, Facebook, Cognito) — **thường khuyên dùng Cognito thay vì gọi trực tiếp** |
| **GetSessionToken** | MFA session với long-term credentials |
| **GetFederationToken** | Federation user không có IAM (legacy) |
| **DecodeAuthorizationMessage** | Decode encoded error từ IAM |

### 2.2 Temporary Credentials

- Trả về `AccessKeyId`, `SecretAccessKey`, `SessionToken`, `Expiration`.
- Duration: **15 phút – 12 giờ** (default 1 giờ; với chain role, max 1 giờ).
- Không thể revoke (đợi expire) — trừ khi revoke session bằng cách edit role policy với condition `aws:TokenIssueTime`.

### 2.3 Cross-account pattern

1. Account A tạo **Role** với trust policy cho Account B.
2. User/service ở B gọi `sts:AssumeRole` với ARN của role A.
3. STS trả temp credentials của A.
4. B dùng creds đó gọi API trong A.

---

## 3. Amazon Cognito — ⭐ QUAN TRỌNG

### 3.1 User Pools vs Identity Pools — THUỘC LÒNG

| | **User Pool** | **Identity Pool** (Federated Identities) |
|---|---|---|
| Mục đích | **Authentication (Who are you?)** | **Authorization (What can you do?)** — grant AWS credentials |
| Output | **JWT token** (ID, Access, Refresh) | **Temporary AWS credentials** từ IAM Role |
| User directory | Quản lý user (sign-up/sign-in) | Không — nhận identity từ provider khác |
| Federation | Facebook, Google, Apple, SAML, OIDC | User Pool, Cognito, Facebook, Google, SAML, OIDC, developer-authenticated |
| Integration | API Gateway authorizer, ALB, AppSync | Truy cập S3, DynamoDB... trực tiếp từ mobile/web |

> 💡 **Pattern phổ biến:**
> - **User Pool only**: API Gateway authorizer → backend Lambda/ECS.
> - **User Pool + Identity Pool**: user login qua User Pool → token → Identity Pool trả AWS creds → client gọi S3/DynamoDB trực tiếp.

### 3.2 Cognito User Pool — tính năng

- **Sign-up, sign-in**, forgot password, MFA (SMS, TOTP).
- **Social/SAML federation**.
- **Hosted UI** (có thể customize).
- **Triggers (Lambda)**:
  - Pre/Post Sign-up, Pre/Post Authentication, Pre Token Generation, Migrate User, Custom Message...
  - Use case: custom validation, migrate user từ DB cũ, modify JWT claims.
- **User groups**: gán role ARN → Identity Pool dùng role này khi user trong group.
- **App Clients**: đại diện 1 app, có `client_id` và optional `client_secret`.
- **OAuth 2.0 flows**: Authorization Code (khuyến nghị), Implicit (deprecated), Client Credentials.

### 3.3 Cognito Identity Pool — tính năng

- **Authenticated users**: có identity từ provider → role "Authenticated".
- **Unauthenticated (guest) users**: role "Unauthenticated" → truy cập hạn chế (VD: xem giá, đọc metadata).
- **Fine-grained role selection**: chọn role dựa trên Cognito group hoặc rule.

### 3.4 Cognito Sync (deprecated) → **AWS AppSync** thay thế.

### 3.5 JWT Tokens

- **ID Token**: chứa identity claims (email, name...). Dùng cho authentication.
- **Access Token**: scope để gọi API.
- **Refresh Token**: đổi lấy ID/Access mới khi hết hạn (default 30 ngày).

---

## 4. AWS KMS (Key Management Service) — ⭐ QUAN TRỌNG

### 4.1 Các loại key

| Loại | Mô tả |
|---|---|
| **AWS Owned Keys** | AWS quản lý hoàn toàn, không nhìn thấy, free |
| **AWS Managed Keys** | `aws/service-name` — AWS tạo cho service (VD `aws/s3`), không tính phí key, rotate tự động 1 năm |
| **Customer Managed Keys (CMK)** | Bạn tạo, quản lý. **$1/month/key**. Rotate tự động/thủ công |
| **Imported Keys** | BYOK (Bring Your Own Key) — import material |
| **Custom Key Store** | Dùng CloudHSM |

### 4.2 Symmetric vs Asymmetric

| | **Symmetric** (AES-256) | **Asymmetric** (RSA, ECC) |
|---|---|---|
| Use case | Encrypt/Decrypt AWS service data | Sign/Verify, Encrypt/Decrypt ở client without symmetric key share |
| Key material lộ ra ngoài KMS? | **KHÔNG bao giờ** | Public key ra được, private thì không |
| AWS service integrate | Phần lớn | Ít |

### 4.3 Giới hạn quan trọng

- **Encrypt/Decrypt API: data tối đa 4 KB** → data lớn hơn phải dùng **Envelope Encryption**.
- **Regional**: KMS key không ra khỏi region (trừ Multi-Region Keys).

### 4.4 Envelope Encryption

Pattern để encrypt data lớn:
1. Call `GenerateDataKey` → KMS trả **plaintext data key** + **encrypted data key (ciphertext)**.
2. Client dùng plaintext data key encrypt data (AES-256 local).
3. Lưu encrypted data key **cùng với encrypted data**.
4. Xóa plaintext data key khỏi memory ngay.
5. Khi decrypt: call `Decrypt` với encrypted data key → plaintext data key → dùng để decrypt data.

**AWS Encryption SDK** làm việc này sẵn cho bạn. **DynamoDB Encryption Client**, **S3 Encryption Client** cũng dùng pattern này.

### 4.5 Key Policies & Grants

- **Key Policy**: resource-based policy gắn vào KMS key.
  - Mặc định khi tạo CMK: cho root account full access → cho IAM policy điều chỉnh.
  - **KMS KHÔNG có implicit allow** từ IAM admin nếu key policy không cho phép. **PHẢI edit key policy** nếu cần ai đó access key (không thể bypass bằng IAM policy).
- **Grants**: delegate temporary permission, thường dùng khi AWS service cần access key cho user.
- **ViaService condition**: giới hạn key chỉ dùng được qua service cụ thể.

> ⚠️ **Bẫy kinh điển:** "User có AdministratorAccess IAM policy nhưng không dùng được KMS key?" → Key policy chưa grant → sửa key policy.

### 4.6 Key Rotation

- **AWS Managed**: auto rotate mỗi **1 năm**, không cấu hình được.
- **Customer Managed Symmetric**: bật rotation → auto mỗi **1 năm** (hoặc custom từ 90-2560 ngày).
- Rotation giữ nguyên key ID, chỉ thay backing key material. Data encrypted bằng old material vẫn decrypt được.
- **Imported keys**: không auto rotate — phải manual bằng cách tạo key mới.
- **Asymmetric keys**: không hỗ trợ automatic rotation.

### 4.7 Multi-Region Keys

- Replicate key sang region khác **với cùng key ID**.
- Dùng cho global app, DR.
- Encrypted data ở region A decrypt được ở region B **mà không cần re-encrypt**.

### 4.8 KMS Request Quotas

- Per-region quota: thường **10,000 req/s** cho `Encrypt/Decrypt/GenerateDataKey*` với symmetric.
- Throttle → `ThrottlingException` (retry với exponential backoff).
- **S3 Bucket Keys** giảm KMS call 99% (đã đề cập Phần 2).

---

## 5. AWS Secrets Manager

### 5.1 Đặc điểm

- Lưu secret (password, API key, OAuth token...).
- **Auto-rotation** bằng Lambda (built-in cho RDS, DocumentDB, Redshift; custom cho khác).
- Encrypt bằng KMS (AWS Managed hoặc CMK).
- Gắn **resource-based policy** để cross-account.
- **Giá**: **$0.40/secret/tháng + $0.05/10,000 API calls**.

### 5.2 Secrets Manager vs SSM Parameter Store — ⭐ PHẢI PHÂN BIỆT

| | **Secrets Manager** | **SSM Parameter Store** |
|---|---|---|
| Giá | $0.40/secret/tháng | **Miễn phí** (Standard), $0.05/param/tháng (Advanced) |
| Auto rotation | ✅ Built-in | ❌ (tự làm qua Lambda + EventBridge) |
| Size | 64 KB | Standard: 4 KB, Advanced: 8 KB |
| Param history | ❌ (chỉ version) | ✅ (Advanced) |
| Cross-account (resource policy) | ✅ | ❌ (chỉ IAM) |
| Built-in integration với RDS | ✅ | ❌ |
| Hierarchy (`/app/prod/db-url`) | ❌ | ✅ |
| Tham chiếu trong CloudFormation | `{{resolve:secretsmanager:...}}` | `{{resolve:ssm:...}}` / `{{resolve:ssm-secure:...}}` |

> 💡 **Quyết định nhanh:**
> - Cần **auto-rotation** (VD: DB password) → **Secrets Manager**.
> - Config thông thường, không nhạy cảm lắm, chi phí là yếu tố → **SSM Parameter Store**.

### 5.3 Rotation pattern

Lambda rotation function có 4 step:
1. **createSecret**: tạo secret mới.
2. **setSecret**: apply new secret vào service.
3. **testSecret**: verify new secret works.
4. **finishSecret**: mark new secret là `AWSCURRENT`, old thành `AWSPREVIOUS`.

---

## 6. AWS Systems Manager Parameter Store

### 6.1 Parameter types

- **String**: plain text.
- **StringList**: comma-separated.
- **SecureString**: encrypt bằng KMS.

### 6.2 Tiers

- **Standard**: 10,000 params, 4 KB mỗi, không phí.
- **Advanced**: 100,000 params, 8 KB, $0.05/param/tháng, có **policies (TTL, expiration notification)**.

### 6.3 Hierarchical naming

```
/my-app/dev/db-url
/my-app/dev/db-password
/my-app/prod/db-url
```
→ `GetParametersByPath` với path `/my-app/dev/` lấy hết param của env dev.

### 6.4 Parameter Policies (Advanced tier)

- **Expiration**: tự xóa sau thời gian.
- **ExpirationNotification**: gửi EventBridge trước X ngày.
- **NoChangeNotification**: alert nếu không update trong X ngày.

---

## 7. AWS Certificate Manager (ACM)

- **Miễn phí** SSL/TLS certificate cho AWS services (ALB, NLB listener TLS, CloudFront, API Gateway).
- **Public certs**: auto-renew.
- **Private certs**: qua ACM Private CA.
- **KHÔNG export** được private key của public cert (trừ khi ACM Private).
- Chỉ dùng trong AWS services — **EC2 không dùng được** ACM trực tiếp.

> ⚠️ **Bẫy:** "Cert cho ALB" → **ACM**. "Cert cho EC2 web server tự host Nginx" → **KHÔNG** dùng ACM được (phải cài cert khác hoặc đặt sau ALB với ACM).

---

## 8. AWS WAF (Web Application Firewall)

### 8.1 Khái niệm

- Layer 7 firewall bảo vệ web app khỏi SQLi, XSS, bad bots, DDoS L7.
- Deploy tại: **CloudFront, ALB, API Gateway, AppSync, Cognito User Pool**.
- **KHÔNG** deploy trực tiếp trên EC2/NLB.

### 8.2 Rules

- **Managed Rules** (AWS & Marketplace): Common, Known Bad Inputs, SQLi, Linux, ...
- **Custom Rules**: match theo IP, country, rate, string, regex, size, headers.
- **Rate-based rule**: giới hạn request rate từ 1 IP.
- **Rule Actions**: Allow, Block, Count, CAPTCHA, Challenge.

### 8.3 Web ACL

- Tập các rule + default action (Allow/Block).
- Scope: CloudFront (global) hoặc Regional.

---

## 9. AWS Shield

- **Shield Standard**: miễn phí, auto-applied. L3/L4 DDoS protection.
- **Shield Advanced**: $3,000/tháng, coverage CloudFront/ALB/NLB/R53/Global Accelerator, 24/7 DRT support, cost protection.

---

## 10. AWS Firewall Manager

- Quản lý tập trung WAF, Shield, Security Group, Route 53 Resolver DNS Firewall qua nhiều account (Organizations).

---

## 11. Các dịch vụ security khác — biết sơ

| Service | Mục đích |
|---|---|
| **Amazon GuardDuty** | Threat detection từ VPC Flow Logs, CloudTrail, DNS logs (ML-based) |
| **Amazon Inspector** | Vulnerability scan EC2, ECR image, Lambda |
| **Amazon Macie** | Discover PII trong S3 (ML-based) |
| **AWS CloudTrail** | Log mọi API call, audit trail. Management events (default), Data events (opt-in, tốn phí) |
| **AWS Config** | Track config change của resource, compliance rule |
| **AWS Artifact** | Portal download compliance report (SOC, PCI, ISO) |
| **AWS Detective** | Investigate security finding |
| **AWS Security Hub** | Aggregate finding từ GuardDuty, Inspector, Macie, Config |

> 💡 **Decision tree cho "phát hiện X":**
> - Threat real-time (bitcoin mining, compromised instance) → **GuardDuty**.
> - Vulnerability trong EC2/ECR/Lambda → **Inspector**.
> - PII trong S3 → **Macie**.
> - API call audit → **CloudTrail**.
> - Config compliance (VD: S3 phải encrypted) → **AWS Config**.

---

## 12. Signed URLs — So sánh S3 vs CloudFront

| | **S3 Pre-signed URL** | **CloudFront Signed URL / Cookie** |
|---|---|---|
| Scope | 1 object cụ thể | 1 file hoặc nhiều file (cookie) |
| Auth backing | IAM creds của người tạo URL | CloudFront Key Pair (public/private) |
| Use case | Tạm thời cho user upload/download | Content distribution với expiration, IP restriction, bảo vệ global |
| Expiration | Short (đến 7 ngày với SigV4) | Custom, dài hơn |

> 💡 **Pattern:** User login → app generate pre-signed URL → user upload lên S3 trực tiếp (tránh proxy qua backend, tiết kiệm).

---

## 🎯 Tip ôn thi Phần 4

1. **IAM evaluation logic**: Explicit Deny → Explicit Allow → Implicit Deny. **Deny luôn thắng**.

2. **Cross-account access cần 2 bên đều cho phép**:
   - Source account: IAM policy của principal.
   - Destination: resource-based policy HOẶC trust.

3. **EC2 → luôn dùng Instance Profile (role)**, không hardcode access key. Credentials từ **IMDSv2**.

4. **Cognito User Pool vs Identity Pool**:
   - User Pool = authentication (JWT token).
   - Identity Pool = authorization (AWS temp creds).
   - Pattern thường: User Pool → Identity Pool → truy cập AWS service từ client.

5. **KMS bẫy lớn nhất**: admin IAM không có nghĩa là dùng được CMK. **Key Policy phải cho phép**.

6. **KMS Encrypt API giới hạn 4 KB** → data lớn hơn → **Envelope Encryption** với `GenerateDataKey`.

7. **Secrets Manager vs Parameter Store**:
   - Cần auto-rotation → Secrets Manager.
   - Config thường, tiết kiệm → Parameter Store.
   - Secret < 4 KB không rotate → Parameter Store SecureString vẫn được.

8. **ACM chỉ dùng cho AWS-managed services** (ALB, CloudFront, API Gateway...). EC2 self-managed không xài được.

9. **WAF deploy tại CloudFront/ALB/API Gateway/AppSync/Cognito** — không trực tiếp EC2/NLB.

10. **STS AssumeRole** cho cross-account, temp credentials 15 phút – 12 giờ (chain role max 1 giờ).

11. **MFA trong IAM policy** → condition `aws:MultiFactorAuthPresent: true`.

12. **S3 Pre-signed URL** kế thừa quyền người tạo — người tạo cần `s3:GetObject`/`PutObject`.

13. **KMS key rotation**: AWS Managed 1 năm (auto, không config), Customer Managed default 1 năm (bật tắt được), Imported/Asymmetric không auto rotate.

14. **CloudTrail vs Config**: CloudTrail = "ai làm gì khi nào" (API call), Config = "resource có compliant không" (state + history).

15. **GuardDuty** dùng ML trên log có sẵn (VPC Flow, CloudTrail, DNS) → không cần deploy agent.

---

**→ Sẵn sàng tiếp tục với Phần 5 (Deployment & Monitoring)?**
