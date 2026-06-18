---
title: "AWS MLA-C01 - Domain 3: Deployment and Orchestration of ML Workflows"
author: thanhnv1808
date: 2026-01-24 12:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, deployment, mlops, sagemaker, pipelines, endpoints]
description: "Domain 3 covers 22% of the exam (~14 questions). Master model deployment patterns, SageMaker endpoints, inference optimization, MLOps, and workflow orchestration."
pin: false
comments: true
---

## Domain 3 Overview

**Exam Weight: 22% (~14 questions)**

This domain focuses on deploying ML models to production, implementing inference endpoints, orchestrating ML workflows, and establishing MLOps practices.

### Task Statements

| Task | Description |
|------|-------------|
| 3.1 | Deploy and serve ML models for inference |
| 3.2 | Implement inference optimization techniques |
| 3.3 | Build and orchestrate ML workflows and pipelines |
| 3.4 | Apply MLOps practices for model lifecycle management |

---

## Task 3.1: Deploy and Serve ML Models for Inference

### SageMaker Deployment Options

```
┌─────────────────────────────────────────────────────────────────┐
│                SAGEMAKER INFERENCE OPTIONS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Real-time Inference        Batch Transform      Async Inference│
│   ┌─────────────────┐        ┌──────────────┐    ┌────────────┐ │
│   │ Hosted Endpoint │        │ Batch Job    │    │ Async      │ │
│   │ • Low latency   │        │ • Large data │    │ Endpoint   │ │
│   │ • Always on     │        │ • No endpoint│    │ • Queue    │ │
│   │ • REST API      │        │ • Cost-eff   │    │ • Large    │ │
│   └─────────────────┘        └──────────────┘    │   payload  │ │
│                                                   └────────────┘ │
│                                                                  │
│   Serverless Inference       Edge Deployment                    │
│   ┌─────────────────┐        ┌──────────────┐                   │
│   │ Auto-scale      │        │ IoT Devices  │                   │
│   │ Pay-per-use     │        │ Edge runtime │                   │
│   │ No idle cost    │        │ Offline      │                   │
│   └─────────────────┘        └──────────────┘                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Deployment Pattern Comparison

| Pattern | Latency | Cost | Use Case |
|---------|---------|------|----------|
| **Real-time Endpoint** | Milliseconds | High (always on) | Interactive applications, web/mobile apps |
| **Serverless Inference** | Sub-second | Low (pay-per-use) | Intermittent traffic, variable load |
| **Batch Transform** | Minutes/Hours | Low | Large datasets, periodic processing |
| **Async Inference** | Seconds/Minutes | Medium | Large payloads, near real-time |
| **Edge Deployment** | Milliseconds | Device-dependent | Offline, low latency, IoT |

### Real-time Inference Endpoints

```
┌─────────────────────────────────────────────────────────────────┐
│                  REAL-TIME ENDPOINT ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Client Application                                            │
│   ┌─────────────────────┐                                       │
│   │  POST /invoke       │                                       │
│   │  {data: [...]}      │                                       │
│   └──────────┬──────────┘                                       │
│              │                                                   │
│              ▼                                                   │
│   ┌─────────────────────┐                                       │
│   │  SageMaker Endpoint │                                       │
│   │  • Load balancing   │                                       │
│   │  • Auto-scaling     │                                       │
│   └──────────┬──────────┘                                       │
│              │                                                   │
│     ┌────────┼────────┬────────┐                                │
│     ▼        ▼        ▼        ▼                                │
│   ┌────┐  ┌────┐  ┌────┐  ┌────┐                               │
│   │Inst│  │Inst│  │Inst│  │Inst│                               │
│   │ 1  │  │ 2  │  │ 3  │  │ 4  │                               │
│   └────┘  └────┘  └────┘  └────┘                               │
│                                                                  │
│   Model artifacts loaded from S3 at deployment                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Creating an Endpoint:**

```python
from sagemaker.model import Model
from sagemaker.predictor import Predictor

# Create model
model = Model(
    image_uri=training_image,
    model_data='s3://bucket/model.tar.gz',
    role=role
)

# Deploy endpoint
predictor = model.deploy(
    initial_instance_count=2,
    instance_type='ml.m5.xlarge',
    endpoint_name='customer-churn-endpoint'
)

# Make prediction
result = predictor.predict(data)
```

### Serverless Inference

**Serverless Inference** automatically scales from zero to handle traffic spikes.

| Feature | Description |
|---------|-------------|
| **Auto-scaling** | Scales to zero when idle |
| **Pricing** | Pay only for compute time and data processed |
| **Cold Start** | Initial requests may have higher latency |
| **Concurrency** | Configurable max concurrent invocations |
| **Memory** | 1GB to 6GB configurable |

```python
from sagemaker.serverless import ServerlessInferenceConfig

serverless_config = ServerlessInferenceConfig(
    memory_size_in_mb=4096,
    max_concurrency=10
)

predictor = model.deploy(
    serverless_inference_config=serverless_config,
    endpoint_name='serverless-endpoint'
)
```

**When to use:**
- Intermittent or unpredictable traffic
- Cost optimization for low-traffic applications
- Development and testing environments

> **Exam Tip**: Serverless inference is ideal for intermittent traffic and scales to zero to minimize costs. Real-time endpoints are for consistent low-latency needs.
{: .prompt-tip }

### Batch Transform

**Batch Transform** performs inference on large datasets without deploying an endpoint.

```
┌─────────────────────────────────────────────────────────────────┐
│                     BATCH TRANSFORM WORKFLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐                                              │
│   │ Input Data   │                                              │
│   │   (S3)       │                                              │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐      ┌──────────────┐                        │
│   │  Transform   │─────▶│  Instances   │                        │
│   │     Job      │      │  (temporary) │                        │
│   └──────────────┘      └──────┬───────┘                        │
│                                │                                 │
│                                ▼                                 │
│                         ┌──────────────┐                         │
│                         │ Predictions  │                         │
│                         │    (S3)      │                         │
│                         └──────────────┘                         │
│                                                                  │
│   • No endpoint deployment                                      │
│   • Instances terminated after job completes                    │
│   • Cost-effective for large batch processing                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sagemaker.transformer import Transformer

transformer = Transformer(
    model_name='my-model',
    instance_count=5,
    instance_type='ml.m5.xlarge',
    output_path='s3://bucket/predictions/'
)

transformer.transform(
    data='s3://bucket/input-data/',
    content_type='text/csv',
    split_type='Line',
    join_source='Input'
)
```

**Key Parameters:**

| Parameter | Description |
|-----------|-------------|
| `split_type` | How to split input (Line, RecordIO, None) |
| `join_source` | Join predictions with input (Input, None) |
| `strategy` | SingleRecord or MultiRecord |
| `max_payload` | Maximum payload size (MB) |
| `max_concurrent_transforms` | Parallel requests per instance |

### Asynchronous Inference

**Async Inference** queues requests and processes them asynchronously.

```
┌─────────────────────────────────────────────────────────────────┐
│                  ASYNCHRONOUS INFERENCE FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Client                    Queue              Endpoint          │
│   ┌────────┐              ┌───────┐          ┌──────────┐       │
│   │Request │─────────────▶│ SQS   │─────────▶│Processing│       │
│   │(S3 URI)│              │ Queue │          │Instances │       │
│   └────────┘              └───────┘          └────┬─────┘       │
│      │                                            │              │
│      │                                            ▼              │
│      │                                       ┌──────────┐        │
│      │                                       │ Results  │        │
│      └──────────────────────────────────────▶│  (S3)    │        │
│         Check status / SNS notification      └──────────┘        │
│                                                                  │
│   Features:                                                      │
│   • Large payloads (up to 1GB)                                  │
│   • Auto-scaling based on queue depth                           │
│   • SNS notifications on completion                             │
│   • Request/response stored in S3                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sagemaker.async_inference import AsyncInferenceConfig

async_config = AsyncInferenceConfig(
    output_path='s3://bucket/async-outputs/',
    notification_config={
        'SuccessTopic': 'arn:aws:sns:us-east-1:123456789012:success',
        'ErrorTopic': 'arn:aws:sns:us-east-1:123456789012:error'
    }
)

predictor = model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    async_inference_config=async_config
)
```

### Multi-Model Endpoints

**Multi-Model Endpoints** host multiple models on the same endpoint to reduce costs.

```
┌─────────────────────────────────────────────────────────────────┐
│                   MULTI-MODEL ENDPOINT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌────────────────────────────────────────────────────────┐    │
│   │  Endpoint (ml.m5.2xlarge)                              │    │
│   │                                                         │    │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │    │
│   │  │ Model A │  │ Model B │  │ Model C │  │ Model D │   │    │
│   │  │(loaded) │  │(loaded) │  │ (disk)  │  │ (disk)  │   │    │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │    │
│   │                                                         │    │
│   │  Dynamic loading from S3 based on TargetModel          │    │
│   └────────────────────────────────────────────────────────┘    │
│                                                                  │
│   Request: {TargetModel: "modelA.tar.gz", Data: [...]}         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Benefits:**
- Cost optimization (one endpoint for multiple models)
- Improved endpoint utilization
- Dynamically load/unload models
- Up to thousands of models per endpoint

**Limitations:**
- Models must use the same framework/container
- Shared compute resources
- Cold start latency for unloaded models

```python
from sagemaker.multidatamodel import MultiDataModel

mdm = MultiDataModel(
    name='multi-model',
    model_data_prefix='s3://bucket/models/',
    image_uri=container_image,
    role=role
)

predictor = mdm.deploy(
    initial_instance_count=2,
    instance_type='ml.m5.xlarge'
)

# Invoke specific model
predictor.predict(data, target_model='modelA.tar.gz')
```

> **Exam Tip**: Multi-model endpoints are cost-effective when hosting many models with similar inference requirements and intermittent traffic per model.
{: .prompt-tip }

### Edge Deployment with SageMaker Edge Manager

Deploy models to edge devices (IoT, mobile, embedded systems).

| Component | Description |
|-----------|-------------|
| **SageMaker Neo** | Optimizes models for edge hardware |
| **Edge Manager** | Manages fleet of edge devices |
| **Edge Agent** | Runs on device, loads models, collects data |
| **Device Fleet** | Logical grouping of edge devices |

**Workflow:**
1. Compile model with SageMaker Neo
2. Package model for edge deployment
3. Deploy to device fleet via Edge Manager
4. Monitor and manage devices

---

## Task 3.2: Implement Inference Optimization Techniques

### Model Optimization Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                  MODEL OPTIMIZATION TECHNIQUES                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │ Compilation │   │Quantization │   │  Pruning    │          │
│   │(SageMaker   │   │(Reduce      │   │(Remove      │          │
│   │   Neo)      │   │ precision)  │   │ weights)    │          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │ Distillation│   │   Caching   │   │ Batching    │          │
│   │(Smaller     │   │(Store       │   │(Group       │          │
│   │  model)     │   │ results)    │   │ requests)   │          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SageMaker Neo

**SageMaker Neo** compiles models for optimized inference on specific hardware.

| Feature | Description |
|---------|-------------|
| **Framework Support** | TensorFlow, PyTorch, MXNet, ONNX, XGBoost |
| **Target Platforms** | Cloud instances, edge devices (ARM, Intel, Nvidia) |
| **Optimization** | Model-specific optimizations for hardware |
| **Performance** | Up to 2x faster inference |

```python
from sagemaker.neo import Neo

# Compile model
compiled_model = model.compile(
    target_instance_family='ml_c5',
    input_shape={'data': [1, 3, 224, 224]},
    framework='pytorch',
    framework_version='1.8',
    role=role,
    job_name='neo-compilation-job'
)

# Deploy compiled model
predictor = compiled_model.deploy(
    initial_instance_count=1,
    instance_type='ml.c5.xlarge'
)
```

### Elastic Inference

**Elastic Inference (EI)** attaches GPU acceleration to CPU instances for deep learning inference.

| Instance Type | Description | Cost |
|---------------|-------------|------|
| **ml.eia2.medium** | 1GB GPU memory | Low |
| **ml.eia2.large** | 2GB GPU memory | Medium |
| **ml.eia2.xlarge** | 4GB GPU memory | High |

```python
predictor = model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    accelerator_type='ml.eia2.medium'
)
```

**When to use:**
- Deep learning models on CPU instances
- Cost optimization vs full GPU instances
- Moderate throughput requirements

> **Exam Tip**: Elastic Inference is being deprecated. For new deployments, consider GPU instances or AWS Inferentia (Inf1/Inf2 instances).
{: .prompt-warning }

### Inference Recommender

**SageMaker Inference Recommender** automatically benchmarks models and recommends optimal configurations.

| Recommendation Type | Description | Duration |
|---------------------|-------------|----------|
| **Instance Recommendations** | Find best instance type | ~45 minutes |
| **Endpoint Recommendations** | Optimize configuration (instance count, scaling) | ~2 hours |

```python
from sagemaker.model import Model

model.right_size(
    sample_payload_url='s3://bucket/sample-payload.json',
    supported_content_types=['application/json'],
    supported_instance_types=['ml.m5.xlarge', 'ml.c5.2xlarge', 'ml.g4dn.xlarge']
)
```

**Output:**
- Cost per hour
- Throughput (transactions per second)
- Latency (p50, p90, p99)
- Recommendations ranked by cost-performance

### Auto-scaling

**Auto-scaling** dynamically adjusts endpoint instance count based on metrics.

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENDPOINT AUTO-SCALING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Traffic Pattern          Instance Count                       │
│                                                                  │
│   High  ────────                     ┌─────────┐                │
│         │      │                     │ Scale   │                │
│   Med   │      └────────             │  Up/    │                │
│         │              │             │  Down   │                │
│   Low   └──────────────┴────         └─────────┘                │
│                                           │                      │
│   Time: 00  06  12  18  24         Min: 1, Max: 10             │
│                                                                  │
│   Scaling Policies:                                             │
│   • Target Tracking (e.g., keep InvocationsPerInstance < 1000) │
│   • Step Scaling (e.g., +2 instances if > 2000 invocations)    │
│   • Scheduled Scaling (e.g., scale up at 8 AM)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```python
import boto3

client = boto3.client('application-autoscaling')

# Register scalable target
client.register_scalable_target(
    ServiceNamespace='sagemaker',
    ResourceId='endpoint/my-endpoint/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    MinCapacity=1,
    MaxCapacity=10
)

# Define target tracking policy
client.put_scaling_policy(
    PolicyName='invocation-scaling-policy',
    ServiceNamespace='sagemaker',
    ResourceId='endpoint/my-endpoint/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    PolicyType='TargetTrackingScaling',
    TargetTrackingScalingPolicyConfiguration={
        'TargetValue': 1000.0,
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'SageMakerVariantInvocationsPerInstance'
        },
        'ScaleInCooldown': 300,
        'ScaleOutCooldown': 60
    }
)
```

**Scaling Metrics:**
- `InvocationsPerInstance`
- `ModelLatency`
- `CPUUtilization`
- `MemoryUtilization`

### Inference Pipeline

**Inference Pipelines** chain multiple containers for pre-processing and inference.

```
┌─────────────────────────────────────────────────────────────────┐
│                    INFERENCE PIPELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Client Request                                                │
│         │                                                        │
│         ▼                                                        │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐    │
│   │  Container  │─────▶│  Container  │─────▶│  Container  │    │
│   │     1       │      │      2      │      │      3      │    │
│   │ SparkML     │      │  Scikit-    │      │  XGBoost    │    │
│   │ Feature Eng │      │   Learn     │      │ Inference   │    │
│   └─────────────┘      └─────────────┘      └─────────────┘    │
│                                                   │              │
│                                                   ▼              │
│                                             Response             │
│                                                                  │
│   Up to 15 containers in sequence                               │
│   Real-time or batch transform                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sagemaker.pipeline import PipelineModel

pipeline_model = PipelineModel(
    name='inference-pipeline',
    role=role,
    models=[
        sparkml_model,
        sklearn_model,
        xgboost_model
    ]
)

predictor = pipeline_model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge'
)
```

---

## Task 3.3: Build and Orchestrate ML Workflows and Pipelines

### SageMaker Pipelines

**SageMaker Pipelines** is a native workflow orchestration service for ML.

```
┌─────────────────────────────────────────────────────────────────┐
│                   SAGEMAKER PIPELINE WORKFLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐                                              │
│   │   Data       │                                              │
│   │  Processing  │                                              │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐      ┌──────────────┐                        │
│   │   Training   │─────▶│  Evaluation  │                        │
│   └──────────────┘      └──────┬───────┘                        │
│                                │                                 │
│                                ▼                                 │
│                         ┌──────────────┐                         │
│                         │  Condition   │                         │
│                         │ (Accuracy>0.9)│                        │
│                         └──────┬───────┘                         │
│                          Yes   │   No                            │
│                    ┌───────────┴───────────┐                     │
│                    ▼                       ▼                     │
│              ┌──────────┐           ┌──────────┐                 │
│              │ Register │           │  Fail    │                 │
│              │  Model   │           │ Pipeline │                 │
│              └──────────┘           └──────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pipeline Components

| Component | Description | Use Case |
|-----------|-------------|----------|
| **Processing Step** | Run data processing jobs | Feature engineering, validation |
| **Training Step** | Train ML models | Model training |
| **Tuning Step** | Hyperparameter optimization | Model tuning |
| **Model Step** | Create/register models | Model registry |
| **Condition Step** | Conditional execution | Model approval |
| **Transform Step** | Batch inference | Batch predictions |
| **Callback Step** | External integration | Manual approval, webhooks |
| **Lambda Step** | Custom logic | Lightweight processing |

### Creating a Pipeline

```python
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import ProcessingStep, TrainingStep, CreateModelStep
from sagemaker.workflow.conditions import ConditionGreaterThanOrEqualTo
from sagemaker.workflow.condition_step import ConditionStep
from sagemaker.workflow.functions import JsonGet

# Processing step
processing_step = ProcessingStep(
    name='PreprocessData',
    processor=sklearn_processor,
    inputs=[...],
    outputs=[...],
    code='preprocess.py'
)

# Training step
training_step = TrainingStep(
    name='TrainModel',
    estimator=xgb_estimator,
    inputs={
        'train': processing_step.properties.ProcessingOutputConfig.Outputs['train'].S3Output.S3Uri
    }
)

# Evaluation step
evaluation_step = ProcessingStep(
    name='EvaluateModel',
    processor=evaluation_processor,
    inputs=[...],
    outputs=[...],
    code='evaluate.py',
    property_files=[evaluation_report]
)

# Condition step
condition = ConditionGreaterThanOrEqualTo(
    left=JsonGet(
        step_name=evaluation_step.name,
        property_file=evaluation_report,
        json_path='metrics.accuracy.value'
    ),
    right=0.9
)

condition_step = ConditionStep(
    name='CheckAccuracy',
    conditions=[condition],
    if_steps=[register_model_step],
    else_steps=[fail_step]
)

# Create pipeline
pipeline = Pipeline(
    name='ml-pipeline',
    parameters=[...],
    steps=[processing_step, training_step, evaluation_step, condition_step]
)

# Execute pipeline
pipeline.upsert(role_arn=role)
execution = pipeline.start()
```

### Pipeline Parameters

**Parameters** allow dynamic pipeline configuration.

```python
from sagemaker.workflow.parameters import (
    ParameterString,
    ParameterInteger,
    ParameterFloat
)

instance_type_param = ParameterString(
    name='TrainingInstanceType',
    default_value='ml.m5.xlarge'
)

instance_count_param = ParameterInteger(
    name='InstanceCount',
    default_value=1
)

# Use in pipeline execution
execution = pipeline.start(
    parameters={
        'TrainingInstanceType': 'ml.m5.2xlarge',
        'InstanceCount': 3
    }
)
```

### Step Functions vs SageMaker Pipelines

| Feature | SageMaker Pipelines | Step Functions |
|---------|---------------------|----------------|
| **Purpose** | ML-specific workflows | General workflow orchestration |
| **Integration** | Native SageMaker integration | AWS service integration |
| **UI** | SageMaker Studio | Step Functions console |
| **Lineage Tracking** | Built-in | Manual implementation |
| **Cost** | Free (pay for resources) | Pay per state transition |
| **Complexity** | Simple for ML | More flexible |

> **Exam Tip**: Use SageMaker Pipelines for ML workflows with native SageMaker integration. Use Step Functions for complex workflows requiring non-SageMaker AWS services.
{: .prompt-tip }

### AWS Step Functions for ML

```json
{
  "StartAt": "DataProcessing",
  "States": {
    "DataProcessing": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createProcessingJob.sync",
      "Parameters": {
        "ProcessingJobName.$": "$.ProcessingJobName",
        "RoleArn": "arn:aws:iam::123456789012:role/SageMakerRole",
        "ProcessingInputs": [...],
        "ProcessingOutputConfig": {...},
        "AppSpecification": {...}
      },
      "Next": "Training"
    },
    "Training": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createTrainingJob.sync",
      "Parameters": {...},
      "Next": "SaveModel"
    },
    "SaveModel": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sagemaker:createModel",
      "End": true
    }
  }
}
```

---

## Task 3.4: Apply MLOps Practices for Model Lifecycle Management

### MLOps Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                      MLOPS LIFECYCLE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│   │  Data   │───▶│  Model  │───▶│ Deploy  │───▶│ Monitor │     │
│   │  Prep   │    │  Build  │    │         │    │         │     │
│   └─────────┘    └─────────┘    └─────────┘    └────┬────┘     │
│        ▲                                              │          │
│        │                                              │          │
│        │         ┌─────────┐                          │          │
│        └─────────│ Retrain │◀─────────────────────────┘          │
│                  └─────────┘                                     │
│                                                                  │
│   Continuous Integration / Continuous Deployment (CI/CD)        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SageMaker Model Registry

**Model Registry** provides centralized model versioning and approval workflow.

| Feature | Description |
|---------|-------------|
| **Model Groups** | Logical collection of model versions |
| **Model Versions** | Individual trained models with metadata |
| **Approval Status** | PendingManualApproval, Approved, Rejected |
| **Metadata** | Metrics, hyperparameters, lineage |
| **Cross-Account** | Share models across accounts |

```python
from sagemaker.model import Model
from sagemaker.model_metrics import MetricsSource, ModelMetrics

# Define model metrics
model_metrics = ModelMetrics(
    model_statistics=MetricsSource(
        s3_uri='s3://bucket/evaluation-report.json',
        content_type='application/json'
    )
)

# Register model
model_package = model.register(
    model_package_group_name='fraud-detection-models',
    approval_status='PendingManualApproval',
    inference_instances=['ml.m5.xlarge', 'ml.m5.2xlarge'],
    transform_instances=['ml.m5.xlarge'],
    model_metrics=model_metrics,
    metadata={'training_date': '2024-01-15', 'framework': 'xgboost'}
)

# Approve model
model_package.update_approval_status(approval_status='Approved')

# Deploy approved model
model_package.deploy(
    initial_instance_count=2,
    instance_type='ml.m5.xlarge'
)
```

### CI/CD for ML

```
┌─────────────────────────────────────────────────────────────────┐
│                       CI/CD PIPELINE FOR ML                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Code Commit (CodeCommit)                                      │
│         │                                                        │
│         ▼                                                        │
│   Build (CodeBuild)                                             │
│   • Run tests                                                   │
│   • Build Docker image                                          │
│   • Push to ECR                                                 │
│         │                                                        │
│         ▼                                                        │
│   Deploy (CodePipeline + SageMaker Pipelines)                   │
│   • Trigger SageMaker Pipeline                                  │
│   • Train model                                                 │
│   • Evaluate model                                              │
│   • Register model if metrics meet threshold                    │
│         │                                                        │
│         ▼                                                        │
│   Deploy Endpoint (CodeDeploy / Lambda)                         │
│   • Blue/Green deployment                                       │
│   • Update endpoint configuration                               │
│         │                                                        │
│         ▼                                                        │
│   Monitor (CloudWatch / Model Monitor)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Model Deployment Strategies

| Strategy | Description | Downtime | Risk |
|----------|-------------|----------|------|
| **All-at-once** | Replace all instances | Yes | High |
| **Blue/Green** | Deploy to new environment, switch traffic | No | Low |
| **Canary** | Gradually shift traffic to new version | No | Low |
| **A/B Testing** | Split traffic between versions | No | Low |

### Blue/Green Deployment

```python
# Create new endpoint configuration
new_endpoint_config = sagemaker_client.create_endpoint_config(
    EndpointConfigName='my-endpoint-config-v2',
    ProductionVariants=[{
        'VariantName': 'AllTraffic',
        'ModelName': 'new-model',
        'InstanceType': 'ml.m5.xlarge',
        'InitialInstanceCount': 2
    }]
)

# Update endpoint (blue/green deployment)
sagemaker_client.update_endpoint(
    EndpointName='my-endpoint',
    EndpointConfigName='my-endpoint-config-v2',
    RetainAllVariantProperties=False
)
```

### Canary Deployment

```python
# Create endpoint config with two variants
endpoint_config = sagemaker_client.create_endpoint_config(
    EndpointConfigName='canary-config',
    ProductionVariants=[
        {
            'VariantName': 'Stable',
            'ModelName': 'stable-model',
            'InstanceType': 'ml.m5.xlarge',
            'InitialInstanceCount': 2,
            'InitialVariantWeight': 9
        },
        {
            'VariantName': 'Canary',
            'ModelName': 'new-model',
            'InstanceType': 'ml.m5.xlarge',
            'InitialInstanceCount': 1,
            'InitialVariantWeight': 1
        }
    ]
)

# Gradually increase canary weight
sagemaker_client.update_endpoint_weights_and_capacities(
    EndpointName='my-endpoint',
    DesiredWeightsAndCapacities=[
        {'VariantName': 'Stable', 'DesiredWeight': 5},
        {'VariantName': 'Canary', 'DesiredWeight': 5}
    ]
)
```

### SageMaker Projects

**SageMaker Projects** provide MLOps templates with CI/CD.

| Template | Description |
|----------|-------------|
| **Model Building, Training, Deployment** | End-to-end pipeline with automated deployment |
| **Model Building and Training** | Pipeline for training without deployment |
| **Model Deployment** | Deployment-only pipeline |

**Components:**
- AWS CodePipeline for orchestration
- AWS CodeCommit for source control
- AWS CodeBuild for building
- SageMaker Pipelines for ML workflows
- Model Registry for versioning

```python
import sagemaker

sagemaker_client = sagemaker.Session().sagemaker_client

# Create project from template
project = sagemaker_client.create_project(
    ProjectName='fraud-detection-project',
    ServiceCatalogProvisioningDetails={
        'ProductId': 'prod-xxxxxxxx',
        'ProvisioningArtifactId': 'pa-xxxxxxxx'
    }
)
```

### Infrastructure as Code

**CloudFormation / CDK** for reproducible ML infrastructure.

```python
from aws_cdk import aws_sagemaker as sagemaker
from aws_cdk import Stack

class MLStack(Stack):
    def __init__(self, scope, id, **kwargs):
        super().__init__(scope, id, **kwargs)

        # Create model
        model = sagemaker.CfnModel(
            self, 'Model',
            execution_role_arn=role_arn,
            primary_container={
                'image': training_image,
                'modelDataUrl': model_artifacts
            }
        )

        # Create endpoint config
        endpoint_config = sagemaker.CfnEndpointConfig(
            self, 'EndpointConfig',
            production_variants=[{
                'variantName': 'AllTraffic',
                'modelName': model.attr_model_name,
                'instanceType': 'ml.m5.xlarge',
                'initialInstanceCount': 2
            }]
        )

        # Create endpoint
        endpoint = sagemaker.CfnEndpoint(
            self, 'Endpoint',
            endpoint_config_name=endpoint_config.attr_endpoint_config_name
        )
```

> **Exam Tip**: SageMaker Projects provide pre-built MLOps templates with CI/CD. Use for production ML workflows requiring automated deployment and governance.
{: .prompt-tip }

---

## Key Definitions

| Term | Definition |
|------|------------|
| **Endpoint** | Hosted model that provides real-time inference via API |
| **Production Variant** | A version of a model deployed to an endpoint |
| **Inference Pipeline** | Chain of containers for preprocessing and inference |
| **Multi-Model Endpoint** | Single endpoint hosting multiple models |
| **Model Registry** | Centralized repository for model versioning and metadata |
| **Blue/Green Deployment** | Deploy new version alongside old, then switch traffic |
| **Canary Deployment** | Gradually shift traffic from old to new version |
| **MLOps** | Practices for automating and improving ML lifecycle |

---

## Practice Questions

### Question 1
A company needs to perform inference on 10 million records once per day. The inference should complete within 2 hours and minimize costs. Which deployment option is most appropriate?

A) Real-time endpoint with auto-scaling
B) Serverless inference
C) Batch Transform
D) Asynchronous inference

<details>
<summary>View Answer</summary>

**Answer: C**

Batch Transform is ideal for large-scale, periodic inference jobs. It:
- Processes large datasets efficiently
- No endpoint to keep running (lower cost)
- Scales automatically for the job
- Perfect for daily batch processing

Real-time endpoints (A) would incur costs when idle. Serverless (B) and async (D) are designed for request/response patterns, not large batch jobs.

</details>

### Question 2
An ML engineer wants to host 500 similar models for different customers, each serving intermittent traffic. What is the most cost-effective deployment strategy?

A) Deploy 500 separate real-time endpoints
B) Use a multi-model endpoint
C) Use serverless inference for each model
D) Use batch transform for all models

<details>
<summary>View Answer</summary>

**Answer: B**

Multi-model endpoints allow hosting multiple models on shared infrastructure, significantly reducing costs when:
- Models have similar resource requirements
- Traffic per model is intermittent
- Many models need to be hosted

Separate endpoints (A) would be very expensive. Serverless (C) would require separate endpoints. Batch transform (D) doesn't fit the use case.

</details>

### Question 3
A model deployment requires zero downtime and the ability to quickly rollback if issues occur. Which deployment strategy should be used?

A) All-at-once deployment
B) Blue/Green deployment
C) In-place update
D) Batch deployment

<details>
<summary>View Answer</summary>

**Answer: B**

Blue/Green deployment:
- Deploys new version alongside existing version
- Switches traffic only when new version is healthy
- Enables instant rollback by switching traffic back
- No downtime during deployment

All-at-once (A) causes downtime. In-place (C) has downtime and difficult rollback. Batch (D) is not a deployment strategy.

</details>

### Question 4
Which SageMaker feature automatically benchmarks different instance types and provides cost-performance recommendations for model deployment?

A) SageMaker Debugger
B) SageMaker Model Monitor
C) SageMaker Inference Recommender
D) SageMaker Clarify

<details>
<summary>View Answer</summary>

**Answer: C**

SageMaker Inference Recommender automatically:
- Benchmarks models on different instance types
- Measures latency, throughput, and cost
- Provides ranked recommendations

Debugger (A) is for training debugging. Model Monitor (B) detects drift. Clarify (D) is for bias detection and explainability.

</details>

### Question 5
A team wants to automate the ML workflow from data processing through model deployment with approval gates. Which AWS service is best suited?

A) AWS Step Functions
B) SageMaker Pipelines
C) AWS CodePipeline
D) AWS Lambda

<details>
<summary>View Answer</summary>

**Answer: B**

SageMaker Pipelines is purpose-built for ML workflows with:
- Native SageMaker integration
- Built-in steps for processing, training, evaluation
- Condition steps for approval gates
- Model registry integration
- Lineage tracking

Step Functions (A) requires more custom configuration. CodePipeline (C) is for general CI/CD. Lambda (D) is too low-level.

</details>

### Question 6
What is the primary benefit of using SageMaker Neo for model deployment?

A) Reduce training time
B) Optimize model for specific hardware
C) Automatically scale endpoints
D) Monitor model performance

<details>
<summary>View Answer</summary>

**Answer: B**

SageMaker Neo compiles models for optimized inference on specific hardware targets (CPUs, GPUs, edge devices), improving inference performance and reducing latency.

Neo doesn't affect training (A), scaling (C), or monitoring (D).

</details>

---

## Key Takeaways

1. **Choose the right inference option**: Real-time for low latency, Batch Transform for large datasets, Serverless for intermittent traffic
2. **Multi-model endpoints save costs**: When hosting many models with similar requirements
3. **Blue/Green for zero downtime**: Enables safe deployments with instant rollback
4. **SageMaker Pipelines for ML workflows**: Native integration with SageMaker services and lineage tracking
5. **Model Registry centralizes versioning**: Track model versions, metadata, and approval status
6. **Auto-scaling optimizes costs**: Scale based on InvocationsPerInstance or other metrics
7. **Inference Recommender finds optimal config**: Automated benchmarking and recommendations
8. **Async inference for large payloads**: Queue-based processing with up to 1GB payloads
9. **SageMaker Projects for MLOps**: Pre-built templates with CI/CD
10. **Canary deployments reduce risk**: Gradually shift traffic to validate new models

---

## Navigation

| Previous | Next |
|----------|------|
| [Domain 2: ML Model Development](/posts/aws-mla-c01-domain-2-model-development/) | [Domain 4: Monitoring and Security](/posts/aws-mla-c01-domain-4-monitoring-security/) |
