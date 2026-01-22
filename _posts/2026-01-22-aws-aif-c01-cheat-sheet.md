---
title: "AWS AIF-C01 - Quick Reference Cheat Sheet"
author: thanhnv1808
date: 2026-01-22 17:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, cheat-sheet, quick-reference, study-guide]
description: One-page cheat sheet for AWS AI Practitioner (AIF-C01) exam. Quick reference for all key concepts, services, and exam tips.
pin: false
comments: true
---

## Exam Quick Facts

| Item | Detail |
|------|--------|
| **Code** | AIF-C01 |
| **Questions** | 65 (50 scored) |
| **Time** | 90 minutes |
| **Pass** | 700/1000 |
| **Cost** | $100 USD |

---

## Domain Weights

```
Domain 3: Foundation Models    ████████████████████████████  28%
Domain 2: Generative AI        ████████████████████████      24%
Domain 1: AI/ML Fundamentals   ████████████████████          20%
Domain 4: Responsible AI       ██████████████                14%
Domain 5: Security/Compliance  ██████████████                14%
```

---

## AI Hierarchy

```
AI (Artificial Intelligence)
 └── ML (Machine Learning)
      └── Deep Learning
           └── Generative AI
```

---

## Learning Types

| Type | Data | Use Case |
|------|------|----------|
| **Supervised** | Labeled | Classification, Regression |
| **Unsupervised** | Unlabeled | Clustering, Anomaly detection |
| **Reinforcement** | Rewards | Games, Robotics |

---

## ML Techniques

| Technique | Output | Example |
|-----------|--------|---------|
| **Regression** | Continuous number | Price prediction |
| **Classification** | Category | Spam detection |
| **Clustering** | Groups | Customer segments |

---

## Data Types

| Structured | Unstructured |
|------------|--------------|
| Database tables | Images |
| CSV files | Videos |
| Spreadsheets | Audio |
| JSON/XML | Text documents |

---

## AWS AI Services Quick Reference

### GenAI Services

| Service | Purpose |
|---------|---------|
| **Bedrock** | Access Foundation Models |
| **Q Business** | Enterprise assistant |
| **Q Developer** | Code assistant |
| **PartyRock** | No-code GenAI builder |
| **SageMaker JumpStart** | Pre-trained models |

### NLP Services

| Service | Purpose |
|---------|---------|
| **Comprehend** | Text analysis (sentiment, entities) |
| **Translate** | Language translation |
| **Transcribe** | Speech-to-text |
| **Polly** | Text-to-speech |
| **Lex** | Chatbots |

### Vision Services

| Service | Purpose |
|---------|---------|
| **Rekognition** | Image/video analysis |
| **Textract** | Document OCR |

### ML Platform

| Service | Purpose |
|---------|---------|
| **SageMaker** | Complete ML platform |
| **Clarify** | Bias detection |
| **Model Monitor** | Production monitoring |
| **Model Cards** | Documentation |

---

## Amazon Bedrock Components

```
Bedrock
├── Model Access (Claude, Titan, Llama, etc.)
├── Knowledge Bases (RAG)
├── Agents (Multi-step tasks)
├── Guardrails (Content filtering)
└── Model Evaluation (Compare models)
```

---

## Temperature Parameter

| Value | Effect | Use For |
|-------|--------|---------|
| **0.0** | Deterministic | Facts, Code |
| **0.5** | Balanced | General use |
| **1.0** | Creative | Brainstorming |

**Lower = Consistent, Higher = Creative**

---

## Prompt Engineering Techniques

| Technique | Description |
|-----------|-------------|
| **Zero-shot** | No examples |
| **One-shot** | One example |
| **Few-shot** | Multiple examples (3-5) |
| **Chain-of-thought** | "Let's think step by step" |

---

## RAG vs Fine-tuning

| RAG | Fine-tuning |
|-----|-------------|
| Current information | Domain expertise |
| No retraining | Requires training data |
| Lower cost | Higher cost |
| Easy updates | Model changes |

**Use RAG for**: Current info, documents, FAQs
**Use Fine-tuning for**: Style, tone, domain language

---

## Evaluation Metrics

| Metric | Used For |
|--------|----------|
| **ROUGE** | Summarization |
| **BLEU** | Translation |
| **Accuracy** | Classification |
| **F1 Score** | Balanced precision/recall |
| **Recall** | When false negatives are costly |
| **Precision** | When false positives are costly |

---

## Bias & Fairness

### Bias Detection Tools

| Tool | Purpose |
|------|---------|
| **Clarify** | Detect bias in data/models |
| **Model Monitor** | Monitor bias drift |
| **A2I** | Human review |

### Model Complexity Tradeoff

```
Simple Models → High Transparency, Lower Performance
Complex Models → Low Transparency, Higher Performance
```

---

## Security Services

| Service | Purpose |
|---------|---------|
| **IAM** | Access control |
| **CloudTrail** | API logging |
| **Config** | Compliance rules |
| **Macie** | Sensitive data discovery |
| **PrivateLink** | Private connectivity |
| **Audit Manager** | Audit preparation |
| **Artifact** | Compliance reports |

---

## Shared Responsibility Model

```
AWS Responsible For:        Customer Responsible For:
├── Physical security       ├── Data
├── Hardware                ├── Access control
├── Networking              ├── IAM configuration
└── Service operation       └── Application security
```

---

## Encryption

| Type | Protects |
|------|----------|
| **At Rest** | Stored data |
| **In Transit** | Data being transferred |

---

## Key Formulas

### Overfitting vs Underfitting

| Problem | Cause | Sign |
|---------|-------|------|
| **Overfitting** | Too complex | Great training, poor test |
| **Underfitting** | Too simple | Poor training and test |

---

## Service Selection Cheat Sheet

### "I need to..."

| Need | Service |
|------|---------|
| Access Foundation Models | **Bedrock** |
| Build custom ML models | **SageMaker** |
| Analyze text sentiment | **Comprehend** |
| Convert speech to text | **Transcribe** |
| Convert text to speech | **Polly** |
| Translate languages | **Translate** |
| Build a chatbot | **Lex** |
| Analyze images | **Rekognition** |
| Extract text from documents | **Textract** |
| Detect bias | **Clarify** |
| Monitor production models | **Model Monitor** |
| Add human review | **A2I** |
| Filter AI content | **Bedrock Guardrails** |
| Log API calls | **CloudTrail** |
| Check compliance | **Config** |
| Find sensitive data | **Macie** |
| Prepare for audit | **Audit Manager** |
| Get compliance reports | **Artifact** |

---

## Responsible AI Features

**F**airness
**I**nclusivity
**R**obustness
**S**afety
**T**ransparency
**V**eracity
**P**rivacy

---

## GenAI Limitations

1. **Hallucinations** - False information
2. **Nondeterminism** - Varying outputs
3. **Bias** - Unfair outputs
4. **Knowledge cutoff** - Outdated info
5. **Context limits** - Token limits

---

## Inference Types

| Type | Speed | Use Case |
|------|-------|----------|
| **Real-time** | Fast | Chatbots, live apps |
| **Batch** | Slow | Reports, bulk processing |

---

## MLOps Concepts

- **Experimentation** - Track experiments
- **Reproducibility** - Repeat results
- **Automation** - CI/CD for ML
- **Monitoring** - Track performance
- **Model Registry** - Version models

---

## Quick Exam Tips

1. **No penalty for guessing** - Answer ALL questions
2. **Eliminate wrong answers first**
3. **Look for keywords** - "most", "best", "least"
4. **Bedrock = Foundation Models**
5. **SageMaker = Custom ML**
6. **Comprehend = NLP/Text**
7. **Rekognition = Images/Video**
8. **Lower temperature = More consistent**
9. **RAG = Current info without retraining**
10. **Clarify = Bias detection**

---

## Common Wrong Answer Traps

| Trap | Reality |
|------|---------|
| "AI can replace human judgment" | AI assists, doesn't replace |
| "More data always better" | Quality > Quantity |
| "Complex models are better" | Depends on use case |
| "GenAI outputs are always accurate" | Hallucinations exist |
| "Fine-tuning is always needed" | Often RAG/prompting suffices |

---

## Last Minute Review

### Must Know Services
1. Amazon Bedrock (and all components)
2. Amazon SageMaker (Clarify, Model Monitor, Model Cards)
3. Amazon Comprehend
4. Amazon Q (Business, Developer)
5. AWS Security services (CloudTrail, Config, IAM)

### Must Know Concepts
1. AI vs ML vs Deep Learning vs GenAI
2. Supervised vs Unsupervised vs Reinforcement
3. RAG architecture and benefits
4. Prompt engineering techniques
5. Responsible AI principles
6. Shared responsibility model

---

**Good Luck on Your Exam!**

---

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)
