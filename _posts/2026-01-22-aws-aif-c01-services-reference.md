---
title: "AWS AIF-C01 - Complete AWS AI/ML Services Reference"
author: thanhnv1808
date: 2026-01-22 15:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, bedrock, sagemaker, comprehend, services-reference]
description: Complete reference guide for all AWS AI/ML services covered in the AIF-C01 exam. Detailed features, use cases, and comparisons.
pin: false
comments: true
---

## Introduction

This comprehensive reference covers all AWS AI/ML services you need to know for the AIF-C01 exam. Use this as your go-to guide for understanding each service's purpose, features, and use cases.

---

## Amazon Bedrock

### Overview
**Amazon Bedrock** is a fully managed service for accessing Foundation Models (FMs) through an API.

### Key Features

| Feature | Description |
|---------|-------------|
| **Model Access** | Access multiple FMs from one API |
| **Serverless** | No infrastructure management |
| **Private** | Your data stays private, not used for training |
| **Customization** | Fine-tune models with your data |

### Available Models

| Provider | Models | Specialization |
|----------|--------|----------------|
| **Anthropic** | Claude 3 (Opus, Sonnet, Haiku) | General purpose, reasoning |
| **Amazon** | Titan (Text, Embeddings, Image) | General purpose, embeddings |
| **Meta** | Llama 2, Llama 3 | Open-source, general purpose |
| **Mistral** | Mistral, Mixtral | Efficient, multilingual |
| **Cohere** | Command, Embed | Enterprise, embeddings |
| **AI21 Labs** | Jurassic | Text generation |
| **Stability AI** | Stable Diffusion | Image generation |

### Bedrock Components

#### Amazon Bedrock Knowledge Bases
| Feature | Description |
|---------|-------------|
| **Purpose** | Implement RAG (Retrieval Augmented Generation) |
| **Data Sources** | S3, web crawlers, Confluence, SharePoint |
| **Vector Stores** | OpenSearch, Aurora, Neptune, Pinecone |
| **Benefits** | Current info, reduced hallucinations, source citations |

#### Amazon Bedrock Agents
| Feature | Description |
|---------|-------------|
| **Purpose** | Autonomous task execution |
| **Capabilities** | Multi-step reasoning, tool use, API calls |
| **Components** | Instructions, action groups, knowledge bases |
| **Use Cases** | Customer service, task automation |

#### Amazon Bedrock Guardrails
| Feature | Description |
|---------|-------------|
| **Content Filters** | Block harmful content (hate, violence, etc.) |
| **Topic Filters** | Deny specific topics |
| **Word Filters** | Block specific words/phrases |
| **PII Filters** | Detect and mask personal information |
| **Contextual Grounding** | Check response accuracy against sources |

#### Amazon Bedrock Model Evaluation
| Feature | Description |
|---------|-------------|
| **Automatic Evaluation** | Built-in metrics (ROUGE, BLEU, etc.) |
| **Human Evaluation** | Custom human review workflows |
| **Comparison** | Compare multiple models side-by-side |

### Bedrock Pricing Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **On-Demand** | Pay per token | Variable workloads |
| **Provisioned Throughput** | Reserved capacity | Consistent workloads |
| **Batch Inference** | Lower cost for large batches | Non-real-time |
| **Model Customization** | Training + hosting costs | Fine-tuning |

> **Exam Tip**: Bedrock is the primary service for accessing Foundation Models. Know its components well!
{: .prompt-tip }

---

## Amazon SageMaker

### Overview
**Amazon SageMaker** is a fully managed machine learning platform for building, training, and deploying ML models.

### Key Components

#### SageMaker Studio
| Feature | Description |
|---------|-------------|
| **IDE** | Integrated development environment for ML |
| **Notebooks** | Jupyter notebooks for experimentation |
| **Collaboration** | Share notebooks and experiments |

#### SageMaker JumpStart
| Feature | Description |
|---------|-------------|
| **Model Hub** | Pre-trained models ready to deploy |
| **Foundation Models** | Access to LLMs and other FMs |
| **Solutions** | End-to-end ML solutions |
| **Fine-tuning** | Customize models with your data |

#### SageMaker Data Wrangler
| Feature | Description |
|---------|-------------|
| **Data Prep** | Visual data preparation |
| **Transformations** | 300+ built-in transformations |
| **Data Quality** | Analyze data quality |
| **Export** | Export to training pipelines |

#### SageMaker Feature Store
| Feature | Description |
|---------|-------------|
| **Feature Repository** | Store and share ML features |
| **Online Store** | Low-latency feature retrieval |
| **Offline Store** | Batch feature access for training |
| **Feature Groups** | Organize related features |

#### SageMaker Clarify
| Feature | Description |
|---------|-------------|
| **Bias Detection** | Detect bias in data and models |
| **Explainability** | Explain model predictions |
| **Metrics** | Pre-training and post-training bias metrics |
| **Reports** | Generate bias and explainability reports |

**Clarify Bias Metrics:**

| Metric | Description |
|--------|-------------|
| **Class Imbalance (CI)** | Difference in class representation |
| **DPL** | Difference in Positive Labels |
| **KL Divergence** | Distribution difference |
| **SHAP Values** | Feature importance |

#### SageMaker Model Monitor
| Feature | Description |
|---------|-------------|
| **Data Quality** | Monitor input data drift |
| **Model Quality** | Track prediction accuracy |
| **Bias Drift** | Detect changes in bias over time |
| **Feature Attribution** | Monitor feature importance |

#### SageMaker Model Cards
| Feature | Description |
|---------|-------------|
| **Documentation** | Standardized model documentation |
| **Intended Use** | Document appropriate use cases |
| **Limitations** | Record known limitations |
| **Metrics** | Store evaluation metrics |
| **Training Details** | Document training information |

#### SageMaker Training
| Feature | Description |
|---------|-------------|
| **Managed Training** | Automatic infrastructure provisioning |
| **Distributed Training** | Scale across multiple instances |
| **Spot Training** | Cost savings with spot instances |
| **Hyperparameter Tuning** | Automatic optimization |

#### SageMaker Endpoints
| Feature | Description |
|---------|-------------|
| **Real-time** | Low-latency predictions |
| **Serverless** | Auto-scaling, pay-per-use |
| **Asynchronous** | Long-running predictions |
| **Batch Transform** | Large-scale batch inference |

> **Exam Tip**: SageMaker is for custom ML development. Know Clarify (bias), Model Monitor (monitoring), and Model Cards (documentation).
{: .prompt-warning }

---

## Amazon Q

### Overview
**Amazon Q** is an AI-powered assistant for business and development tasks.

### Amazon Q Variants

| Variant | Purpose | Users |
|---------|---------|-------|
| **Amazon Q Business** | Enterprise knowledge assistant | Business users |
| **Amazon Q Developer** | Code assistance | Developers |
| **Amazon Q in QuickSight** | BI assistance | Analysts |
| **Amazon Q in Connect** | Contact center assistance | Agents |
| **Amazon Q in AWS Console** | AWS assistance | AWS users |

### Amazon Q Business
| Feature | Description |
|---------|-------------|
| **Data Connectors** | Connect to enterprise data sources |
| **Knowledge Search** | Search across connected sources |
| **Summarization** | Summarize documents and conversations |
| **Plugins** | Integrate with enterprise apps |
| **Admin Controls** | Manage access and permissions |

### Amazon Q Developer
| Feature | Description |
|---------|-------------|
| **Code Generation** | Generate code from comments |
| **Code Completion** | Real-time code suggestions |
| **Code Explanation** | Explain code functionality |
| **Security Scanning** | Find vulnerabilities |
| **Code Transformation** | Modernize legacy code |

---

## Amazon Comprehend

### Overview
**Amazon Comprehend** is a natural language processing (NLP) service using machine learning.

### Features

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Sentiment Analysis** | Detect positive/negative/neutral/mixed | Customer feedback |
| **Entity Recognition** | Extract people, places, organizations | Information extraction |
| **Key Phrase Extraction** | Identify important phrases | Document summarization |
| **Language Detection** | Identify text language | Multi-language support |
| **Syntax Analysis** | Parse parts of speech | Text analysis |
| **Topic Modeling** | Discover topics in documents | Content organization |
| **Custom Classification** | Train custom classifiers | Domain-specific needs |
| **Custom Entity Recognition** | Train custom entity extractors | Specialized entities |

### Comprehend Medical
| Feature | Description |
|---------|-------------|
| **Medical Entity Extraction** | Extract medical terms |
| **PHI Detection** | Detect protected health information |
| **ICD-10 Linking** | Link to medical codes |
| **RxNorm Linking** | Link to medication codes |

> **Exam Tip**: Comprehend = NLP (text analysis). Know sentiment, entities, key phrases, and language detection.
{: .prompt-tip }

---

## Amazon Transcribe

### Overview
**Amazon Transcribe** is an automatic speech recognition (ASR) service.

### Features

| Feature | Description |
|---------|-------------|
| **Speech-to-Text** | Convert audio to text |
| **Real-time** | Live transcription |
| **Batch** | Process audio files |
| **Speaker Identification** | Identify different speakers |
| **Custom Vocabulary** | Add domain-specific terms |
| **Language Identification** | Auto-detect language |
| **Redaction** | Remove sensitive information |
| **Subtitles** | Generate subtitle files |

### Transcribe Variants

| Variant | Use Case |
|---------|----------|
| **Transcribe** | General transcription |
| **Transcribe Medical** | Medical conversations |
| **Transcribe Call Analytics** | Contact center analytics |

---

## Amazon Translate

### Overview
**Amazon Translate** is a neural machine translation service.

### Features

| Feature | Description |
|---------|-------------|
| **Real-time Translation** | Instant text translation |
| **Batch Translation** | Translate document batches |
| **Custom Terminology** | Use your specific terms |
| **Profanity Masking** | Filter inappropriate content |
| **Formality Control** | Formal/informal output |
| **Brevity** | Concise translations |
| **75+ Languages** | Wide language support |

---

## Amazon Polly

### Overview
**Amazon Polly** is a text-to-speech (TTS) service.

### Features

| Feature | Description |
|---------|-------------|
| **Neural TTS** | Natural-sounding voices |
| **Standard TTS** | Basic voices |
| **SSML Support** | Control pronunciation, pauses |
| **Lexicons** | Custom pronunciation |
| **Speech Marks** | Timing information |
| **Multiple Formats** | MP3, OGG, PCM |

### Voice Types

| Type | Quality | Cost |
|------|---------|------|
| **Neural** | Most natural | Higher |
| **Long-Form** | Optimized for long content | Medium |
| **Standard** | Basic quality | Lower |

---

## Amazon Rekognition

### Overview
**Amazon Rekognition** is an image and video analysis service.

### Features

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Object Detection** | Identify objects in images | Content categorization |
| **Scene Detection** | Identify scene types | Media organization |
| **Face Detection** | Detect faces | Photo management |
| **Face Analysis** | Age, gender, emotions | User experience |
| **Face Comparison** | Compare faces | Identity verification |
| **Celebrity Recognition** | Identify famous people | Media tagging |
| **Text Detection (OCR)** | Extract text from images | Document processing |
| **Content Moderation** | Detect inappropriate content | Content safety |
| **Custom Labels** | Train custom detectors | Specialized detection |
| **Video Analysis** | Analyze video content | Media processing |

### Rekognition Use Cases

| Use Case | Features Used |
|----------|--------------|
| **Identity Verification** | Face comparison, liveness detection |
| **Content Moderation** | Unsafe content detection |
| **Media Analysis** | Object, scene, celebrity detection |
| **Security** | Face detection, person tracking |

---

## Amazon Lex

### Overview
**Amazon Lex** is a service for building conversational interfaces (chatbots).

### Features

| Feature | Description |
|---------|-------------|
| **Natural Language Understanding** | Understand user intent |
| **Automatic Speech Recognition** | Voice input |
| **Multi-turn Conversations** | Context maintenance |
| **Slot Filling** | Collect required information |
| **Integration** | Connect to Lambda, other services |
| **Multi-channel** | Deploy to multiple platforms |

### Lex Components

| Component | Description |
|-----------|-------------|
| **Bot** | The conversational application |
| **Intent** | User's goal or action |
| **Slot** | Information to collect |
| **Utterance** | Sample phrases |
| **Fulfillment** | Action to take |

---

## Amazon Textract

### Overview
**Amazon Textract** extracts text, forms, and tables from documents.

### Features

| Feature | Description |
|---------|-------------|
| **Text Detection** | Extract printed/handwritten text |
| **Form Extraction** | Extract key-value pairs |
| **Table Extraction** | Extract tabular data |
| **Query-based Extraction** | Answer specific questions |
| **Expense Analysis** | Process receipts/invoices |
| **ID Document Analysis** | Extract ID information |
| **Lending Document Analysis** | Process loan documents |

---

## Amazon Personalize

### Overview
**Amazon Personalize** creates personalized recommendations.

### Features

| Feature | Description |
|---------|-------------|
| **User Personalization** | Recommend items for users |
| **Similar Items** | Find related items |
| **Personalized Ranking** | Rank items for users |
| **Real-time** | Update recommendations instantly |
| **Batch** | Generate bulk recommendations |

### Use Cases
- E-commerce product recommendations
- Content recommendations (videos, articles)
- Personalized search results
- Targeted marketing

---

## Amazon Forecast

### Overview
**Amazon Forecast** creates time-series forecasts.

### Features

| Feature | Description |
|---------|-------------|
| **AutoML** | Automatic algorithm selection |
| **Multiple Algorithms** | DeepAR+, Prophet, ARIMA, etc. |
| **Related Time Series** | Include external factors |
| **Weather Integration** | Include weather data |
| **What-if Analysis** | Scenario planning |

### Use Cases
- Demand forecasting
- Inventory planning
- Resource planning
- Financial forecasting

---

## Amazon Fraud Detector

### Overview
**Amazon Fraud Detector** identifies potentially fraudulent activities.

### Features

| Feature | Description |
|---------|-------------|
| **ML Models** | Pre-trained fraud detection |
| **Rules Engine** | Custom business rules |
| **Real-time** | Instant fraud detection |
| **Event Types** | Account registration, transactions, etc. |

---

## Amazon Kendra

### Overview
**Amazon Kendra** is an intelligent enterprise search service.

### Features

| Feature | Description |
|---------|-------------|
| **Natural Language** | Search with questions |
| **Document Understanding** | Understand document context |
| **Connectors** | Connect to data sources |
| **Relevance Tuning** | Customize search results |
| **FAQ Extraction** | Automatic FAQ answers |

---

## Amazon Augmented AI (A2I)

### Overview
**Amazon A2I** adds human review to ML predictions.

### Features

| Feature | Description |
|---------|-------------|
| **Human Review** | Add human oversight |
| **Workflows** | Define review conditions |
| **Workforce** | Use internal or external reviewers |
| **Integration** | Works with Rekognition, Textract, SageMaker |

### Use Cases
- Content moderation review
- Document processing verification
- Medical diagnosis review
- Any high-stakes ML predictions

---

## PartyRock

### Overview
**PartyRock** is a no-code GenAI application builder.

### Features

| Feature | Description |
|---------|-------------|
| **No-Code** | Build apps without coding |
| **Templates** | Start from examples |
| **Widgets** | Text, image, user input |
| **Sharing** | Share apps publicly |
| **Free** | No AWS account needed |

### Use Cases
- Learning GenAI concepts
- Prototyping ideas
- Simple GenAI applications
- Educational purposes

---

## AWS Security Services for AI

### AWS IAM
| Feature | Purpose for AI |
|---------|----------------|
| **Roles** | Service-to-service access |
| **Policies** | Fine-grained permissions |
| **MFA** | Enhanced security |

### Amazon Macie
| Feature | Purpose |
|---------|---------|
| **Data Discovery** | Find sensitive data in S3 |
| **PII Detection** | Identify personal information |
| **Alerts** | Notify on sensitive data |

### AWS PrivateLink
| Feature | Purpose |
|---------|---------|
| **VPC Endpoints** | Private access to services |
| **No Internet** | Keep traffic private |

---

## AWS Governance Services for AI

### AWS CloudTrail
| Feature | Purpose |
|---------|---------|
| **API Logging** | Record all API calls |
| **Audit Trail** | Who did what, when |

### AWS Config
| Feature | Purpose |
|---------|---------|
| **Compliance Rules** | Check resource compliance |
| **Change Tracking** | Monitor configuration changes |

### AWS Audit Manager
| Feature | Purpose |
|---------|---------|
| **Audit Evidence** | Collect compliance evidence |
| **Frameworks** | Map to compliance standards |

---

## Service Comparison Tables

### When to Use Which Service

| Need | Service |
|------|---------|
| Access Foundation Models | Amazon Bedrock |
| Build custom ML models | Amazon SageMaker |
| Code assistance | Amazon Q Developer |
| Text analysis (NLP) | Amazon Comprehend |
| Speech-to-text | Amazon Transcribe |
| Text-to-speech | Amazon Polly |
| Translation | Amazon Translate |
| Image analysis | Amazon Rekognition |
| Chatbots | Amazon Lex |
| Document extraction | Amazon Textract |
| Recommendations | Amazon Personalize |
| Forecasting | Amazon Forecast |
| Fraud detection | Amazon Fraud Detector |
| Enterprise search | Amazon Kendra |
| Human review | Amazon A2I |
| No-code GenAI | PartyRock |

### GenAI Service Selection

| Scenario | Service |
|----------|---------|
| Production GenAI apps | Amazon Bedrock |
| Business knowledge assistant | Amazon Q Business |
| Developer assistance | Amazon Q Developer |
| Learning/prototyping | PartyRock |
| Custom FM deployment | SageMaker JumpStart |

### NLP Service Selection

| Task | Service |
|------|---------|
| Sentiment analysis | Comprehend |
| Entity extraction | Comprehend |
| Text generation | Bedrock |
| Translation | Translate |
| Speech-to-text | Transcribe |
| Text-to-speech | Polly |
| Document OCR | Textract |
| Chatbot | Lex |

---

## Quick Reference Cards

### Amazon Bedrock Quick Reference

```
Bedrock = Foundation Models as a Service

Components:
├── Model Access (Claude, Titan, Llama, etc.)
├── Knowledge Bases (RAG)
├── Agents (Autonomous tasks)
├── Guardrails (Safety)
└── Model Evaluation (Compare models)

Pricing:
├── On-Demand (per token)
├── Provisioned Throughput (reserved)
└── Batch (lower cost)
```

### Amazon SageMaker Quick Reference

```
SageMaker = Complete ML Platform

Components:
├── Studio (IDE)
├── JumpStart (Pre-trained models)
├── Data Wrangler (Data prep)
├── Feature Store (Feature management)
├── Training (Model training)
├── Clarify (Bias & explainability)
├── Model Monitor (Production monitoring)
├── Model Cards (Documentation)
└── Endpoints (Deployment)
```

---

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)

---

*Questions about AWS AI services? Leave a comment below!*
