---
title: "AWS MLA-C01 - Complete AWS ML Services Reference for ML Engineers"
author: thanhnv1808
date: 2026-01-24 16:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, services, reference]
description: Comprehensive reference guide for all AWS ML services covered in the MLA-C01 exam. Detailed features, architecture patterns, use cases, and pricing.
pin: false
comments: true
---

## Introduction

This comprehensive reference covers all AWS services ML Engineers need to know for the MLA-C01 exam, with focus on practical implementation, MLOps patterns, and production considerations.

---

## Amazon SageMaker - Complete Reference

### Overview
**Amazon SageMaker** is AWS's fully managed machine learning platform providing end-to-end ML workflow capabilities.

### Core Architecture

```
SageMaker Architecture:
┌─────────────────────────────────────────────────────────────┐
│                    SageMaker Studio (IDE)                    │
├─────────────────────────────────────────────────────────────┤
│  Development  │  Data Prep   │   Training   │  Deployment   │
├───────────────┼──────────────┼──────────────┼───────────────┤
│ Notebooks     │ Data Wrangler│ Training Jobs│ Endpoints     │
│ Studio Lab    │ Processing   │ Tuning       │ Batch Transform│
│ JumpStart     │ Feature Store│ Experiments  │ Edge Manager  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    MLOps & Governance                        │
├─────────────────────────────────────────────────────────────┤
│ Pipelines │ Model Registry │ Model Monitor │ Clarify │ Cards│
└─────────────────────────────────────────────────────────────┘
```

---

## SageMaker Studio

### Features

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Unified IDE** | Single web-based interface | All ML development |
| **Notebooks** | Jupyter notebooks with multiple kernels | Experimentation |
| **Domains** | Multi-user workspace | Team collaboration |
| **User Profiles** | Individual user environments | Access control |
| **Apps** | JupyterServer, KernelGateway | Compute management |

### Domain Architecture

```
SageMaker Domain:
├── Execution Role (IAM)
├── VPC Configuration (optional)
├── User Profiles
│   ├── User 1
│   │   ├── Apps (Jupyter, Kernel)
│   │   └── Home Directory (EFS)
│   └── User 2
│       ├── Apps
│       └── Home Directory
└── Shared Spaces (optional)
```

### Setup Types

| Setup Type | Control Level | Best For |
|------------|---------------|----------|
| **Quick Setup** | Minimal configuration | Learning, testing |
| **Standard Setup** | Full control over networking | Production environments |

### Pricing

- Domain: No charge
- Notebook instances: Per instance-hour
- Apps: Charged when running
- Storage: EFS charges for home directories

---

## SageMaker Training

### Training Job Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Single Instance** | Train on one instance | Small datasets, prototyping |
| **Distributed Data Parallel** | Split data across instances | Large datasets |
| **Distributed Model Parallel** | Split model across instances | Large models (LLMs) |
| **Spot Training** | Use EC2 Spot instances | Cost savings (up to 90%) |
| **Managed Spot** | SageMaker manages checkpoints | Fault-tolerant training |

### Built-in Algorithms

#### Supervised Learning

| Algorithm | Type | Best For |
|-----------|------|----------|
| **XGBoost** | Gradient boosting | Tabular data, classification, regression |
| **Linear Learner** | Linear models | Large-scale linear problems |
| **K-Nearest Neighbors (KNN)** | Instance-based | Classification, regression |
| **Factorization Machines** | Factorization | High-dimensional sparse data |

#### Unsupervised Learning

| Algorithm | Type | Best For |
|-----------|------|----------|
| **K-Means** | Clustering | Customer segmentation |
| **Principal Component Analysis (PCA)** | Dimensionality reduction | Feature reduction |
| **Random Cut Forest (RCF)** | Anomaly detection | Outlier detection |

#### Deep Learning

| Algorithm | Type | Best For |
|-----------|------|----------|
| **Image Classification** | CNN | Image categorization |
| **Object Detection** | CNN | Object localization |
| **Semantic Segmentation** | CNN | Pixel-level classification |
| **Sequence-to-Sequence** | RNN | Machine translation, text summarization |

### Training Input Modes

| Mode | Description | Performance | Cost |
|------|-------------|-------------|------|
| **File Mode** | Download all data before training | Slower start, faster training | Higher storage cost |
| **Pipe Mode** | Stream data during training | Faster start, potentially slower | Lower storage cost |
| **FastFile Mode** | Lazy loading of files | Best of both | Optimal |

### Training Instance Types

| Family | vCPUs | Memory | Network | Best For |
|--------|-------|--------|---------|----------|
| **ml.m5** | 2-96 | 8-384 GB | Up to 25 Gbps | General purpose |
| **ml.c5** | 2-72 | 4-192 GB | Up to 25 Gbps | Compute optimized |
| **ml.p3** | 8-96 | 61-768 GB | Up to 100 Gbps | GPU training (deep learning) |
| **ml.p4d** | 96 | 1152 GB | 400 Gbps | Large-scale deep learning |
| **ml.g4dn** | 4-96 | 16-384 GB | Up to 100 Gbps | Cost-effective GPU |
| **ml.trn1** | 32-128 | 512 GB | Up to 800 Gbps | AWS Trainium (optimized DL) |

### Distributed Training Strategies

#### Data Parallelism

```
Data Parallel Training:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Instance 1  │  │  Instance 2  │  │  Instance 3  │
│              │  │              │  │              │
│  Model Copy  │  │  Model Copy  │  │  Model Copy  │
│      ↓       │  │      ↓       │  │      ↓       │
│  Data Batch 1│  │  Data Batch 2│  │  Data Batch 3│
└──────────────┘  └──────────────┘  └──────────────┘
        ↓                 ↓                 ↓
        └─────────────────┴─────────────────┘
                  Gradient Aggregation
```

**Frameworks Supported:**
- PyTorch DDP (Distributed Data Parallel)
- TensorFlow Distributed
- Horovod

#### Model Parallelism

```
Model Parallel Training:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Instance 1  │  │  Instance 2  │  │  Instance 3  │
│              │  │              │  │              │
│  Layers 1-10 │→ │ Layers 11-20 │→ │ Layers 21-30 │
│              │  │              │  │              │
└──────────────┘  └──────────────┘  └──────────────┘
     Same Data Batch Flows Through
```

**Use Cases:**
- Large language models (billions of parameters)
- Models too large for single GPU memory

### Checkpointing

| Feature | Description |
|---------|-------------|
| **Local Checkpoints** | Saved on training instance |
| **S3 Checkpoints** | Saved to S3 periodically |
| **Managed Spot Checkpointing** | Automatic checkpoint management |

**Best Practices:**
- Checkpoint every 5-10 minutes for Spot training
- Save to S3 for fault tolerance
- Use for long-running training jobs

---

## SageMaker Hyperparameter Tuning

### Tuning Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Bayesian Optimization** | Learns from previous trials | Most efficient, default choice |
| **Random Search** | Random hyperparameter combinations | Baseline comparison |
| **Grid Search** | Exhaustive search | Small hyperparameter space |
| **Hyperband** | Early stopping of poor trials | Large-scale tuning |

### Tuning Job Configuration

```python
HyperparameterTuner(
    estimator=estimator,
    objective_metric_name='validation:accuracy',
    objective_type='Maximize',  # or 'Minimize'
    hyperparameter_ranges={
        'learning_rate': ContinuousParameter(0.001, 0.1),
        'num_layers': IntegerParameter(1, 5),
        'activation': CategoricalParameter(['relu', 'tanh'])
    },
    max_jobs=100,           # Total training jobs
    max_parallel_jobs=10,   # Concurrent jobs
    strategy='Bayesian',
    early_stopping_type='Auto'
)
```

### Parameter Types

| Type | Description | Example |
|------|-------------|---------|
| **ContinuousParameter** | Floating point range | learning_rate (0.001, 0.1) |
| **IntegerParameter** | Integer range | batch_size (32, 256) |
| **CategoricalParameter** | Discrete choices | optimizer ['adam', 'sgd'] |

### Early Stopping

- **Auto**: SageMaker decides when to stop
- **Off**: No early stopping
- Benefits: Saves time and cost by stopping unpromising trials

---

## SageMaker Data Wrangler

### Features

| Feature | Description |
|---------|-------------|
| **Visual Data Preparation** | No-code data transformation |
| **300+ Transformations** | Built-in data operations |
| **Data Quality Analysis** | Automatic quality checks |
| **Feature Engineering** | Create new features |
| **Export Options** | Pipeline, notebook, Python code |

### Data Sources

- Amazon S3
- Amazon Athena
- Amazon Redshift
- Snowflake
- Databricks

### Transformation Categories

| Category | Examples |
|----------|----------|
| **Manage Columns** | Drop, rename, duplicate |
| **Transform** | One-hot encode, ordinal encode |
| **Clean** | Handle missing, outliers |
| **Format** | Change data types |
| **Feature Engineering** | Mathematical operations, binning |
| **Custom** | PySpark, pandas, SQL |

### Data Insights

- Column statistics
- Histogram
- Scatter plots
- Target leakage detection
- Quick model evaluation
- Bias detection

---

## SageMaker Feature Store

### Architecture

```
Feature Store:
┌─────────────────────────────────────────┐
│           Feature Groups                 │
├─────────────────────────────────────────┤
│  Record Identifier  │  Event Time       │
├─────────────────────┼───────────────────┤
│     Feature 1       │    Feature 2      │
│     Feature 3       │    Feature 4      │
└─────────────────────────────────────────┘
           ↓                    ↓
    ┌─────────────┐      ┌─────────────┐
    │Online Store │      │Offline Store│
    │  (DynamoDB) │      │    (S3)     │
    │             │      │             │
    │ Low latency │      │ Historical  │
    │ (ms)        │      │ Analysis    │
    └─────────────┘      └─────────────┘
```

### Store Types

| Store Type | Backend | Latency | Use Case |
|------------|---------|---------|----------|
| **Online Store** | In-memory | Single-digit ms | Real-time inference |
| **Offline Store** | S3 + Glue Catalog | Minutes | Training, batch inference |
| **Both** | Online + Offline | Varies | Most common |

### Feature Group Configuration

```python
feature_group = FeatureGroup(
    name="customer-features",
    sagemaker_session=session
)

feature_group.load_feature_definitions(
    data_frame=df  # Auto-detect schema
)

feature_group.create(
    s3_uri=f"s3://{bucket}/feature-store",
    record_identifier_name="customer_id",
    event_time_feature_name="event_time",
    role_arn=role,
    enable_online_store=True,  # Enable online store
    offline_store_kms_key_id="kms-key-id"  # Optional encryption
)
```

### Querying Features

**Online Store (Low Latency):**
```python
# Get single record
session.boto_session.client('sagemaker-featurestore-runtime').get_record(
    FeatureGroupName="customer-features",
    RecordIdentifierValueAsString="C12345"
)
```

**Offline Store (Athena):**
```python
# SQL query
query = f"""
SELECT customer_id, tenure, monthly_spend
FROM "{feature_group.athena_query().table_name}"
WHERE monthly_spend > 100
"""
feature_group.athena_query().run(query_string=query)
results = feature_group.athena_query().as_dataframe()
```

### Feature Store Benefits

1. **Feature Consistency**: Same features for training and inference
2. **Feature Reusability**: Share features across teams
3. **Point-in-Time Correctness**: Historical feature values
4. **Feature Discovery**: Catalog of available features

---

## SageMaker Pipelines

### Pipeline Components

```
ML Pipeline:
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Processing  │ →  │   Training   │ →  │  Evaluation  │
│     Step     │    │     Step     │    │     Step     │
└──────────────┘    └──────────────┘    └──────────────┘
                                               ↓
                                        ┌──────────────┐
                                        │  Condition   │
                                        │     Step     │
                                        └──────────────┘
                                         ↙            ↘
                                ┌──────────────┐  ┌──────────────┐
                                │   Register   │  │   Reject     │
                                │    Model     │  │    Model     │
                                └──────────────┘  └──────────────┘
```

### Step Types

| Step Type | Purpose | Example |
|-----------|---------|---------|
| **ProcessingStep** | Data processing | Data cleaning, feature engineering |
| **TrainingStep** | Model training | Train XGBoost, PyTorch model |
| **TransformStep** | Batch inference | Score large datasets |
| **CreateModelStep** | Create model artifact | Prepare for deployment |
| **RegisterModelStep** | Register to Model Registry | Version management |
| **ConditionStep** | Conditional logic | Deploy only if accuracy > 0.85 |
| **LambdaStep** | Custom logic | Send notifications, custom validation |
| **TuningStep** | Hyperparameter tuning | Optimize hyperparameters |
| **QualityCheckStep** | Data/model quality | Monitor drift |
| **ClarifyCheckStep** | Bias detection | Check for bias |

### Pipeline Parameters

```python
from sagemaker.workflow.parameters import (
    ParameterInteger,
    ParameterString,
    ParameterFloat,
    ParameterBoolean
)

# Define parameters
instance_type = ParameterString(
    name="TrainingInstanceType",
    default_value="ml.m5.xlarge"
)

max_depth = ParameterInteger(
    name="MaxDepth",
    default_value=5
)

learning_rate = ParameterFloat(
    name="LearningRate",
    default_value=0.1
)
```

### Conditional Execution

```python
from sagemaker.workflow.conditions import ConditionGreaterThanOrEqualTo
from sagemaker.workflow.functions import JsonGet

# Condition: accuracy >= 0.85
cond = ConditionGreaterThanOrEqualTo(
    left=JsonGet(
        step_name="EvaluateModel",
        property_file=evaluation_report,
        json_path="metrics.accuracy.value"
    ),
    right=0.85
)

# Conditional step
step_cond = ConditionStep(
    name="CheckAccuracy",
    conditions=[cond],
    if_steps=[step_register_model],
    else_steps=[step_notify_failure]
)
```

### Pipeline Execution

```python
# Create pipeline
pipeline = Pipeline(
    name="ml-pipeline",
    parameters=[instance_type, max_depth],
    steps=[step_process, step_train, step_eval, step_cond]
)

# Upsert (create or update)
pipeline.upsert(role_arn=role)

# Execute pipeline
execution = pipeline.start(
    parameters={
        "TrainingInstanceType": "ml.m5.2xlarge",
        "MaxDepth": 7
    }
)

# Monitor
execution.wait()
execution.list_steps()
```

---

## SageMaker Model Deployment

### Deployment Options

| Option | Latency | Throughput | Cost | Use Case |
|--------|---------|------------|------|----------|
| **Real-time Endpoint** | Low (ms) | Medium | Hourly instance charge | Interactive applications |
| **Serverless Endpoint** | Medium (1-2s) | Low-Medium | Per-request | Intermittent traffic |
| **Asynchronous Endpoint** | High (min) | High | Hourly + per-request | Large payloads, long processing |
| **Batch Transform** | High (min-hr) | Very High | Per-job | Offline scoring |
| **Edge Deployment** | Lowest (on-device) | Device-dependent | One-time | IoT, mobile apps |

### Real-time Endpoints

#### Endpoint Architecture

```
Real-time Endpoint:
┌─────────────────────────────────────────┐
│           Endpoint (Load Balancer)       │
└─────────────────────────────────────────┘
                    ↓
    ┌───────────────┴───────────────┐
    ↓                               ↓
┌─────────────┐              ┌─────────────┐
│  Variant A  │              │  Variant B  │
│  (70% wt)   │              │  (30% wt)   │
├─────────────┤              ├─────────────┤
│ Model A     │              │ Model B     │
│ Instance 1  │              │ Instance 1  │
│ Instance 2  │              │             │
└─────────────┘              └─────────────┘
```

#### Auto-scaling Configuration

```python
# Configure auto-scaling
client = boto3.client('application-autoscaling')

# Register scalable target
client.register_scalable_target(
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    MinCapacity=1,
    MaxCapacity=10
)

# Define scaling policy
client.put_scaling_policy(
    PolicyName='scale-on-invocations',
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    PolicyType='TargetTrackingScaling',
    TargetTrackingScalingPolicyConfiguration={
        'TargetValue': 1000.0,  # Target invocations per instance
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'SageMakerVariantInvocationsPerInstance'
        },
        'ScaleInCooldown': 300,   # 5 minutes
        'ScaleOutCooldown': 60    # 1 minute
    }
)
```

### Serverless Endpoints

**Benefits:**
- No instance management
- Automatic scaling (0 to N)
- Pay only for inference time

**Limitations:**
- 4 GB model size limit
- 60 second timeout
- Cold start latency (1-2 seconds)

**Configuration:**
```python
predictor = model.deploy(
    serverless_inference_config=ServerlessInferenceConfig(
        memory_size_in_mb=4096,  # 1024, 2048, 3072, 4096, 5120, 6144
        max_concurrency=20       # Max concurrent invocations
    )
)
```

### Multi-Model Endpoints

**Architecture:**
```
Multi-Model Endpoint:
┌────────────────────────────────────┐
│  Single Endpoint Instance          │
├────────────────────────────────────┤
│  Model Cache (Memory)              │
│  ┌──────┐ ┌──────┐ ┌──────┐      │
│  │Model1│ │Model2│ │Model3│ ...  │
│  └──────┘ └──────┘ └──────┘      │
├────────────────────────────────────┤
│  Model Storage (S3)                │
│  model1.tar.gz, model2.tar.gz...   │
└────────────────────────────────────┘
```

**Use Cases:**
- Hundreds/thousands of similar models
- Models for different customers/regions
- Infrequently accessed models

**Cost Savings:**
- Share compute across models
- Up to 90% cost reduction vs separate endpoints

### Inference Pipelines

```
Serial Inference Pipeline:
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Preprocessing│ → │   Model 1    │ → │   Model 2    │
│  (Spark)     │   │  (XGBoost)   │   │   (Custom)   │
└──────────────┘   └──────────────┘   └──────────────┘
```

**Benefits:**
- Single endpoint for entire workflow
- Lower latency (no inter-service calls)
- Simplified deployment

---

## SageMaker Model Monitor

### Monitoring Types

| Type | Monitors | Alerts On |
|------|----------|-----------|
| **Data Quality** | Input data distribution | Feature drift, missing values |
| **Model Quality** | Prediction accuracy | Accuracy degradation |
| **Bias Drift** | Bias metrics | Emerging bias |
| **Feature Attribution** | Feature importance | Importance changes |

### Data Quality Monitoring

```
Data Quality Monitoring Flow:
┌──────────────┐
│   Endpoint   │ → Captures Input/Output
└──────────────┘
       ↓
┌──────────────┐
│ Data Capture │ → Stores to S3
│   (S3)       │
└──────────────┘
       ↓
┌──────────────┐
│  Monitoring  │ → Compares to Baseline
│   Schedule   │
└──────────────┘
       ↓
┌──────────────┐
│   Violations │ → CloudWatch Metrics
│   Report     │
└──────────────┘
```

### Baseline Creation

```python
from sagemaker.model_monitor import DefaultModelMonitor

monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge'
)

# Create baseline statistics
monitor.suggest_baseline(
    baseline_dataset='s3://bucket/training-data.csv',
    dataset_format=DatasetFormat.csv(header=False),
    output_s3_uri='s3://bucket/baseline'
)
```

### Monitoring Schedule

```python
from sagemaker.model_monitor import CronExpressionGenerator

monitor.create_monitoring_schedule(
    monitor_schedule_name='hourly-monitoring',
    endpoint_input=endpoint_name,
    output_s3_uri='s3://bucket/monitoring-reports',
    statistics=monitor.baseline_statistics(),
    constraints=monitor.suggested_constraints(),
    schedule_cron_expression=CronExpressionGenerator.hourly(),
    enable_cloudwatch_metrics=True
)
```

### Violation Detection

**Types of Violations:**
- Data type mismatch
- Missing required features
- Out-of-range values
- Distribution drift (statistical distance)

**Actions:**
- CloudWatch alarm
- SNS notification
- Lambda trigger for automated response

---

## SageMaker Clarify

### Bias Detection

#### Pre-training Bias Metrics

| Metric | Description | Formula |
|--------|-------------|---------|
| **Class Imbalance (CI)** | Difference in class representation | (n_a - n_d) / (n_a + n_d) |
| **Difference in Proportions of Labels (DPL)** | Label distribution difference | q_a - q_d |
| **KL Divergence** | Distribution divergence | KL(P_a \|\| P_d) |
| **Jensen-Shannon Divergence** | Symmetric distribution divergence | JS(P_a, P_d) |

**Interpretation:**
- **CI**: Values near 0 are balanced
- **DPL**: Values near 0 indicate equal label proportions
- **KL/JS**: Lower values indicate similar distributions

#### Post-training Bias Metrics

| Metric | Description | Use Case |
|--------|-------------|----------|
| **Difference in Positive Proportions in Predicted Labels (DPPL)** | Prediction rate difference | Classification fairness |
| **Disparate Impact (DI)** | Ratio of positive prediction rates | Regulatory compliance |
| **Difference in Conditional Acceptance (DCA)** | Acceptance rate difference | Loan/admission decisions |
| **Conditional Demographic Disparity (CDD)** | Outcome disparity controlling for label | Fine-grained fairness |

### Explainability (SHAP)

**SHAP Values:**
- Explain individual predictions
- Feature importance
- Global and local explanations

```python
from sagemaker.clarify import SHAPConfig

shap_config = SHAPConfig(
    baseline=[baseline_data],
    num_samples=100,
    agg_method='mean_abs',
    save_local_shap_values=True
)

clarify_processor.run_explainability(
    data_config=data_config,
    model_config=model_config,
    explainability_config=shap_config
)
```

---

## SageMaker Model Registry

### Model Package Groups

```
Model Registry:
┌────────────────────────────────────┐
│  Model Package Group: churn-model  │
├────────────────────────────────────┤
│  Version 1 │ Status: Approved      │
│  Version 2 │ Status: PendingApproval│
│  Version 3 │ Status: Rejected      │
└────────────────────────────────────┘
```

### Model Approval Status

| Status | Description | Actions |
|--------|-------------|---------|
| **PendingManualApproval** | Awaiting review | Manual approval needed |
| **Approved** | Ready for production | Can be deployed |
| **Rejected** | Not suitable | Cannot be deployed |

### Model Package Creation

```python
from sagemaker.model import Model
from sagemaker.model_metrics import MetricsSource, ModelMetrics

model.register(
    content_types=["text/csv"],
    response_types=["application/json"],
    inference_instances=["ml.m5.large", "ml.m5.xlarge"],
    transform_instances=["ml.m5.xlarge"],
    model_package_group_name="customer-churn-models",
    approval_status="PendingManualApproval",
    model_metrics=ModelMetrics(
        model_statistics=MetricsSource(
            s3_uri="s3://bucket/metrics/statistics.json",
            content_type="application/json"
        ),
        bias=MetricsSource(
            s3_uri="s3://bucket/metrics/bias.json",
            content_type="application/json"
        ),
        explainability=MetricsSource(
            s3_uri="s3://bucket/metrics/explainability.json",
            content_type="application/json"
        )
    ),
    metadata_properties={
        "ProjectId": "proj-123",
        "GeneratedBy": "ml-engineer@company.com"
    },
    customer_metadata_properties={
        "TrainingDataset": "s3://bucket/data/train.csv",
        "ValidationAccuracy": "0.92"
    }
)
```

---

## SageMaker JumpStart

### Categories

| Category | Description | Models |
|----------|-------------|--------|
| **Foundation Models** | Pre-trained LLMs | BERT, GPT-J, BLOOM, Flan-T5 |
| **Vision Models** | Image tasks | ResNet, EfficientNet, YOLO |
| **Text Models** | NLP tasks | RoBERTa, DistilBERT |
| **Tabular Models** | Structured data | LightGBM, CatBoost, TabNet |
| **Solutions** | End-to-end workflows | Fraud detection, churn prediction |

### Deployment Options

1. **One-Click Deploy**: Deploy pre-trained model
2. **Fine-tune**: Train on your data
3. **Incremental Training**: Continue training existing model

---

## AWS Glue for ML

### Glue Components for ML

| Component | Purpose | ML Use Case |
|-----------|---------|-------------|
| **Glue Crawlers** | Schema discovery | Catalog datasets |
| **Glue Data Catalog** | Metadata repository | Feature Store backend |
| **Glue ETL Jobs** | Data transformation | Data preparation |
| **Glue DataBrew** | Visual data prep | No-code feature engineering |

### Glue Integration with SageMaker

```python
# Use Glue Catalog with SageMaker Processing
from sagemaker.processing import ProcessingInput

processing_input = ProcessingInput(
    source=f"s3://{bucket}/raw-data/",
    destination="/opt/ml/processing/input",
    input_name="glue-input",
    s3_data_type="S3Prefix",
    s3_input_mode="File"
)
```

---

## Amazon EMR for ML

### EMR for ML Workloads

**Use Cases:**
- Large-scale data preprocessing
- Feature engineering on big data
- Spark MLlib training
- Data lake integration

### EMR + SageMaker Integration

```
EMR Processing → SageMaker Training:
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Raw Data    │  →  │ EMR Cluster  │  →  │  Processed   │
│  (S3/HDFS)   │     │ (Spark ETL)  │     │  Features    │
└──────────────┘     └──────────────┘     └──────────────┘
                                                  ↓
                                          ┌──────────────┐
                                          │  SageMaker   │
                                          │   Training   │
                                          └──────────────┘
```

### SageMaker Spark SDK

```python
from sagemaker_pyspark import SageMakerEstimator

estimator = SageMakerEstimator(
    trainingImage=training_image,
    modelImage=inference_image,
    roleArn=role,
    requestRowSerializer=ProtobufRequestRowSerializer(),
    responseRowDeserializer=ProtobufResponseRowDeserializer(),
    hyperParameters={'num_round': '100', 'max_depth': '5'}
)

# Train on Spark DataFrame
model = estimator.fit(spark_df)
```

---

## AWS Step Functions for ML Workflows

### Step Functions + SageMaker

**Use Cases:**
- Orchestrate complex ML workflows
- Human-in-the-loop workflows
- Multi-stage pipelines
- Error handling and retries

### Workflow Example

```json
{
  "StartAt": "DataPreprocessing",
  "States": {
    "DataPreprocessing": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createProcessingJob.sync",
      "Parameters": {
        "ProcessingJobName.$": "$.jobName",
        "RoleArn": "arn:aws:iam::123456789012:role/SageMakerRole"
      },
      "Next": "TrainModel"
    },
    "TrainModel": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createTrainingJob.sync",
      "Parameters": {
        "TrainingJobName.$": "$.trainingJobName"
      },
      "Next": "EvaluateModel"
    },
    "EvaluateModel": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:evaluate-model",
      "Next": "CheckAccuracy"
    },
    "CheckAccuracy": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.accuracy",
          "NumericGreaterThan": 0.85,
          "Next": "DeployModel"
        }
      ],
      "Default": "NotifyFailure"
    },
    "DeployModel": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createEndpoint",
      "End": true
    },
    "NotifyFailure": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "End": true
    }
  }
}
```

---

## Amazon CloudWatch for ML Monitoring

### CloudWatch Metrics for SageMaker

#### Training Metrics

| Metric | Description | Namespace |
|--------|-------------|-----------|
| **TrainingJobStatus** | Job status | AWS/SageMaker |
| **ResourceUtilization** | CPU, GPU, memory usage | /aws/sagemaker/TrainingJobs |

#### Endpoint Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| **Invocations** | Request count | > baseline |
| **ModelLatency** | Inference time (ms) | > 500ms |
| **Invocation4XXErrors** | Client errors | > 0 |
| **Invocation5XXErrors** | Server errors | > 0 |
| **CPUUtilization** | CPU usage % | > 80% |
| **MemoryUtilization** | Memory usage % | > 80% |
| **DiskUtilization** | Disk usage % | > 80% |

### CloudWatch Alarms

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Create alarm for high latency
cloudwatch.put_metric_alarm(
    AlarmName='high-inference-latency',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='ModelLatency',
    Namespace='AWS/SageMaker',
    Period=300,
    Statistic='Average',
    Threshold=500.0,  # 500ms
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:ml-alerts'],
    Dimensions=[
        {'Name': 'EndpointName', 'Value': 'my-endpoint'},
        {'Name': 'VariantName', 'Value': 'AllTraffic'}
    ]
)
```

---

## Amazon EventBridge for ML Automation

### EventBridge Patterns for ML

#### Training Job Completion

```json
{
  "source": ["aws.sagemaker"],
  "detail-type": ["SageMaker Training Job State Change"],
  "detail": {
    "TrainingJobStatus": ["Completed"]
  }
}
```

#### Model Monitor Violations

```json
{
  "source": ["aws.sagemaker"],
  "detail-type": ["SageMaker Model Monitor Execution Status Change"],
  "detail": {
    "MonitoringExecutionStatus": ["CompletedWithViolations"]
  }
}
```

### Automated Workflows

**Trigger Actions:**
- Lambda function
- Step Functions workflow
- SNS notification
- SageMaker Pipeline execution

---

## AWS Lambda for ML Inference

### Lambda Use Cases

| Use Case | Description |
|----------|-------------|
| **Feature Engineering** | Pre-process inputs before inference |
| **Model Serving** | Serve small models directly |
| **Post-processing** | Transform model outputs |
| **Routing** | Route requests to appropriate model |

### Lambda + SageMaker Pattern

```python
import json
import boto3

runtime = boto3.client('sagemaker-runtime')

def lambda_handler(event, context):
    # Pre-process
    features = extract_features(event)

    # Invoke SageMaker endpoint
    response = runtime.invoke_endpoint(
        EndpointName='my-endpoint',
        ContentType='text/csv',
        Body=','.join(map(str, features))
    )

    # Post-process
    prediction = json.loads(response['Body'].read())
    result = transform_output(prediction)

    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }
```

### Lambda Limitations for ML

- **15-minute timeout**: Not for long-running inference
- **10 GB memory max**: Limited model size
- **250 MB deployment package**: Use container images for larger models
- **Cold start**: 1-2 second latency

---

## IAM for ML Workloads

### SageMaker Execution Role

**Required Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-ml-bucket/*",
        "arn:aws:s3:::my-ml-bucket"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```

### Least Privilege Principle

**Training Job Role:**
- Read training data from S3
- Write model artifacts to S3
- Write logs to CloudWatch

**Endpoint Role:**
- Read model artifacts from S3
- Write logs to CloudWatch
- (No S3 write access)

---

## Service Comparison Tables

### When to Use Which Service

| Need | Service |
|------|---------|
| **End-to-end ML platform** | SageMaker |
| **Visual data preparation** | Data Wrangler / Glue DataBrew |
| **Feature management** | Feature Store |
| **Large-scale ETL** | Glue / EMR |
| **Workflow orchestration** | SageMaker Pipelines / Step Functions |
| **Real-time inference** | SageMaker Real-time Endpoints |
| **Batch inference** | Batch Transform |
| **Serverless inference** | Serverless Endpoints / Lambda |
| **Model monitoring** | Model Monitor |
| **Bias detection** | Clarify |
| **Metrics & alarms** | CloudWatch |
| **Event-driven automation** | EventBridge |

### Training vs Inference Compute

| Workload | Compute Type | Pricing Model |
|----------|--------------|---------------|
| **Training** | On-demand instances | Per second, min 1 minute |
| **Training (Spot)** | Spot instances | Up to 90% discount |
| **Real-time Endpoint** | Always-on instances | Per hour |
| **Serverless Endpoint** | Auto-scaling | Per inference request |
| **Batch Transform** | On-demand job | Per job duration |

---

## Quick Reference Cards

### SageMaker Training Quick Reference

```
Training Job:
├── Algorithm (Built-in / BYOC)
├── Input Data (S3)
├── Instance Type & Count
├── Hyperparameters
├── Output Path (S3)
└── Distributed Training (optional)
    ├── Data Parallel
    └── Model Parallel
```

### SageMaker Deployment Quick Reference

```
Deployment Options:
├── Real-time Endpoint
│   ├── Standard (fixed instances)
│   ├── Auto-scaling
│   ├── Multi-model
│   └── Multi-variant (A/B)
├── Serverless Endpoint
├── Asynchronous Endpoint
├── Batch Transform
└── Edge Deployment
```

### MLOps Pipeline Quick Reference

```
Complete MLOps Pipeline:
1. Data Ingestion
   ├── S3 / Database / Streaming
   └── Glue Crawlers (catalog)

2. Data Processing
   ├── Glue ETL / EMR (large scale)
   ├── SageMaker Processing (ML-focused)
   └── Data Wrangler (visual)

3. Feature Engineering
   └── Feature Store (online/offline)

4. Model Training
   ├── SageMaker Training Jobs
   ├── Hyperparameter Tuning
   └── Experiments

5. Model Evaluation
   ├── Processing Job (custom metrics)
   └── Clarify (bias, explainability)

6. Model Registry
   └── Version control & approval

7. Model Deployment
   ├── Endpoints
   └── A/B Testing

8. Monitoring
   ├── Model Monitor (drift)
   ├── CloudWatch (metrics)
   └── Clarify (bias drift)

9. Orchestration
   ├── SageMaker Pipelines
   └── Step Functions

10. Automation
    └── EventBridge (triggers)
```

---

## Pricing Considerations

### Cost Optimization Strategies

| Strategy | Savings | Trade-off |
|----------|---------|-----------|
| **Spot Training** | Up to 90% | Possible interruptions |
| **Serverless Endpoints** | Pay-per-use | Cold start latency |
| **Multi-Model Endpoints** | Up to 90% | Model loading latency |
| **Batch Transform** | Job-based | No real-time capability |
| **Auto-scaling** | 30-70% | Configuration complexity |
| **Right-sizing instances** | 20-50% | Performance testing needed |

### Instance Type Selection Guide

| Workload | Instance Family | Example |
|----------|-----------------|---------|
| **Small tabular models** | ml.m5 | ml.m5.large |
| **Large tabular models** | ml.c5 | ml.c5.2xlarge |
| **Computer vision** | ml.p3 / ml.g4dn | ml.p3.2xlarge |
| **NLP (small models)** | ml.g4dn | ml.g4dn.xlarge |
| **NLP (large models)** | ml.p4d / ml.trn1 | ml.p4d.24xlarge |
| **Inference (CPU)** | ml.m5 / ml.c5 | ml.m5.large |
| **Inference (GPU)** | ml.g4dn | ml.g4dn.xlarge |

---

**Back to Series**: [AWS Machine Learning Engineer Associate (MLA-C01) - Complete Study Guide](/posts/aws-mla-c01-series/)

---

*Questions about AWS ML services? Leave a comment below!*
