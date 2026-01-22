---
title: "AWS AIF-C01 - Domain 5: Security, Compliance, and Governance"
author: thanhnv1808
date: 2026-01-22 13:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, security, compliance, governance, iam]
description: Domain 5 covers 14% of the AWS AI Practitioner exam. Learn about securing AI systems, compliance, and governance frameworks.
pin: false
comments: true
---

## Domain Overview

**Domain 5: Security, Compliance, and Governance for AI Solutions** represents **14%** of the exam (approximately 7 questions).

This domain covers two main task statements:
1. Explain methods to secure AI systems
2. Recognize governance and compliance regulations for AI systems

---

## Task 5.1: Methods to Secure AI Systems

### AWS Shared Responsibility Model for AI

Understanding who is responsible for what in AI workloads:

| Layer | AWS Responsibility | Customer Responsibility |
|-------|-------------------|------------------------|
| **Infrastructure** | Physical security, hardware | Configuration, access |
| **Services** | Service operation, updates | Data, access control |
| **AI Models** | Model hosting (managed) | Data privacy, model output |
| **Data** | Storage infrastructure | Data content, classification |
| **Applications** | N/A | Application security |

```
AWS Manages:          Customer Manages:
├── Physical security  ├── Data
├── Hardware          ├── Access control
├── Service updates   ├── Model outputs
└── Networking        └── Application logic
```

> **Exam Tip**: AWS secures the cloud infrastructure; customers secure their data and configurations in the cloud.
{: .prompt-tip }

### AWS Security Services for AI

#### IAM (Identity and Access Management)

| Component | Purpose | Use Case |
|-----------|---------|----------|
| **IAM Users** | Individual identities | Developer access |
| **IAM Roles** | Temporary credentials | Service-to-service |
| **IAM Policies** | Permission definitions | Access control |
| **IAM Groups** | Collection of users | Team management |

**Best Practices for AI Workloads:**
- Use least privilege principle
- Use roles instead of long-term credentials
- Enable MFA for sensitive operations
- Regularly audit permissions

#### Encryption

| Type | Description | AWS Service |
|------|-------------|-------------|
| **At Rest** | Data stored encrypted | S3 encryption, EBS encryption |
| **In Transit** | Data encrypted during transfer | TLS/SSL, VPC endpoints |
| **Client-side** | Encrypted before sending | AWS Encryption SDK |

> **Exam Tip**: Know the difference between encryption at rest (stored data) and in transit (data being transferred).
{: .prompt-warning }

#### Amazon Macie
| Feature | Description |
|---------|-------------|
| **Data Discovery** | Find sensitive data in S3 |
| **Classification** | Categorize data types |
| **PII Detection** | Identify personal information |
| **Alerts** | Notify on sensitive data |

#### AWS PrivateLink
| Feature | Description |
|---------|-------------|
| **Private Connectivity** | Access services without internet |
| **VPC Endpoints** | Connect to AWS services privately |
| **Security** | Keep traffic within AWS network |

### Security and Privacy Considerations for AI

#### Application Security

| Area | Considerations |
|------|---------------|
| **Input Validation** | Sanitize user inputs |
| **Output Filtering** | Filter harmful content |
| **Authentication** | Verify user identity |
| **Authorization** | Control access to functions |

#### AI-Specific Threats

| Threat | Description | Mitigation |
|--------|-------------|------------|
| **Prompt Injection** | Malicious prompts alter behavior | Input validation, guardrails |
| **Data Poisoning** | Corrupted training data | Data validation, monitoring |
| **Model Extraction** | Stealing model through queries | Rate limiting, access control |
| **Adversarial Attacks** | Inputs designed to fool model | Robust training, detection |

#### Infrastructure Protection

| Layer | Protection Methods |
|-------|-------------------|
| **Network** | VPCs, Security Groups, NACLs |
| **Compute** | Instance hardening, patching |
| **Data** | Encryption, access control |
| **Application** | WAF, Shield, security testing |

### Data Lineage and Documentation

#### Data Lineage
Track data from source to use:

```
Source → Collection → Processing → Storage → Model → Prediction
   ↓          ↓            ↓           ↓         ↓         ↓
 Origin    Method      Transform    Location   Training   Output
```

| Component | Description |
|-----------|-------------|
| **Source** | Where data originated |
| **Collection** | How data was gathered |
| **Processing** | Transformations applied |
| **Storage** | Where data is stored |
| **Usage** | How data is used |

#### Amazon SageMaker Model Cards
Document model provenance:

| Section | Content |
|---------|---------|
| **Model Overview** | Purpose and description |
| **Training Data** | Data sources, characteristics |
| **Evaluation Results** | Performance metrics |
| **Intended Use** | Appropriate applications |
| **Limitations** | Known constraints |

#### Data Cataloging
| Tool | Purpose |
|------|---------|
| **AWS Glue Data Catalog** | Metadata repository |
| **Amazon DataZone** | Data governance |
| **AWS Lake Formation** | Data lake management |

### Best Practices for Secure Data Engineering

| Practice | Description |
|----------|-------------|
| **Data Quality Assessment** | Validate data accuracy and completeness |
| **Privacy-Enhancing Technologies** | Anonymization, pseudonymization |
| **Access Control** | Role-based access to data |
| **Data Integrity** | Ensure data hasn't been tampered |
| **Audit Trails** | Log all data access |
| **Data Classification** | Label data by sensitivity |

---

## Task 5.2: Governance and Compliance

### AWS Governance and Compliance Services

#### AWS Config
| Feature | Description |
|---------|-------------|
| **Resource Tracking** | Monitor AWS resource configurations |
| **Compliance Rules** | Define and check compliance |
| **Change Tracking** | Record configuration changes |
| **Remediation** | Auto-fix non-compliant resources |

#### Amazon Inspector
| Feature | Description |
|---------|-------------|
| **Vulnerability Scanning** | Find security vulnerabilities |
| **Automated Assessment** | Continuous security checks |
| **Risk Prioritization** | Focus on critical issues |

#### AWS Audit Manager
| Feature | Description |
|---------|-------------|
| **Audit Preparation** | Collect audit evidence |
| **Compliance Mapping** | Map to frameworks |
| **Reporting** | Generate audit reports |
| **Custom Frameworks** | Build custom compliance checks |

#### AWS Artifact
| Feature | Description |
|---------|-------------|
| **Compliance Reports** | Access AWS compliance reports |
| **Agreements** | Review and accept agreements |
| **Certifications** | SOC, ISO, PCI-DSS documentation |

#### AWS CloudTrail
| Feature | Description |
|---------|-------------|
| **API Logging** | Record all API calls |
| **User Activity** | Track who did what |
| **Event History** | Historical audit trail |
| **Integration** | Works with SIEM tools |

#### AWS Trusted Advisor
| Feature | Description |
|---------|-------------|
| **Best Practices** | Check against AWS recommendations |
| **Security Checks** | Identify security gaps |
| **Cost Optimization** | Find cost savings |
| **Performance** | Improve performance |

> **Exam Tip**: Know which service to use: CloudTrail for API logging, Config for compliance, Audit Manager for audit reports.
{: .prompt-tip }

### Data Governance Strategies

#### Data Lifecycle Management

```
Creation → Storage → Usage → Archival → Deletion
    ↓          ↓        ↓         ↓          ↓
  Classify   Secure    Control   Retain    Purge
```

| Stage | Governance Activities |
|-------|----------------------|
| **Creation** | Classification, metadata |
| **Storage** | Encryption, access control |
| **Usage** | Logging, monitoring |
| **Archival** | Retention policies |
| **Deletion** | Secure destruction |

#### Key Governance Concepts

| Concept | Description |
|---------|-------------|
| **Data Residency** | Where data must be stored geographically |
| **Logging** | Record data access and changes |
| **Monitoring** | Real-time observation of data systems |
| **Retention** | How long to keep data |
| **Observation** | Visibility into data usage |

### Governance Protocol Processes

#### Policy Components

| Component | Description |
|-----------|-------------|
| **Policies** | Rules for AI system use |
| **Review Cadence** | How often to review/update |
| **Review Strategies** | Approaches to assessment |
| **Frameworks** | Structured governance approaches |
| **Transparency Standards** | Disclosure requirements |
| **Training Requirements** | Team education needs |

#### Generative AI Security Scoping Matrix

Framework for assessing GenAI risks:

| Dimension | Assessment Areas |
|-----------|-----------------|
| **Data Sensitivity** | What data does the model access? |
| **Use Case Risk** | What could go wrong? |
| **User Access** | Who can use the system? |
| **Output Impact** | What are the consequences? |
| **Compliance Requirements** | What regulations apply? |

### Compliance Considerations for AI

#### Common Compliance Frameworks

| Framework | Focus | Relevance to AI |
|-----------|-------|-----------------|
| **GDPR** | Data privacy (EU) | Personal data in AI |
| **HIPAA** | Healthcare (US) | Medical AI applications |
| **SOC 2** | Security controls | AI service security |
| **ISO 27001** | Information security | AI system security |
| **PCI-DSS** | Payment data | Financial AI |

#### AI-Specific Compliance

| Consideration | Description |
|---------------|-------------|
| **AI Transparency** | Disclose AI use to users |
| **Algorithmic Accountability** | Explain AI decisions |
| **Data Rights** | User control over their data |
| **Model Governance** | Track model versions and changes |

---

## Security Best Practices Summary

### Defense in Depth for AI

```
┌─────────────────────────────────────────┐
│           Application Security           │
│  ┌─────────────────────────────────┐    │
│  │       Data Security              │    │
│  │  ┌─────────────────────────┐    │    │
│  │  │    Model Security       │    │    │
│  │  │  ┌─────────────────┐    │    │    │
│  │  │  │ Infrastructure  │    │    │    │
│  │  │  └─────────────────┘    │    │    │
│  │  └─────────────────────────┘    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

| Layer | Security Measures |
|-------|------------------|
| **Infrastructure** | VPCs, encryption, IAM |
| **Model** | Access control, monitoring |
| **Data** | Classification, encryption |
| **Application** | Input validation, output filtering |

---

## Practice Questions

### Question 1
Under the AWS Shared Responsibility Model, who is responsible for securing the data used to train AI models?

A. AWS
B. Customer
C. Both AWS and Customer equally
D. Neither - it depends on the service

<details>
<summary>Show Answer</summary>
<b>B. Customer</b>

Customers are responsible for their data, including training data, while AWS is responsible for securing the underlying infrastructure.
</details>

### Question 2
Which AWS service would you use to automatically log all API calls made in your AWS account?

A. AWS Config
B. AWS CloudTrail
C. Amazon Inspector
D. AWS Audit Manager

<details>
<summary>Show Answer</summary>
<b>B. AWS CloudTrail</b>

CloudTrail records API calls made in your AWS account, providing an audit trail of all activities.
</details>

### Question 3
A company needs to ensure their AI system can only be accessed through private network connections without going over the internet. Which AWS feature should they use?

A. Amazon Macie
B. AWS PrivateLink
C. AWS Config
D. Amazon Inspector

<details>
<summary>Show Answer</summary>
<b>B. AWS PrivateLink</b>

AWS PrivateLink enables private connectivity to AWS services without using the public internet.
</details>

### Question 4
Which AWS service helps discover and protect sensitive data stored in Amazon S3?

A. AWS Config
B. Amazon Macie
C. AWS CloudTrail
D. AWS Trusted Advisor

<details>
<summary>Show Answer</summary>
<b>B. Amazon Macie</b>

Amazon Macie uses machine learning to discover, classify, and protect sensitive data in S3.
</details>

### Question 5
An organization needs to prepare for a compliance audit of their AI systems. Which AWS service would help collect and organize audit evidence?

A. AWS Config
B. AWS CloudTrail
C. AWS Audit Manager
D. AWS Artifact

<details>
<summary>Show Answer</summary>
<b>C. AWS Audit Manager</b>

AWS Audit Manager helps collect audit evidence and map to compliance frameworks for audit preparation.
</details>

### Question 6
What is the primary purpose of data lineage in AI governance?

A. To encrypt data at rest
B. To track data from source through all transformations to usage
C. To automatically delete old data
D. To compress data for storage

<details>
<summary>Show Answer</summary>
<b>B. To track data from source through all transformations to usage</b>

Data lineage documents the origin of data and how it has been transformed and used throughout the AI pipeline.
</details>

---

## Key Takeaways

1. **Shared Responsibility**: AWS secures infrastructure; customers secure data and configurations
2. **Encryption**: At rest (stored) and in transit (moving)
3. **IAM**: Use least privilege, roles over users, enable MFA
4. **Key Services**:
   - CloudTrail: API logging
   - Config: Compliance rules
   - Audit Manager: Audit preparation
   - Macie: Sensitive data discovery
   - PrivateLink: Private connectivity
5. **Data Governance**: Lifecycle, residency, retention, logging
6. **AI Security Threats**: Prompt injection, data poisoning, model extraction

### Service Selection Guide

```
Log API calls?                    → CloudTrail
Check compliance rules?           → AWS Config
Prepare for audit?                → Audit Manager
Find sensitive data in S3?        → Amazon Macie
Private network access?           → AWS PrivateLink
Get compliance reports?           → AWS Artifact
Security vulnerability scan?      → Amazon Inspector
Best practices recommendations?   → Trusted Advisor
```

---

**Previous Lesson**: [Domain 4: Guidelines for Responsible AI](/posts/aws-aif-c01-domain-4-responsible-ai/)

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)

---

## Final Exam Preparation Tips

### Study Priority by Domain Weight

| Priority | Domain | Weight |
|----------|--------|--------|
| 1 | Domain 3: Applications of Foundation Models | 28% |
| 2 | Domain 2: Fundamentals of GenAI | 24% |
| 3 | Domain 1: Fundamentals of AI and ML | 20% |
| 4 | Domain 4: Responsible AI | 14% |
| 5 | Domain 5: Security, Compliance, Governance | 14% |

### Key Concepts to Review

1. **AI/ML Basics**: Supervised, unsupervised, reinforcement learning
2. **GenAI**: Tokens, embeddings, LLMs, Foundation Models
3. **RAG**: When and how to use it
4. **Prompt Engineering**: Zero-shot, few-shot, chain-of-thought
5. **Amazon Bedrock**: Features, agents, knowledge bases
6. **Responsible AI**: Bias, fairness, explainability
7. **Security**: IAM, encryption, shared responsibility

### Exam Day Tips

- Read questions carefully - look for keywords
- Eliminate obviously wrong answers
- Don't spend too much time on one question
- Answer ALL questions (no penalty for guessing)
- Review flagged questions if time permits

Good luck on your AWS AI Practitioner exam!

---

*Questions about Security and Compliance? Leave a comment below!*
