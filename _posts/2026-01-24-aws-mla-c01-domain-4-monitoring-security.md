---
title: "AWS MLA-C01 - Domain 4: ML Solution Monitoring, Maintenance, and Security"
author: thanhnv1808
date: 2026-01-24 13:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, monitoring, security, model-monitor, drift, iam]
description: "Domain 4 covers 24% of the exam (~15 questions). Master model monitoring, drift detection, retraining strategies, security best practices, and compliance."
pin: false
comments: true
---

## Domain 4 Overview

**Exam Weight: 24% (~15 questions)**

This domain focuses on monitoring ML models in production, detecting and addressing drift, implementing security controls, and maintaining model performance over time.

### Task Statements

| Task | Description |
|------|-------------|
| 4.1 | Monitor ML models in production |
| 4.2 | Detect and respond to data and model drift |
| 4.3 | Implement model retraining and updating strategies |
| 4.4 | Secure ML solutions and ensure compliance |

---

## Task 4.1: Monitor ML Models in Production

### Monitoring Dimensions

```
┌─────────────────────────────────────────────────────────────────┐
│                    ML MONITORING DIMENSIONS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │  Model      │   │   Data      │   │Infrastructure│         │
│   │ Performance │   │   Quality   │   │   Health    │          │
│   │ • Accuracy  │   │ • Drift     │   │ • Latency   │          │
│   │ • Precision │   │ • Missing   │   │ • Errors    │          │
│   │ • Recall    │   │ • Outliers  │   │ • CPU/Memory│          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │  Business   │   │   Bias      │   │   Cost      │          │
│   │   Metrics   │   │             │   │             │          │
│   │ • Revenue   │   │ • Fairness  │   │ • Resource  │          │
│   │ • Users     │   │ • Equity    │   │ • Budget    │          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SageMaker Model Monitor

**SageMaker Model Monitor** continuously monitors model quality in production.

| Monitor Type | Purpose | Detects |
|--------------|---------|---------|
| **Data Quality** | Monitor input data distribution | Data drift, missing features, type changes |
| **Model Quality** | Monitor prediction quality | Model drift, accuracy degradation |
| **Bias Drift** | Monitor for bias | Bias introduction over time |
| **Feature Attribution** | Monitor feature importance | Feature drift, attribution changes |

### Data Quality Monitoring

```
┌─────────────────────────────────────────────────────────────────┐
│                  DATA QUALITY MONITORING FLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Training Data (Baseline)                                      │
│   ┌──────────────────┐                                          │
│   │ Create Baseline  │                                          │
│   │ • Statistics     │                                          │
│   │ • Constraints    │                                          │
│   └────────┬─────────┘                                          │
│            │                                                     │
│            ▼                                                     │
│   Production Data → Monitor → Compare → Violations?             │
│   ┌──────────────┐   ┌──────┐  ┌──────┐    │                   │
│   │ Endpoint     │   │Check │  │Stats │    ├─Yes→ Alert        │
│   │ Captures     │   │Data  │  │Match?│    │                   │
│   └──────────────┘   └──────┘  └──────┘    └─No → Continue     │
│                                                                  │
│   Monitored Aspects:                                            │
│   • Feature distributions                                       │
│   • Missing values                                              │
│   • Data types                                                  │
│   • Statistical properties (mean, std, min, max)                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Setting up Data Quality Monitoring:**

```python
from sagemaker.model_monitor import DataCaptureConfig, DefaultModelMonitor
from sagemaker.model_monitor.dataset_format import DatasetFormat

# Enable data capture on endpoint
data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,
    destination_s3_uri='s3://bucket/data-capture',
    capture_options=['Input', 'Output']
)

predictor = model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    data_capture_config=data_capture_config
)

# Create baseline from training data
baseline_processor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    max_runtime_in_seconds=3600
)

baseline_processor.suggest_baseline(
    baseline_dataset='s3://bucket/training-data/train.csv',
    dataset_format=DatasetFormat.csv(header=True),
    output_s3_uri='s3://bucket/baseline',
    wait=True
)

# Schedule monitoring
monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    max_runtime_in_seconds=3600
)

monitor.create_monitoring_schedule(
    monitor_schedule_name='data-quality-monitor',
    endpoint_input=predictor.endpoint_name,
    statistics=baseline_processor.baseline_statistics(),
    constraints=baseline_processor.suggested_constraints(),
    schedule_cron_expression='cron(0 * * * ? *)',  # Hourly
    output_s3_uri='s3://bucket/monitoring-results'
)
```

### Model Quality Monitoring

**Model Quality Monitoring** tracks model prediction accuracy against ground truth labels.

| Metric | Description | For |
|--------|-------------|-----|
| **Accuracy** | Overall correctness | Classification |
| **Precision** | Positive prediction accuracy | Classification |
| **Recall** | True positive rate | Classification |
| **F1 Score** | Harmonic mean | Classification |
| **AUC** | Area under ROC curve | Binary classification |
| **MSE/RMSE** | Prediction error | Regression |
| **MAE** | Absolute error | Regression |

```python
from sagemaker.model_monitor import ModelQualityMonitor

model_quality_monitor = ModelQualityMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    max_runtime_in_seconds=3600
)

# Create baseline
model_quality_monitor.suggest_baseline(
    baseline_dataset='s3://bucket/validation-data-with-predictions.csv',
    dataset_format=DatasetFormat.csv(header=True),
    problem_type='BinaryClassification',
    inference_attribute='prediction',
    probability_attribute='probability',
    ground_truth_attribute='label',
    output_s3_uri='s3://bucket/model-quality-baseline'
)

# Schedule monitoring
model_quality_monitor.create_monitoring_schedule(
    monitor_schedule_name='model-quality-monitor',
    endpoint_input=predictor.endpoint_name,
    problem_type='BinaryClassification',
    ground_truth_input='s3://bucket/ground-truth/',
    constraints=model_quality_monitor.suggested_constraints(),
    schedule_cron_expression='cron(0 0 * * ? *)',  # Daily
    output_s3_uri='s3://bucket/model-quality-results'
)
```

### CloudWatch Metrics for Endpoints

| Metric | Description | Threshold |
|--------|-------------|-----------|
| **Invocations** | Number of requests | Monitor for traffic changes |
| **ModelLatency** | Time for model inference | < 1 second for real-time |
| **OverheadLatency** | SageMaker overhead | Monitor trends |
| **Invocation4XXErrors** | Client errors | < 1% |
| **Invocation5XXErrors** | Server errors | < 0.1% |
| **ModelSetupTime** | Time to load model | Monitor cold starts |
| **CPUUtilization** | CPU usage | < 80% sustained |
| **MemoryUtilization** | Memory usage | < 85% |
| **DiskUtilization** | Disk usage | < 90% |

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Create alarm for high error rate
cloudwatch.put_metric_alarm(
    AlarmName='high-endpoint-errors',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='Invocation5XXErrors',
    Namespace='AWS/SageMaker',
    Period=300,
    Statistic='Sum',
    Threshold=10,
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:alert-topic'],
    Dimensions=[
        {'Name': 'EndpointName', 'Value': 'my-endpoint'},
        {'Name': 'VariantName', 'Value': 'AllTraffic'}
    ]
)
```

### Logging and Debugging

| Service | Purpose | Use Case |
|---------|---------|----------|
| **CloudWatch Logs** | Centralized logging | Application logs, errors |
| **CloudTrail** | API call history | Audit, compliance |
| **SageMaker Debugger** | Training debugging | Tensor analysis, profiling |
| **SageMaker Model Monitor** | Production monitoring | Drift detection |

**CloudWatch Logs Insights Query:**

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by endpoint_name
| sort count desc
```

> **Exam Tip**: SageMaker Model Monitor has four types: Data Quality, Model Quality, Bias Drift, and Feature Attribution. Data Quality monitoring is the most commonly used.
{: .prompt-tip }

---

## Task 4.2: Detect and Respond to Data and Model Drift

### Types of Drift

```
┌─────────────────────────────────────────────────────────────────┐
│                        TYPES OF DRIFT                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Data Drift (Covariate Shift)                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ Training: Age distribution 20-60, mean=40                │  │
│   │ Production: Age distribution 25-70, mean=50  ← DRIFT     │  │
│   │                                                           │  │
│   │ Input feature distribution changes over time             │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Concept Drift (Posterior Shift)                               │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ Training: Income > 50K → Approved                        │  │
│   │ Production: Income > 50K → Rejected  ← DRIFT             │  │
│   │                                                           │  │
│   │ Relationship between features and target changes         │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Label Drift (Prior Shift)                                     │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ Training: 50% fraud, 50% legitimate                      │  │
│   │ Production: 5% fraud, 95% legitimate  ← DRIFT            │  │
│   │                                                           │  │
│   │ Distribution of target variable changes                  │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Drift Detection Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **Statistical Tests** | Compare distributions (KS test, Chi-square) | Numerical/categorical features |
| **Distance Metrics** | KL divergence, Wasserstein distance | Distribution comparison |
| **Threshold Monitoring** | Compare statistics (mean, std) to baseline | Simple drift detection |
| **Model Performance** | Track accuracy, precision, recall | Concept drift |
| **Feature Attribution** | Monitor feature importance changes | Feature drift |

### Data Drift Detection

**Statistical Constraint Violations:**

```json
{
  "violations": [
    {
      "feature_name": "age",
      "constraint_check_type": "data_drift_check",
      "description": "Mean has drifted from 40.5 to 52.3 (threshold: 3 std dev)",
      "metric": "mean",
      "baseline_value": 40.5,
      "current_value": 52.3
    },
    {
      "feature_name": "income",
      "constraint_check_type": "missing_values_check",
      "description": "Missing value percentage increased from 2% to 15%",
      "baseline_value": 0.02,
      "current_value": 0.15
    }
  ]
}
```

### Model Drift Detection

**Performance Degradation Indicators:**

| Indicator | Description | Action |
|-----------|-------------|--------|
| **Accuracy Drop** | Overall accuracy decreases | Investigate and retrain |
| **Precision Drop** | More false positives | Check data quality, retrain |
| **Recall Drop** | More false negatives | Review feature relevance |
| **Prediction Distribution Shift** | Output distribution changes | Analyze input drift |
| **Confidence Calibration** | Predicted probabilities miscalibrated | Recalibrate or retrain |

### Drift Response Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                   DRIFT RESPONSE DECISION TREE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Drift Detected                                                │
│         │                                                        │
│         ▼                                                        │
│   Is it data drift?                                             │
│   ├─ Yes → ┌─────────────────────────────────┐                  │
│   │        │ • Validate data sources         │                  │
│   │        │ • Check data pipeline           │                  │
│   │        │ • Update feature engineering    │                  │
│   │        │ • Consider retraining           │                  │
│   │        └─────────────────────────────────┘                  │
│   │                                                              │
│   └─ No → Is it concept drift?                                  │
│          ├─ Yes → ┌─────────────────────────┐                   │
│          │        │ • Retrain with recent   │                   │
│          │        │   data                  │                   │
│          │        │ • Update model          │                   │
│          │        │ • Consider online       │                   │
│          │        │   learning              │                   │
│          │        └─────────────────────────┘                   │
│          │                                                       │
│          └─ No → Infrastructure issue?                          │
│                   └─ Check logs, resources                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Alerting on Drift

```python
import boto3

sns = boto3.client('sns')

# Process monitoring results
def process_violations(violations_file):
    with open(violations_file) as f:
        violations = json.load(f)

    if violations['violations']:
        # Send alert
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789012:drift-alerts',
            Subject='Model Drift Detected',
            Message=f"Violations detected: {json.dumps(violations, indent=2)}"
        )

        # Trigger retraining pipeline
        trigger_retraining()

# Lambda function to check monitoring results
def lambda_handler(event, context):
    s3_location = event['detail']['MonitoringScheduleArn']
    violations_file = download_from_s3(s3_location)
    process_violations(violations_file)
```

> **Exam Tip**: Data drift changes input distribution, concept drift changes the relationship between inputs and outputs. Both require different remediation strategies.
{: .prompt-warning }

---

## Task 4.3: Implement Model Retraining and Updating Strategies

### Retraining Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                     RETRAINING STRATEGIES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Scheduled Retraining           Triggered Retraining           │
│   ┌──────────────────┐           ┌──────────────────┐           │
│   │ • Daily          │           │ • Drift detected │           │
│   │ • Weekly         │           │ • Performance    │           │
│   │ • Monthly        │           │   degradation    │           │
│   │ • Fixed interval │           │ • New data       │           │
│   │                  │           │   available      │           │
│   └──────────────────┘           └──────────────────┘           │
│                                                                  │
│   Continuous/Online Learning     Manual Retraining              │
│   ┌──────────────────┐           ┌──────────────────┐           │
│   │ • Real-time      │           │ • On-demand      │           │
│   │   updates        │           │ • After review   │           │
│   │ • Incremental    │           │ • Major changes  │           │
│   │   learning       │           │                  │           │
│   └──────────────────┘           └──────────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Retraining Data Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **All Historical Data** | Retrain on entire dataset | When all data is relevant |
| **Recent Data Only** | Use only recent time window | Fast-changing patterns |
| **Weighted Data** | Give more weight to recent data | Gradual pattern changes |
| **Sliding Window** | Fixed-size window of recent data | Time series, streaming |
| **Incremental Learning** | Update model with new data only | Online learning scenarios |

### Automated Retraining Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                  AUTOMATED RETRAINING PIPELINE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐                                              │
│   │   Trigger    │                                              │
│   │ (Drift/Time) │                                              │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐      ┌──────────────┐                        │
│   │  Fetch New   │─────▶│   Validate   │                        │
│   │    Data      │      │     Data     │                        │
│   └──────────────┘      └──────┬───────┘                        │
│                                │                                 │
│                                ▼                                 │
│   ┌──────────────┐      ┌──────────────┐                        │
│   │   Register   │◀─────│    Train     │                        │
│   │    Model     │      │  New Model   │                        │
│   └──────┬───────┘      └──────────────┘                        │
│          │                     ▲                                 │
│          ▼                     │ No                              │
│   ┌──────────────┐      ┌──────────────┐                        │
│   │    Deploy    │◀─────│   Better     │                        │
│   │  (Canary)    │  Yes │ Performance? │                        │
│   └──────────────┘      └──────────────┘                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**SageMaker Pipeline for Retraining:**

```python
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import ProcessingStep, TrainingStep
from sagemaker.workflow.conditions import ConditionGreaterThan
from sagemaker.workflow.condition_step import ConditionStep
from sagemaker.workflow.functions import JsonGet

# Data validation step
validation_step = ProcessingStep(
    name='ValidateData',
    processor=validation_processor,
    code='validate_data.py',
    inputs=[...],
    outputs=[...],
    property_files=[validation_report]
)

# Training step
training_step = TrainingStep(
    name='TrainModel',
    estimator=estimator,
    inputs={'train': train_data}
)

# Evaluation step
evaluation_step = ProcessingStep(
    name='EvaluateModel',
    processor=evaluation_processor,
    code='evaluate.py',
    inputs=[...],
    outputs=[...],
    property_files=[evaluation_report]
)

# Condition: Deploy only if better than current model
condition = ConditionGreaterThan(
    left=JsonGet(
        step_name=evaluation_step.name,
        property_file=evaluation_report,
        json_path='metrics.accuracy'
    ),
    right=0.92  # Current model accuracy
)

# Conditional deployment
condition_step = ConditionStep(
    name='CheckPerformance',
    conditions=[condition],
    if_steps=[register_step, deploy_step],
    else_steps=[notify_failure_step]
)

# Create pipeline
retraining_pipeline = Pipeline(
    name='automated-retraining-pipeline',
    steps=[validation_step, training_step, evaluation_step, condition_step]
)
```

### Model Versioning

| Aspect | Description |
|--------|-------------|
| **Version Number** | Semantic versioning (e.g., v1.2.3) |
| **Metadata** | Training date, data version, hyperparameters |
| **Performance Metrics** | Accuracy, latency, resource usage |
| **Approval Status** | Pending, Approved, Rejected |
| **Lineage** | Data, code, and model lineage |

### Champion/Challenger Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                  CHAMPION/CHALLENGER PATTERN                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Production Endpoint                                           │
│   ┌────────────────────────────────────────────────────────┐    │
│   │                                                         │    │
│   │   ┌──────────────┐                  ┌──────────────┐   │    │
│   │   │  Champion    │  90% traffic     │ Challenger   │   │    │
│   │   │  (Current)   │◀─────────────────│    (New)     │   │    │
│   │   │   Model v2   │                  │   Model v3   │   │    │
│   │   └──────────────┘                  └──────────────┘   │    │
│   │         │                                   │           │    │
│   │         └───────────────┬───────────────────┘           │    │
│   │                         │                               │    │
│   └─────────────────────────┼───────────────────────────────┘    │
│                             ▼                                    │
│                    Compare Performance                           │
│                             │                                    │
│                   If Challenger better:                          │
│                   • Promote to Champion                          │
│                   • Increase traffic gradually                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```python
# Deploy champion/challenger
endpoint_config = sagemaker_client.create_endpoint_config(
    EndpointConfigName='champion-challenger-config',
    ProductionVariants=[
        {
            'VariantName': 'Champion',
            'ModelName': 'model-v2',
            'InstanceType': 'ml.m5.xlarge',
            'InitialInstanceCount': 3,
            'InitialVariantWeight': 9
        },
        {
            'VariantName': 'Challenger',
            'ModelName': 'model-v3',
            'InstanceType': 'ml.m5.xlarge',
            'InitialInstanceCount': 1,
            'InitialVariantWeight': 1
        }
    ]
)
```

> **Exam Tip**: Use champion/challenger pattern to validate new models in production before full deployment. Start with small traffic percentage (5-10%) to challenger.
{: .prompt-tip }

---

## Task 4.4: Secure ML Solutions and Ensure Compliance

### Security Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                      ML SECURITY LAYERS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────┐       │
│   │  Data Security                                      │       │
│   │  • Encryption at rest (S3, EBS)                     │       │
│   │  • Encryption in transit (TLS/SSL)                  │       │
│   │  • Data classification and tagging                  │       │
│   └─────────────────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────────────────┐       │
│   │  Network Security                                   │       │
│   │  • VPC isolation                                    │       │
│   │  • Private subnets                                  │       │
│   │  • Security groups and NACLs                        │       │
│   │  • VPC endpoints (PrivateLink)                      │       │
│   └─────────────────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────────────────┐       │
│   │  Access Control                                     │       │
│   │  • IAM roles and policies                           │       │
│   │  • Resource-based policies                          │       │
│   │  • MFA and temporary credentials                    │       │
│   └─────────────────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────────────────┐       │
│   │  Monitoring & Compliance                            │       │
│   │  • CloudTrail logging                               │       │
│   │  • Config rules                                     │       │
│   │  • GuardDuty threat detection                       │       │
│   └─────────────────────────────────────────────────────┘       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### IAM for SageMaker

**Execution Role** - Role assumed by SageMaker to access resources.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
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
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer"
      ],
      "Resource": "*"
    }
  ]
}
```

**User/Role Policies** - Control who can use SageMaker.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "sagemaker:CreateTrainingJob",
        "sagemaker:CreateModel",
        "sagemaker:CreateEndpoint",
        "sagemaker:DescribeTrainingJob",
        "sagemaker:DescribeEndpoint"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "sagemaker:DeleteEndpoint",
        "sagemaker:DeleteModel"
      ],
      "Resource": "*"
    }
  ]
}
```

### IAM Best Practices

| Practice | Description |
|----------|-------------|
| **Least Privilege** | Grant minimum permissions needed |
| **Separate Roles** | Different roles for training, deployment, monitoring |
| **Resource Tags** | Use tags for fine-grained access control |
| **Condition Keys** | Add conditions (IP, time, MFA) to policies |
| **Regular Audits** | Review and rotate permissions |
| **Service Control Policies** | Organization-wide guardrails |

### Network Isolation with VPC

```
┌─────────────────────────────────────────────────────────────────┐
│                    VPC ISOLATION ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   VPC (10.0.0.0/16)                                             │
│   ┌────────────────────────────────────────────────────────┐    │
│   │                                                         │    │
│   │  Private Subnet (10.0.1.0/24)                          │    │
│   │  ┌──────────────────────────────────────────────────┐  │    │
│   │  │  SageMaker Training Job                          │  │    │
│   │  │  • No internet access                            │  │    │
│   │  │  • Access S3 via VPC endpoint                    │  │    │
│   │  └──────────────────────────────────────────────────┘  │    │
│   │                                                         │    │
│   │  Private Subnet (10.0.2.0/24)                          │    │
│   │  ┌──────────────────────────────────────────────────┐  │    │
│   │  │  SageMaker Endpoint                              │  │    │
│   │  │  • Isolated from internet                        │  │    │
│   │  └──────────────────────────────────────────────────┘  │    │
│   │                                                         │    │
│   │  VPC Endpoints                                         │    │
│   │  • S3 (gateway endpoint)                               │    │
│   │  • SageMaker Runtime (interface endpoint)              │    │
│   │  • SageMaker API (interface endpoint)                  │    │
│   │  • CloudWatch Logs (interface endpoint)                │    │
│   │                                                         │    │
│   └────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**VPC Configuration:**

```python
# Training job with VPC
estimator = XGBoost(
    ...,
    subnets=['subnet-12345', 'subnet-67890'],
    security_group_ids=['sg-12345'],
    enable_network_isolation=False  # Set True for complete isolation
)

# Endpoint with VPC
predictor = model.deploy(
    ...,
    vpc_config={
        'Subnets': ['subnet-12345', 'subnet-67890'],
        'SecurityGroupIds': ['sg-12345']
    }
)
```

**Network Isolation Mode:**
- Complete isolation from internet
- No inbound/outbound network calls
- Training data and model artifacts must be in S3

### Encryption

| Type | Service | Method |
|------|---------|--------|
| **At Rest** | S3 | SSE-S3, SSE-KMS, SSE-C |
| **At Rest** | EBS | KMS encryption |
| **In Transit** | All | TLS 1.2+ |
| **Training** | SageMaker | Inter-container encryption |

```python
# Training with encryption
estimator = XGBoost(
    ...,
    encrypt_inter_container_traffic=True,
    volume_kms_key='arn:aws:kms:us-east-1:123456789012:key/12345678',
    output_kms_key='arn:aws:kms:us-east-1:123456789012:key/12345678'
)

# S3 encryption
s3_client.put_object(
    Bucket='my-bucket',
    Key='model.tar.gz',
    Body=model_data,
    ServerSideEncryption='aws:kms',
    SSEKMSKeyId='arn:aws:kms:us-east-1:123456789012:key/12345678'
)
```

### Compliance and Governance

| Framework | Description | AWS Services |
|-----------|-------------|--------------|
| **HIPAA** | Healthcare data protection | Eligible SageMaker services |
| **PCI DSS** | Payment card data security | Shared responsibility |
| **SOC** | Security and availability controls | AWS compliance reports |
| **GDPR** | EU data protection | Data residency, encryption |
| **FedRAMP** | US government standards | GovCloud regions |

### Data Protection Strategies

| Strategy | Description | Implementation |
|----------|-------------|----------------|
| **Data Masking** | Hide sensitive data | Pre-processing |
| **Tokenization** | Replace sensitive data with tokens | External tokenization service |
| **Anonymization** | Remove PII | Feature engineering |
| **Access Logging** | Track data access | CloudTrail, S3 access logs |
| **Data Retention** | Automated deletion | S3 lifecycle policies |

### SageMaker Security Features

| Feature | Description |
|---------|-------------|
| **VPC Mode** | Run in isolated VPC |
| **Network Isolation** | No network access during training |
| **Encryption** | At-rest and in-transit encryption |
| **IAM Integration** | Fine-grained access control |
| **Notebook Instance Lifecycle** | Pre-signed URLs, session duration limits |
| **Model Package Signing** | Verify model integrity |

### Audit and Compliance Monitoring

```python
# CloudTrail event for SageMaker
{
    "eventName": "CreateEndpoint",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDACKCEVSQ6C2EXAMPLE",
        "arn": "arn:aws:iam::123456789012:user/Alice"
    },
    "eventTime": "2024-01-15T14:30:00Z",
    "requestParameters": {
        "endpointName": "fraud-detection-endpoint",
        "endpointConfigName": "fraud-detection-config"
    },
    "responseElements": {
        "endpointArn": "arn:aws:sagemaker:us-east-1:123456789012:endpoint/fraud-detection-endpoint"
    }
}
```

**AWS Config Rules for SageMaker:**
- `sagemaker-endpoint-config-kms-key-configured`
- `sagemaker-notebook-no-direct-internet-access`
- `sagemaker-endpoint-in-vpc`

> **Exam Tip**: For highly sensitive data, use VPC mode with network isolation and KMS encryption. Always enable inter-container traffic encryption for distributed training.
{: .prompt-warning }

---

## Key Definitions

| Term | Definition |
|------|------------|
| **Data Drift** | Change in input feature distribution over time |
| **Concept Drift** | Change in relationship between features and target |
| **Model Degradation** | Decrease in model performance over time |
| **Ground Truth** | Actual labels for predictions, used to evaluate model quality |
| **Baseline** | Reference statistics from training data for comparison |
| **VPC Endpoint** | Private connection to AWS services without internet gateway |
| **Network Isolation** | Complete network isolation during training |
| **Least Privilege** | Grant minimum permissions necessary for a task |

---

## Practice Questions

### Question 1
Which type of monitoring should be configured to detect if the distribution of input features has changed significantly from training data?

A) Model Quality Monitoring
B) Data Quality Monitoring
C) Bias Drift Monitoring
D) Feature Attribution Monitoring

<details>
<summary>View Answer</summary>

**Answer: B**

Data Quality Monitoring compares production input data against baseline statistics from training data. It detects:
- Changes in feature distributions
- Missing values
- Data type changes
- Statistical property changes (mean, std, etc.)

Model Quality (A) monitors prediction quality. Bias Drift (C) monitors fairness metrics. Feature Attribution (D) monitors feature importance.

</details>

### Question 2
A production model's accuracy has dropped from 95% to 82% over the past month, but input feature distributions remain similar to training data. What type of drift is this?

A) Data drift
B) Label drift
C) Concept drift
D) Infrastructure drift

<details>
<summary>View Answer</summary>

**Answer: C**

Concept drift occurs when the relationship between features and the target variable changes, even if feature distributions stay the same. This causes model performance degradation.

Data drift (A) would show changes in input distributions. Label drift (B) is a change in target variable distribution. Infrastructure drift (D) is not a standard term.

</details>

### Question 3
Which SageMaker feature enables running training jobs in complete network isolation without any internet access?

A) VPC configuration
B) Security groups
C) Network isolation mode
D) Private subnets

<details>
<summary>View Answer</summary>

**Answer: C**

Network isolation mode (`enable_network_isolation=True`) provides complete isolation with no inbound or outbound network calls. Training data and model artifacts must be in S3.

VPC configuration (A) provides isolation but doesn't prevent all network access. Security groups (B) control traffic but don't provide complete isolation. Private subnets (D) are part of VPC config.

</details>

### Question 4
An ML engineer wants to test a new model version in production with 5% of traffic before full deployment. Which strategy should they use?

A) Blue/Green deployment
B) All-at-once deployment
C) Canary deployment
D) Rolling deployment

<details>
<summary>View Answer</summary>

**Answer: C**

Canary deployment gradually shifts traffic to a new version, starting with a small percentage (e.g., 5%). This allows validation with real traffic before full rollout.

Blue/Green (A) switches all traffic at once. All-at-once (B) replaces immediately. Rolling (D) updates instances sequentially.

</details>

### Question 5
Which IAM best practice should be followed when granting permissions to a SageMaker execution role?

A) Grant full administrator access
B) Use the principle of least privilege
C) Grant all SageMaker permissions
D) Use root account credentials

<details>
<summary>View Answer</summary>

**Answer: B**

The principle of least privilege means granting only the minimum permissions needed for the specific task. This reduces security risk and follows AWS best practices.

Full admin access (A) violates least privilege. All SageMaker permissions (C) is excessive. Root credentials (D) should never be used for services.

</details>

### Question 6
What is the recommended approach for handling highly sensitive healthcare data during ML model training on SageMaker?

A) Use default SageMaker configuration
B) Enable VPC mode with network isolation and KMS encryption
C) Store data unencrypted in S3
D) Use public endpoints for faster access

<details>
<summary>View Answer</summary>

**Answer: B**

For sensitive healthcare data (HIPAA compliance):
- VPC mode isolates resources
- Network isolation prevents data exfiltration
- KMS encryption protects data at rest and in transit

Default config (A) lacks security controls. Unencrypted storage (C) violates compliance. Public endpoints (D) increase attack surface.

</details>

---

## Key Takeaways

1. **Four types of Model Monitor**: Data Quality, Model Quality, Bias Drift, Feature Attribution
2. **Data drift vs concept drift**: Data drift is input distribution change, concept drift is relationship change
3. **Enable data capture**: Required for Model Monitor to function
4. **Baseline from training data**: Create statistical baseline for comparison
5. **CloudWatch metrics**: Monitor Invocations, ModelLatency, Errors, CPU/Memory
6. **Automated retraining**: Trigger on drift detection or scheduled intervals
7. **Champion/Challenger pattern**: Test new models with small traffic percentage
8. **VPC with network isolation**: Maximum security for sensitive data
9. **Encrypt everything**: At-rest (KMS), in-transit (TLS), inter-container
10. **Least privilege IAM**: Grant minimum necessary permissions
11. **CloudTrail for audit**: Track all SageMaker API calls
12. **Model versioning**: Track metrics, metadata, and approval status

---

## Navigation

| Previous | Next |
|----------|------|
| [Domain 3: Deployment and Orchestration](/posts/aws-mla-c01-domain-3-deployment-orchestration/) | [Practice Exam](/posts/aws-mla-c01-practice-exam/) |
