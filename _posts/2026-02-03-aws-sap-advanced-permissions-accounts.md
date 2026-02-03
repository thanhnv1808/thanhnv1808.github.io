---
title: "AWS SAP - Advanced Permissions & Accounts"
author: thanhnv1808
date: 2026-02-03 10:00:00 +0700
categories: [AWS, Solutions Architect Professional]
tags: [aws, sap, iam, organizations, scp, sts, permissions, cross-account, ram, security]
description: Comprehensive guide to Advanced Permissions & Accounts for AWS Solutions Architect Professional (SAP) exam. Covers Organizations, SCPs, STS, Permissions Boundaries, Cross-Account Access, and more.
pin: false
comments: true
---

This lesson covers **Advanced Permissions & Accounts** - one of the most critical domains for the AWS Solutions Architect Professional (SAP) exam, representing 15-20% of Domain 1: Design for Organizational Complexity.

## Overview

| Topic | Exam Weight |
|-------|-------------|
| AWS Organizations | High |
| Service Control Policies (SCP) | High |
| Security Token Service (STS) | High |
| Permissions Boundaries | Medium |
| Cross-Account Access | High |
| AWS RAM | Medium |

---

## 1. AWS Organizations

### What is AWS Organizations?

AWS Organizations is a **FREE** account management service that consolidates multiple AWS accounts into a hierarchical structure.

```
Management Account (root of organization)
├── Organizational Unit: Production
│   ├── Account: Prod-App1
│   └── Account: Prod-Database
├── Organizational Unit: Development
│   ├── Account: Dev-Sandbox
│   └── Account: Dev-Testing
└── Organizational Unit: Security
    └── Account: Security-Logging
```

### Key Features

| Feature | Description |
|---------|-------------|
| Consolidated Billing | One bill, volume discounts, RI sharing |
| Service Control Policies | Permission boundaries at OU/account level |
| Account Creation | Programmatically create accounts via API |
| Centralized Management | Tag policies, backup policies |

### Management Account - Important Points

> **SAP Exam Trap:** SCPs do NOT apply to the management account. If asked "how to restrict the management account," the answer is IAM policies only.
{: .prompt-warning }

- Pays the bill for entire organization
- Can apply SCPs but is **NOT affected by SCPs itself**
- Should have minimal workloads (billing/governance only)
- Cannot leave the organization

### [DEMO] AWS Organizations

**Demo Steps:**
1. Add 2 accounts: Production and Development
   - 1 account invited
   - 1 account created from management account
2. From management account, switch between PROD and DEV

> **Warning:** If you get an error "You have exceeded the allowed number of AWS Accounts", request a quota increase at [Service Quotas](https://console.aws.amazon.com/servicequotas/home?region=us-east-1#!/services/organizations/quotas/L-29A0C5DF)
{: .prompt-warning }

---

## 2. Service Control Policies (SCP)

### What Are SCPs?

SCPs are **permission filters** (boundaries) that define the maximum permissions for accounts in an organization.

> **Critical Mental Model:** SCPs act as a filter on IAM permissions. They can only **RESTRICT**, never **GRANT**.
{: .prompt-tip }

### SCP Strategies

**1. Allow List Strategy (Default)**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

**2. Deny List Strategy - Prevent Leaving Organization**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```

**3. Enforce Regional Restrictions (GDPR Compliance)**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
      }
    }
  ]
}
```

### SCP Inheritance Model

```
Root OU (SCP: Deny S3)
└── Prod OU (SCP: Deny EC2)
    └── Account A → Has BOTH restrictions
```

> **SAP Exam Key:** The most restrictive combination wins. All SCPs from root to account are evaluated.
{: .prompt-info }

### What SCPs Do NOT Affect

- **Management account** (immune to SCPs)
- **Service-linked roles** (AWS-managed)
- Operations done by AWS services on your behalf

### [DEMO] Using Service Control Policies

**Demo Overview:**
- Update the structure within the organization
- Apply an SCP to the PRODUCTION account to test their capabilities

**Demo Steps:**
1. PROD account adds bucket with cat image:

![Samson Cat](/assets/img/aws-sap/samson.jpeg){: width="400" }
_Cat image for S3 bucket demo_

2. Demo S3 access, enable Service control policies on org for PROD to see S3 bucket
3. Create SCP to "Allow All Except S3" → Add to PROD → Deny S3 so PROD cannot access S3

**denyS3.json:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```

4. Clean up: Delete resources after demo

---

## 3. AWS Security Token Service (STS)

### What is STS?

STS creates **temporary, limited-privilege credentials** for AWS identities:
1. Access Key ID (starts with ASIA...)
2. Secret Access Key
3. Session Token
4. Expiration timestamp

### Core STS APIs

| API | Duration | Use Case |
|-----|----------|----------|
| AssumeRole | 15 min - 12 hours | Cross-account, EC2 instance profiles |
| AssumeRoleWithSAML | Up to 12 hours | Enterprise SSO (AD, Okta) |
| AssumeRoleWithWebIdentity | 15 min - 12 hours | Mobile apps (Google, Facebook) |
| GetSessionToken | 15 min - 36 hours | MFA-protected API access |
| GetFederationToken | 15 min - 36 hours | Identity brokers |

### Session Policies

When assuming a role, you can pass an additional session policy:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --role-session-name mysession \
  --policy '{"Version":"2012-10-17","Statement":[...]}'
```

**Effective permissions = intersection of:**
1. Role's identity policy
2. Session policy (if provided)
3. Any SCPs on the account
4. Any permissions boundaries on the role

---

## 4. Revoking IAM Role Temporary Credentials

### The Challenge

Temporary credentials issued by STS **cannot be deleted**. They remain valid until expiration.

> **Key Concept:** If credentials leak, what do you change? Trust policy or permissions policy? Temp credentials cannot be invalidated on-demand, so a specific technique is needed.
{: .prompt-info }

### The Solution: Inline Deny Policy with Time Condition

**Step 1:** Note the current time (e.g., 2026-02-03T14:30:00Z)

**Step 2:** Attach an inline deny policy to the role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "DateLessThan": {
          "aws:TokenIssueTime": "2026-02-03T14:30:00Z"
        }
      }
    }
  ]
}
```

**How it works:**
- All credentials issued BEFORE 14:30 are denied (revoked)
- New AssumeRole calls after 14:30 work normally
- Old sessions fail immediately, even if not expired

> **SAP Exam Pattern:** "A role has been compromised. How can you immediately revoke access?" → Inline deny policy with `aws:TokenIssueTime` condition
{: .prompt-tip }

### [DEMO] Revoking Temporary Credentials - PART 1

![Revoking Credentials Demo](/assets/img/aws-sap/revoking-credentials-demo.png)
_Demo architecture for revoking temporary credentials_

**1-Click Deployment:**
[Deploy CloudFormation Stack](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0038-aws-pro-revoking-temporary-credentials/A4LHostingInc.yaml&stackName=A4L)

**Lesson Commands:**

```bash
# ON EC2
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Get the role name
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/REPLACE_ME
```

**Local Machine - LINUX or MACOS:**
```bash
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=
aws s3 ls
aws ec2 describe-instances --region us-east-1
```

**Local Machine - WINDOWS:**
```cmd
SET AWS_ACCESS_KEY_ID=AKID
SET AWS_SECRET_ACCESS_KEY=SAK
SET AWS_SESSION_TOKEN=TOKEN
aws s3 ls
aws ec2 describe-instances --region us-east-1
```

**ON EC2 (Test Invalidation):**
```bash
aws ec2 describe-instances --region us-east-1
aws s3 ls
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Get the role name
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/REPLACE_ME
```

**ON EC2 (After Stop and Start):**
```bash
aws ec2 describe-instances --region us-east-1
aws s3 ls
```

### [DEMO] Revoking Temporary Credentials - PART 2

Continue the security incident demo and clean up resources after completion.

---

## 5. Policy Interpretation Deep Dive

### The Policy Evaluation Framework

```
1. Explicit Deny (anywhere) → DENY
2. Explicit Allow (with no denies) → ALLOW
3. Implicit Deny (default) → DENY
```

### Example 1: Holiday Gifts Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:GetObject",
        "s3:GetObjectAcl",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::holidaygifts/*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectAcl"
      ],
      "Resource": "arn:aws:s3:::holidaygifts/*",
      "Condition": {
        "DateGreaterThan": {"aws:CurrentTime": "2022-12-01T00:00:00Z"},
        "DateLessThan": {"aws:CurrentTime": "2022-12-25T06:00:00Z"}
      }
    }
  ]
}
```

**Analysis:** Users can upload/delete objects anytime, but reading objects is denied between Dec 1 and Dec 25 (holiday gift surprise protection).

### Example 2: Deny Non-Approved Regions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Deny NonApproved Regions",
      "Effect": "Deny",
      "NotAction": [
        "cloudfront:*",
        "iam:*",
        "route53:*",
        "support:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-southeast-2",
            "eu-west-1"
          ]
        }
      }
    }
  ]
}
```

**Analysis:** Denies all actions outside ap-southeast-2 and eu-west-1, EXCEPT for global services (CloudFront, IAM, Route53, Support).

### Example 3: S3 Home Directory Pattern

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::cl-animals4life",
      "Condition": {
        "StringLike": {
          "s3:prefix": [
            "",
            "home/",
            "home/${aws:username}/*"
          ]
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::cl-animals4life/home/${aws:username}",
        "arn:aws:s3:::cl-animals4life/home/${aws:username}/*"
      ]
    }
  ]
}
```

**Analysis:** Users can list all buckets, list the home/ prefix, but only have full access to their own home directory (`home/${aws:username}/`).

---

## 6. Permissions Boundaries

### What Are Permissions Boundaries?

A **guardrail for delegation** that sets the maximum permissions an IAM entity can have.

```
Effective Permissions = Identity Policy AND Permissions Boundary
```

### Use Case: Safe Delegation

**Problem:** Delegate IAM user/role creation without privilege escalation.

**Solution:** Require boundary attachment:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateUser",
        "iam:CreateRole",
        "iam:AttachUserPolicy",
        "iam:AttachRolePolicy"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "iam:PermissionsBoundary": "arn:aws:iam::123456789012:policy/TeamBoundary"
        }
      }
    }
  ]
}
```

> **SAP Exam Pattern:** "How can you delegate IAM role creation while preventing privilege escalation?" → Permissions boundaries with required attachment condition
{: .prompt-tip }

### Reference Policies

- [User Boundary Policy](https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0055-aws-mixed-iam-permissions-boundaries/01_a4luserboundary.json)
- [Admin Boundary Policy](https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0055-aws-mixed-iam-permissions-boundaries/02_a4ladminboundary.json)
- [Admin Permissions Policy](https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0055-aws-mixed-iam-permissions-boundaries/03_a4ladminpermissionspolicy.json)

---

## 7. AWS Permissions Evaluation Logic

### The Full Decision Flow

![Permissions Evaluation](/assets/img/aws-sap/permissions-evaluation.png)
_AWS Permissions Evaluation Flow_

> **At the professional level, understanding this process is essential.** The same process is used each time an identity attempts to access a resource.
{: .prompt-info }

```
Request Made
    ↓
Is there an explicit DENY anywhere?
    YES → DENY (stop)
    NO → continue
    ↓
Is account in Organization?
    YES → Does SCP allow? NO → DENY
    YES → continue
    ↓
Is this cross-account?
    YES → Does resource policy allow? NO → DENY
          Does identity policy allow? NO → DENY
    NO → Does identity policy allow? NO → DENY
    ↓
Is there a permissions boundary?
    YES → Does it allow? NO → DENY
    ↓
Is there a session policy?
    YES → Does it allow? NO → DENY
    ↓
ALLOW
```

---

## 8. Cross-Account Access to S3

### Three Patterns Comparison

| Feature | IAM Roles | Bucket Policies | ACLs |
|---------|-----------|-----------------|------|
| Object ownership | Target account | Source account | Source account |
| Audit trail | AssumeRole events | Direct access | Direct access |
| Scalability | High | Medium | Low |
| AWS recommendation | **Preferred** | Acceptable | Legacy |

### [DEMO] Cross Account Access to S3 - SETUP (STAGE 1)

![Cross Account S3 Demo](/assets/img/aws-sap/cross-account-s3-demo.png)
_Cross-account S3 access demo architecture_

**Values to Note:**
- IAMUSER bob PASSWORD
- Account ID of the GENERAL ACCOUNT
- Canonical User ID for the GENERAL ACCOUNT
- Name of CrossAccountS3Role
- Account ID of the PRODUCTION Account
- Console URL Access URL for the petpics1 bucket
- Console URL Access URL for the petpics2 bucket
- Console URL Access URL for the petpics3 bucket

**1-Click Deployment - PRODUCTION ACCOUNT:**
[Deploy CloudFormation Stack](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0039-aws-mixed-cross-account-s3/prod_bucketsandrole.yaml&stackName=buckets)

**Bucket Images:**

| Bucket | Image |
|--------|-------|
| petpics1 | ![Ginny](/assets/img/aws-sap/bucket_images/petpics1/ginny.jpg){: width="200" } |
| petpics2 | ![Hermione](/assets/img/aws-sap/bucket_images/petpics2/hermione.jpg){: width="200" } |
| petpics3 | ![Maxi](/assets/img/aws-sap/bucket_images/petpics3/maxi.jpg){: width="200" } |

**Bucket Policy Template:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::REPLACEMEMANAGEMENTACCOUNTID:user/iamadmin"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::REPLACEME_BUCKETNAME/*",
                "arn:aws:s3:::REPLACEME_BUCKETNAME"
            ]
        }
    ]
}
```

### [DEMO] Cross Account Access to S3 - ACL (STAGE 2)

Experience cross-account S3 access using **ACL** method.

### [DEMO] Cross Account Access to S3 - BUCKET POLICY (STAGE 3)

Experience cross-account S3 access using **Bucket Policy** method.

### Object Ownership Rules

> **Critical for SAP Exam:**
{: .prompt-warning }

| Scenario | Object Owner |
|----------|--------------|
| IAM user uploads via bucket policy | **Uploader's account** (source) |
| User assumes role then uploads | **Role's account** (target) |

**Solution for bucket policy uploads:** Use S3 Object Ownership "Bucket owner enforced" setting or require `bucket-owner-full-control` ACL.

---

## 9. AWS Resource Access Manager (RAM)

### What Can RAM Share?

| Shareable | Not Shareable |
|-----------|---------------|
| VPC subnets | S3 buckets |
| Transit Gateway | KMS keys |
| Route 53 Resolver rules | Lambda functions |
| EC2 Dedicated Hosts | IAM roles |

> **SAP Exam Trap:** RAM shares **VPC subnets**, NOT entire VPCs.
{: .prompt-warning }

### VPC Sharing Architecture

```
Central Networking Account (Owner)
├── Manages: VPC, subnets, route tables, IGW, NAT
│
Participant Accounts
├── Launch: EC2, RDS, Lambda into shared subnets
└── Manage: Own security groups, own resources
```

**Benefits:**
- Single VPC, many accounts (up to 100 participants)
- Centralized network management
- Cost savings (shared NAT Gateways)

---

## 10. Service Quotas

### Key Quotas for SAP Exam

| Service | Quota | Type |
|---------|-------|------|
| IAM Users | 5,000 | Soft |
| IAM Roles | 1,000 | Soft |
| S3 Buckets | 100 (can increase to 1,000) | Soft |
| VPCs per region | 5 | Soft |
| Accounts in Org | 1,000 | Soft |

### Multi-Account Strategy

**Problem:** Hitting VPC quota (5 per region)

**Solutions:**
- Request quota increase
- Multi-account strategy (each account gets 5 VPCs)
- VPC sharing via AWS RAM

---

## Quiz

### Q1: What functionality does STS provide?
- [ ] It allows you to create and manage identities
- [ ] It securely stores and manages long term credentials and their rotation
- [ ] It provides a managed, hierarchical store of configuration documents (YAML and JSON)
- [x] **It generates short term credentials which can be used to interact with AWS resources**
- [ ] It lets you create user and identity pools for ID federation

### Q2: Which is NOT a 'real' identity that can be referenced by ARNs in resource policies?
- [ ] IAM Users
- [x] **IAM Groups**
- [ ] IAM Role
- [ ] AWS Account
- [ ] AWS Account Root User

### Q3: How are role sessions revoked?
- [ ] The credentials are explicitly expired by STS
- [x] **A inline policy is added to the role with an explicit deny for role assumptions before .. NOW**
- [ ] A inline policy is added to the temporary credentials with an explicit deny for generations before .. NOW
- [ ] The role is disabled and a new role generated with the same name

### Q4: SCP allows S3, managed policy allows S3, inline policy denies S3 - effective access?
- [x] **Denied**
- [ ] Allowed
- [ ] It depends on the policy priority order
- [ ] It depends if the AWS Account is a master or member account

### Q5: Cross-account: SCP denies S3 in Account B, resource policy allows, identity policy allows - effective access?
- [x] **Denied**
- [ ] Allowed
- [ ] Not enough information to decide
- [ ] It depends if account B is a master or member account

### Q6: Can RAM share VPCs between accounts?
- [ ] No
- [ ] Yes
- [ ] It depends if the accounts are in the same AWS Org
- [ ] It depends if the VPCs are in the same region
- [x] **No - but it can share subnets**

### Q7: What AWS feature allows admin permissions delegation?
- [ ] IAM Roles
- [ ] IAM Users
- [ ] SCP
- [x] **Policy Boundaries**
- [ ] Resource Policies

### Q8: Who owns objects when IAM users from Account A add to bucket B (via bucket policy)?
- [ ] Account B
- [x] **Account A**
- [ ] It depends IAM user settings
- [ ] It depends on the bucket settings

### Q9: Who owns objects when assuming a role in Account B to add to bucket B?
- [x] **Account B**
- [ ] Account A
- [ ] It depends on the bucket settings
- [ ] It depends on sts settings

---

## Exam Tips & Traps

### Common Keyword Signals

| Keyword | Answer |
|---------|--------|
| "Immediately revoke all sessions" | Inline deny policy with `aws:TokenIssueTime` |
| "Share VPC across 50 accounts" | AWS RAM |
| "Prevent privilege escalation" | Permissions boundaries |
| "Enforce across organization" | SCPs at OU level |
| "Who owns objects" | Check if role assumption or bucket policy |

### Must-Remember Facts

- **One explicit deny beats any number of allows**
- **IAM groups cannot be principals in resource policies**
- **Management account is immune to SCPs**
- **RAM shares subnets, not entire VPCs**
- **STS doesn't have a RevokeToken API**

### Day Before Exam Checklist

- [ ] Can you explain policy evaluation in under 2 minutes?
- [ ] Do you know the difference between SCPs and permissions boundaries?
- [ ] Can you write a cross-account trust policy from memory?
- [ ] Do you understand object ownership in both bucket policy and role assumption scenarios?
- [ ] Can you list 5 resources shareable with RAM?

---

## Summary

| Topic | Key Point |
|-------|-----------|
| Organizations | Management account immune to SCPs |
| SCPs | Filter permissions, explicit deny wins |
| STS | Temporary credentials (15 min to 36 hours) |
| Revoking Credentials | Inline deny with `aws:TokenIssueTime` |
| Permissions Boundaries | Safe delegation, prevent privilege escalation |
| Cross-Account S3 | IAM Roles preferred, object ownership matters |
| AWS RAM | Shares subnets, not S3/KMS/Lambda |

---

**Good luck on your AWS Solutions Architect Professional exam!**

---

*Questions about Advanced Permissions & Accounts? Leave a comment below!*
