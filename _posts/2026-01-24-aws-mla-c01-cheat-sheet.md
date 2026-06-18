---
title: "AWS MLA-C01 - Quick Reference Cheat Sheet"
author: thanhnv1808
date: 2026-01-24 17:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, cheat-sheet, quick-reference, study-guide]
description: Comprehensive cheat sheet for AWS Machine Learning Engineer Associate (MLA-C01) exam. Quick reference for all key concepts, services, and exam tips.
pin: false
comments: true
---

## Exam Quick Facts

| Item | Detail |
|------|--------|
| **Code** | MLA-C01 |
| **Questions** | 65 (85 scored, 15 unscored) |
| **Time** | 170 minutes (2h 50min) |
| **Pass** | 720/1000 |
| **Cost** | $75 USD |
| **Validity** | 3 years |

---

## Domain Weights

```
Domain 1: Data Preparation         ████████████████████████████  28%
Domain 2: Model Development         ██████████████████████████    26%
Domain 4: Monitoring/Security       ████████████████████          24%
Domain 3: Deployment/Orchestration  ██████████████████████        22%
```

---

## Data Storage Services

| Service | Type | Use Case | Cost |
|---------|------|----------|------|
| **S3** | Object storage | Data lake, training data | Low |
| **S3 Glacier** | Archive | Long-term retention | Very low |
| **Redshift** | Data warehouse | Analytics, aggregated features | Medium |
| **DynamoDB** | NoSQL | Real-time feature serving | Medium |
| **RDS/Aurora** | Relational | Transactional data | Medium |
| **ElastiCache** | In-memory | Low-latency features | High |
| **FSx Lustre** | File system | High-throughput training | High |
| **Feature Store** | Feature repository | Online + offline features | Medium |

---

## Data Formats

| Format | Type | Best For | Compression |
|--------|------|----------|-------------|
| **Parquet** | Columnar | Large ML datasets | Excellent |
| **CSV** | Row | Small datasets, exchange | None |
| **JSON** | Semi-structured | API data, logs | Poor |
| **Avro** | Row | Streaming events | Good |
| **RecordIO** | Binary | SageMaker training | Good |

**Exam Tip**: Parquet is preferred for large-scale ML workloads
{: .prompt-tip }

---

## Data Ingestion Services

| Service | Type | Scaling | Use Case |
|---------|------|---------|----------|
| **Kinesis Data Streams** | Real-time | Manual | Custom consumers |
| **Kinesis Firehose** | Near real-time | Auto | Simple delivery to S3/Redshift |
| **Glue ETL** | Batch | Auto | Large transformations |
| **EMR** | Batch | Manual | Big data processing |
| **Database Migration Service** | Batch | Auto | Database migration |

---

## Feature Engineering

### Encoding Methods

| Method | Cardinality | Output | Best For |
|--------|-------------|--------|----------|
| **One-Hot** | Low (<10) | Binary columns | Nominal categories |
| **Label** | Any | Integers | Ordinal categories |
| **Target** | High (>50) | Float | Tree models, high cardinality |
| **Frequency** | High | Float/Int | Rare categories |
| **Embedding** | Very high | Dense vector | Deep learning, NLP |

### Scaling Methods

| Method | Formula | Use Case |
|--------|---------|----------|
| **Min-Max** | (x-min)/(max-min) | Neural networks, [0,1] range |
| **Standard** | (x-μ)/σ | Linear models, SVM |
| **Robust** | (x-median)/IQR | Data with outliers |
| **Log** | log(x+1) | Right-skewed distributions |

---

## AWS Glue Components

```
Glue Crawlers → Glue Data Catalog → Glue ETL Jobs
     ↓              ↓                    ↓
  Discover       Store              Transform
  Schema        Metadata              Data
```

**Job Types:**
- **Spark ETL**: Large-scale transformations
- **Python Shell**: Light transformations, APIs
- **Streaming**: Real-time processing
- **Ray**: ML workloads

---

## SageMaker Feature Store

| Component | Description |
|-----------|-------------|
| **Online Store** | DynamoDB-based, <10ms latency |
| **Offline Store** | S3-based, for training |
| **Feature Group** | Collection of features |
| **Record Identifier** | Unique key |
| **Event Time** | For point-in-time queries |

**Key Feature**: Automatic sync between online and offline stores
{: .prompt-tip }

---

## SageMaker Built-in Algorithms

### Tabular Data

| Algorithm | Type | GPU | Use Case |
|-----------|------|-----|----------|
| **XGBoost** | Tree-based | Optional | Best for tabular data |
| **Linear Learner** | Linear | Yes | Large-scale, linear relationships |
| **k-NN** | Instance-based | Yes | No training phase |
| **Factorization Machines** | Matrix | Yes | Sparse, high-dimensional |

### Computer Vision

| Algorithm | Task | Transfer Learning |
|-----------|------|-------------------|
| **Image Classification** | Categorization | ResNet CNN |
| **Object Detection** | Localization | SSD, ResNet |
| **Semantic Segmentation** | Pixel-level | FCN, PSPNet |

### NLP

| Algorithm | Task | Output |
|-----------|------|--------|
| **BlazingText** | Classification, embeddings | Categories or vectors |
| **Seq2Seq** | Translation, summarization | Text sequences |
| **LDA** | Topic modeling | Topic distributions |

### Time Series

| Algorithm | Type | Features |
|-----------|------|----------|
| **DeepAR** | RNN-based | Probabilistic forecasts |

### Anomaly Detection

| Algorithm | Method | Use Case |
|-----------|--------|----------|
| **Random Cut Forest** | Unsupervised | Streaming anomalies |

---

## Hyperparameter Tuning

### Strategies

| Strategy | Method | Efficiency |
|----------|--------|-----------|
| **Bayesian** | Intelligent sampling | High (default) |
| **Random** | Random sampling | Medium |
| **Hyperband** | Early stopping | High |

### Key Concepts

```python
max_jobs = 20           # Total training jobs
max_parallel_jobs = 3   # Concurrent jobs
early_stopping = 'Auto' # Stop poor performers
```

**Warm Start**: Use previous tuning results to initialize new jobs

---

## Training Configuration

### Instance Types

| Family | Purpose | Cost | Example |
|--------|---------|------|---------|
| **ml.m5** | General purpose | Low | ml.m5.xlarge |
| **ml.c5** | Compute optimized | Low-Med | ml.c5.2xlarge |
| **ml.p3** | GPU (V100) | High | ml.p3.2xlarge |
| **ml.p4** | GPU (A100) | Very high | ml.p4d.24xlarge |
| **ml.g4dn** | GPU (T4, cost-effective) | Medium | ml.g4dn.xlarge |
| **ml.inf1** | AWS Inferentia | Low | ml.inf1.xlarge |

### Data Input Modes

| Mode | Method | Best For |
|------|--------|----------|
| **File** | Download all data | Small/medium datasets |
| **Pipe** | Stream from S3 | Large datasets, faster startup |
| **Fast File** | Lazy loading | Large datasets, random access |

### Distributed Training

```
Data Parallelism:
├── Use when: Large datasets, model fits in memory
├── Method: Replicate model on each instance
└── SageMaker: Data Parallel library

Model Parallelism:
├── Use when: Large model doesn't fit in GPU memory
├── Method: Split model across instances
└── SageMaker: Model Parallel library
```

---

## Evaluation Metrics

### Classification

| Metric | Formula | Use When |
|--------|---------|----------|
| **Accuracy** | (TP+TN)/(TP+TN+FP+FN) | Balanced datasets |
| **Precision** | TP/(TP+FP) | False positives costly |
| **Recall** | TP/(TP+FN) | False negatives costly |
| **F1 Score** | 2×(P×R)/(P+R) | Balance P and R |
| **AUC-ROC** | Area under curve | Binary classification |

### Regression

| Metric | Description |
|--------|-------------|
| **MSE** | Mean squared error |
| **RMSE** | Root mean squared error |
| **MAE** | Mean absolute error |
| **R²** | Coefficient of determination |

---

## Model Deployment Options

| Option | Latency | Cost | Use Case |
|--------|---------|------|----------|
| **Real-time Endpoint** | ms | High (always on) | Low latency apps |
| **Serverless** | sub-second | Low (pay-per-use) | Intermittent traffic |
| **Batch Transform** | min-hours | Low | Large batch jobs |
| **Async Inference** | sec-min | Medium | Large payloads (up to 1GB) |
| **Multi-Model** | ms | Very low | Many similar models |
| **Edge** | ms | Device-dependent | Offline, IoT |

### Decision Tree

```
Need real-time with consistent traffic? → Real-time Endpoint
Need real-time with intermittent traffic? → Serverless
Need to process millions of records periodically? → Batch Transform
Need to process large payloads (>6MB)? → Async Inference
Need to host hundreds of models? → Multi-Model Endpoint
Need offline inference on devices? → Edge Deployment
```

---

## Deployment Strategies

| Strategy | Method | Downtime | Rollback | Risk |
|----------|--------|----------|----------|------|
| **All-at-once** | Replace all | Yes | Hard | High |
| **Blue/Green** | Switch 100% | No | Easy | Low |
| **Canary** | Gradual % | No | Easy | Low |
| **A/B Testing** | Split traffic | No | Easy | Low |

### Canary Deployment Pattern

```
Start: 5% new, 95% old
  ↓ Monitor metrics
  ↓ If good
Increase: 25% new, 75% old
  ↓ Monitor metrics
  ↓ If good
Final: 100% new
```

---

## Auto-scaling Metrics

| Metric | Description | Typical Target |
|--------|-------------|----------------|
| **InvocationsPerInstance** | Requests per instance | 1000-5000 |
| **ModelLatency** | Inference time | <1000ms |
| **CPUUtilization** | CPU usage % | 70-80% |
| **MemoryUtilization** | Memory usage % | 70-80% |

```python
# Target tracking policy
TargetValue = 1000  # invocations/instance
ScaleOutCooldown = 60   # seconds
ScaleInCooldown = 300   # seconds
```

---

## SageMaker Pipelines

### Step Types

| Step | Purpose |
|------|---------|
| **Processing** | Data processing, evaluation |
| **Training** | Model training |
| **Tuning** | Hyperparameter optimization |
| **Model** | Create/register model |
| **Condition** | Conditional execution |
| **Transform** | Batch inference |
| **Callback** | External integration |
| **Lambda** | Custom logic |

### Pipeline Structure

```
Data Processing
     ↓
   Training
     ↓
  Evaluation
     ↓
Condition (accuracy > 0.9?)
     ├─ Yes → Register Model → Deploy
     └─ No → Fail/Notify
```

---

## MLOps Components

```
Model Registry:
├── Model Groups (logical grouping)
├── Model Versions
├── Approval Status (Pending/Approved/Rejected)
└── Metadata (metrics, hyperparameters)

SageMaker Projects:
├── Pre-built templates
├── CodePipeline + CodeCommit + CodeBuild
├── Automated workflows
└── Best practices

CI/CD Pipeline:
Code → Build → Train → Evaluate → Register → Deploy → Monitor
```

---

## Model Monitor Types

| Type | Monitors | Baseline From |
|------|----------|---------------|
| **Data Quality** | Input distributions | Training data |
| **Model Quality** | Prediction accuracy | Validation data |
| **Bias Drift** | Fairness metrics | Training data |
| **Feature Attribution** | Feature importance | SHAP values |

### Setup Requirements

1. Enable data capture on endpoint
2. Create baseline from training/validation data
3. Schedule monitoring job (hourly/daily)
4. Configure CloudWatch alerts

---

## Drift Types

| Type | What Changes | Detection | Remediation |
|------|-------------|-----------|-------------|
| **Data Drift** | Input distribution | Statistical tests | Update preprocessing |
| **Concept Drift** | Input-output relationship | Performance degradation | Retrain model |
| **Label Drift** | Target distribution | Class imbalance | Rebalance/retrain |

---

## Security Checklist

### Data Security

- [x] S3 encryption (SSE-S3/KMS)
- [x] EBS encryption (KMS)
- [x] TLS 1.2+ in transit
- [x] Inter-container encryption (distributed training)

### Network Security

- [x] VPC mode
- [x] Private subnets
- [x] Security groups
- [x] VPC endpoints (S3, SageMaker, CloudWatch)
- [x] Network isolation (`enable_network_isolation=True`)

### Access Control

- [x] IAM roles (least privilege)
- [x] Resource-based policies
- [x] MFA for console access
- [x] Temporary credentials

### Compliance

- [x] CloudTrail logging
- [x] AWS Config rules
- [x] Audit Manager
- [x] Compliance certifications (HIPAA, SOC, PCI)

---

## CloudWatch Metrics

### Endpoint Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| **Invocations** | Request count | Trend analysis |
| **ModelLatency** | Inference time | >1000ms |
| **OverheadLatency** | SageMaker overhead | Trend analysis |
| **Invocation4XXErrors** | Client errors | >1% |
| **Invocation5XXErrors** | Server errors | >0.1% |
| **CPUUtilization** | CPU usage | >80% |
| **MemoryUtilization** | Memory usage | >85% |

### Training Metrics

- Training loss
- Validation loss
- Custom metrics (accuracy, F1, etc.)

---

## Cost Optimization

| Strategy | Savings | Use Case |
|----------|---------|----------|
| **Spot Instances** | Up to 90% | Fault-tolerant training |
| **Serverless Inference** | Pay-per-use | Intermittent traffic |
| **Multi-Model Endpoints** | 50-90% | Many models |
| **S3 Lifecycle Policies** | 40-80% | Archival |
| **Right-size Instances** | 20-50% | Match workload |
| **Batch Transform** | 30-60% | vs always-on endpoint |

---

## Inference Optimization

| Technique | Method | Benefit |
|-----------|--------|---------|
| **SageMaker Neo** | Model compilation | 2x faster |
| **Elastic Inference** | GPU acceleration | Cost-effective |
| **Inference Recommender** | Auto-benchmark | Find optimal config |
| **Batching** | Group requests | Higher throughput |
| **Caching** | Store results | Reduce latency |
| **Model Quantization** | Reduce precision | Smaller, faster |

---

## Common Exam Scenarios

### "Choose the Best Algorithm"

| Data Type | Algorithm |
|-----------|-----------|
| Tabular | XGBoost |
| Images | Image Classification/Object Detection |
| Text classification | BlazingText |
| Time series | DeepAR |
| High-dimensional sparse | Factorization Machines |
| Anomaly detection | Random Cut Forest |
| Embeddings | Object2Vec, BlazingText |

### "Choose the Best Deployment"

| Scenario | Solution |
|----------|----------|
| Low latency, consistent traffic | Real-time Endpoint |
| Low latency, intermittent traffic | Serverless |
| Process 10M records daily | Batch Transform |
| Large payloads (100MB images) | Async Inference |
| 500 customer-specific models | Multi-Model Endpoint |
| Zero downtime deployment | Blue/Green |
| Gradual rollout | Canary |

### "Choose the Best Monitoring"

| Need | Solution |
|------|----------|
| Detect input distribution changes | Data Quality Monitor |
| Track prediction accuracy | Model Quality Monitor |
| Monitor fairness | Bias Drift Monitor |
| Feature importance changes | Feature Attribution Monitor |
| Track API calls | CloudTrail |
| Alert on high error rate | CloudWatch Alarms |

---

## Key Exam Tips

1. **Parquet for large datasets** - Always choose Parquet for large-scale ML
2. **XGBoost for tabular** - Default choice for tabular classification/regression
3. **GPU for deep learning** - Use ml.p3/g4dn for computer vision, NLP
4. **Bayesian optimization** - Default and best for hyperparameter tuning
5. **Spot for cost savings** - Up to 90% savings with checkpointing
6. **Feature Store for both** - Online (real-time) + offline (batch) features
7. **Serverless for intermittent** - No idle cost, auto-scales
8. **Batch Transform for bulk** - Large datasets, no persistent endpoint
9. **Blue/Green for safety** - Easy rollback, zero downtime
10. **Data Quality Monitor** - Most commonly used monitoring type
11. **Concept drift = retrain** - Model performance drops, inputs similar
12. **VPC + KMS for HIPAA** - Network isolation + encryption for compliance
13. **Least privilege IAM** - Minimum permissions needed
14. **Point-in-time queries** - Prevent data leakage in Feature Store
15. **Precision for FP cost** - When false positives are expensive
16. **Recall for FN cost** - When false negatives are expensive

---

## Quick Decision Flowcharts

### Data Storage

```
Need real-time feature serving? → Feature Store (online)
Need batch training features? → Feature Store (offline) or S3 Parquet
Need high-throughput training? → FSx for Lustre
Need analytics queries? → Redshift or Athena on S3
Need transactional data? → RDS/Aurora
```

### Encoding Categorical Variables

```
Cardinality < 10? → One-Hot Encoding
Ordinal relationship? → Label Encoding
Cardinality > 50 + Tree model? → Target Encoding
Deep learning? → Embedding Layer
```

### Distributed Training

```
Model fits in GPU memory + large dataset? → Data Parallelism
Model too large for single GPU? → Model Parallelism
Both? → Use both strategies
```

---

## Memory Aids

### PARQUET = Best Format
**P**erformance (fast columnar reads)
**A**nalytics (optimized for)
**R**ecommended (for ML)
**Q**uerying (with Athena)
**U**niversal (works everywhere)
**E**fficient (compression)
**T**abular (data)

### SPOT = Cost Savings
**S**avings (up to 90%)
**P**oint-in-time interruptions
**O**ptional (needs checkpointing)
**T**raining (long-running jobs)

### DRIFT Detection
**D**ata drift = Distribution changes
**R**etrain when detected
**I**nput vs output relationship (concept drift)
**F**eature importance monitoring
**T**rack with Model Monitor

---

## Last-Minute Review

### Must Remember

1. **Domain weights**: 28% Data Prep, 26% Model Dev, 24% Monitoring, 22% Deployment
2. **XGBoost**: #1 tabular algorithm
3. **Parquet**: #1 file format
4. **Feature Store**: Online + offline + point-in-time
5. **Bayesian**: Default tuning strategy
6. **Serverless**: Intermittent traffic
7. **Batch Transform**: Bulk inference
8. **Blue/Green**: Zero downtime
9. **Data Quality Monitor**: Input distribution
10. **VPC + network isolation**: Maximum security

### Common Mistakes to Avoid

- Don't choose CSV over Parquet for large datasets
- Don't use CPU instances for deep learning
- Don't forget to enable data capture for Model Monitor
- Don't deploy without evaluation (use condition step)
- Don't use real-time endpoints for intermittent traffic
- Don't forget point-in-time queries prevent data leakage
- Don't use one-hot encoding for high cardinality
- Don't ignore concept drift (requires retraining)

---

## Navigation

| Previous | Home |
|----------|------|
| [Practice Exam](/posts/aws-mla-c01-practice-exam/) | [Series Overview](/posts/aws-mla-c01-series-overview/) |
