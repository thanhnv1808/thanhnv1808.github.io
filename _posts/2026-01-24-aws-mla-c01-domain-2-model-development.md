---
title: "AWS MLA-C01 - Domain 2: ML Model Development"
author: thanhnv1808
date: 2026-01-24 11:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, model-development, sagemaker, algorithms, training]
description: "Domain 2 covers 26% of the exam (~17 questions). Master algorithm selection, SageMaker built-in algorithms, training strategies, hyperparameter tuning, and model evaluation."
pin: false
comments: true
---

## Domain 2 Overview

**Exam Weight: 26% (~17 questions)**

This domain focuses on selecting appropriate ML algorithms, training models using Amazon SageMaker, implementing hyperparameter optimization, and evaluating model performance.

### Task Statements

| Task | Description |
|------|-------------|
| 2.1 | Select ML algorithms for specific use cases |
| 2.2 | Train and validate ML models using SageMaker |
| 2.3 | Perform hyperparameter tuning and optimization |
| 2.4 | Evaluate and compare model performance |

---

## Task 2.1: Select ML Algorithms for Specific Use Cases

### ML Problem Types and Algorithms

```
┌─────────────────────────────────────────────────────────────────┐
│                    ML PROBLEM TYPES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Supervised Learning          Unsupervised Learning            │
│   ┌─────────────────┐          ┌─────────────────┐             │
│   │ Classification  │          │   Clustering    │             │
│   │ • Binary        │          │   • K-Means     │             │
│   │ • Multiclass    │          │   • DBSCAN      │             │
│   │ • Multilabel    │          │   • Hierarchical│             │
│   └─────────────────┘          └─────────────────┘             │
│                                                                  │
│   ┌─────────────────┐          ┌─────────────────┐             │
│   │  Regression     │          │ Dimensionality  │             │
│   │ • Linear        │          │  Reduction      │             │
│   │ • Time Series   │          │ • PCA           │             │
│   │ • Forecasting   │          │ • t-SNE         │             │
│   └─────────────────┘          └─────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SageMaker Built-in Algorithms

#### Classification Algorithms

| Algorithm | Use Case | Input Format | Key Features |
|-----------|----------|--------------|--------------|
| **XGBoost** | Tabular data classification/regression | CSV, LibSVM, Parquet, RecordIO | Gradient boosting, handles missing values, fast |
| **Linear Learner** | Binary/multiclass classification, regression | RecordIO-protobuf, CSV | Scalable, linear models, auto-normalization |
| **k-NN** | Classification/regression | RecordIO-protobuf, CSV | Instance-based, no training phase |
| **Factorization Machines** | High-dimensional sparse data | RecordIO-protobuf | Click prediction, recommendations |
| **Image Classification** | Computer vision | Image files (JPG, PNG) | ResNet CNN, transfer learning |
| **BlazingText** | Text classification | Text files | FastText implementation, word2vec |

#### Regression Algorithms

| Algorithm | Use Case | Best For |
|-----------|----------|----------|
| **XGBoost** | General tabular regression | Complex non-linear relationships |
| **Linear Learner** | Linear regression | Large-scale, high-dimensional data |
| **k-NN** | Non-parametric regression | Small datasets, local patterns |
| **DeepAR** | Time series forecasting | Multiple related time series |

#### Clustering Algorithms

| Algorithm | Use Case | Characteristics |
|-----------|----------|----------------|
| **k-Means** | Customer segmentation, anomaly detection | Requires k specification, scalable |
| **Random Cut Forest (RCF)** | Anomaly detection | Unsupervised, handles streaming data |

#### NLP and Text Algorithms

| Algorithm | Use Case | Output |
|-----------|----------|--------|
| **BlazingText** | Text classification, word embeddings | Categories or word vectors |
| **Sequence-to-Sequence** | Machine translation, summarization | Text sequences |
| **LDA (Latent Dirichlet Allocation)** | Topic modeling | Topic distributions |
| **Neural Topic Model (NTM)** | Topic discovery | Topic mixtures |

#### Computer Vision Algorithms

| Algorithm | Use Case | Input |
|-----------|----------|-------|
| **Image Classification** | Object categorization | Images |
| **Object Detection** | Object localization | Images with annotations |
| **Semantic Segmentation** | Pixel-level classification | Images with masks |

#### Recommendation Algorithms

| Algorithm | Use Case | Method |
|-----------|----------|--------|
| **Factorization Machines** | Click-through rate prediction | Matrix factorization |
| **Neural Collaborative Filtering** | User-item recommendations | Deep learning |

> **Exam Tip**: XGBoost is the go-to algorithm for tabular data and is one of the most commonly used SageMaker built-in algorithms.
{: .prompt-tip }

### Algorithm Selection Framework

```
┌─────────────────────────────────────────────────────────────────┐
│               ALGORITHM SELECTION DECISION TREE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   What is your data type?                                       │
│   ├─ Tabular → XGBoost, Linear Learner, k-NN                   │
│   ├─ Images → Image Classification, Object Detection           │
│   ├─ Text → BlazingText, Seq2Seq, LDA                          │
│   ├─ Time Series → DeepAR                                      │
│   └─ Sparse/High-dim → Factorization Machines                  │
│                                                                  │
│   What is your task?                                            │
│   ├─ Classification → XGBoost, Linear Learner, Image Class     │
│   ├─ Regression → XGBoost, Linear Learner, DeepAR              │
│   ├─ Clustering → k-Means, RCF                                 │
│   ├─ Recommendations → Factorization Machines                  │
│   └─ Anomaly Detection → RCF                                   │
│                                                                  │
│   What is your data size?                                       │
│   ├─ Small (<10K) → k-NN, traditional ML                       │
│   ├─ Medium → XGBoost, Linear Learner                          │
│   └─ Large (>1M) → Linear Learner, Neural algorithms           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### XGBoost Deep Dive

**XGBoost** (eXtreme Gradient Boosting) is the most versatile SageMaker algorithm.

| Feature | Description |
|---------|-------------|
| **Algorithm Type** | Gradient boosted trees |
| **Problem Types** | Classification, regression, ranking |
| **Training** | Distributed training supported |
| **Instance Types** | CPU or GPU (GPU faster for large datasets) |
| **Hyperparameters** | 40+ parameters for fine-tuning |

**Key Hyperparameters:**

| Parameter | Description | Typical Values |
|-----------|-------------|----------------|
| `num_round` | Number of boosting rounds | 50-500 |
| `max_depth` | Tree depth | 3-10 |
| `eta` | Learning rate | 0.01-0.3 |
| `subsample` | Fraction of training data | 0.5-1.0 |
| `colsample_bytree` | Fraction of features | 0.5-1.0 |
| `objective` | Loss function | reg:squarederror, binary:logistic |

```python
# XGBoost training example
from sagemaker.xgboost import XGBoost

xgb = XGBoost(
    entry_point='train.py',
    framework_version='1.7-1',
    instance_type='ml.m5.xlarge',
    instance_count=1,
    hyperparameters={
        'max_depth': 5,
        'eta': 0.2,
        'objective': 'binary:logistic',
        'num_round': 100
    }
)

xgb.fit({'train': s3_train_data, 'validation': s3_val_data})
```

### Linear Learner Deep Dive

**Linear Learner** provides optimized linear models for large-scale datasets.

| Feature | Description |
|---------|-------------|
| **Algorithms** | Linear regression, logistic regression, softmax |
| **Optimization** | SGD, Adam, Adagrad, etc. |
| **Auto-tuning** | Automatic normalization, multiple parallel models |
| **Scalability** | Handles billions of features |

**Key Hyperparameters:**

| Parameter | Description |
|-----------|-------------|
| `predictor_type` | binary_classifier, multiclass_classifier, regressor |
| `learning_rate` | Step size for optimizer |
| `l1` | L1 regularization |
| `wd` | L2 regularization (weight decay) |
| `epochs` | Number of passes through data |

> **Exam Tip**: Linear Learner automatically trains multiple models with different hyperparameters and selects the best one.
{: .prompt-tip }

### DeepAR for Time Series

**DeepAR** is a supervised learning algorithm for forecasting time series using RNNs.

| Feature | Description |
|---------|-------------|
| **Input** | Multiple related time series |
| **Output** | Probabilistic forecasts |
| **Use Cases** | Demand forecasting, capacity planning |
| **Advantages** | Learns across time series, handles missing data |

**Input Format:**
```json
{
    "start": "2023-01-01 00:00:00",
    "target": [120.5, 135.2, 142.8, ...],
    "cat": [0],
    "dynamic_feat": [[0.5], [0.6], [0.7], ...]
}
```

---

## Task 2.2: Train and Validate ML Models Using SageMaker

### SageMaker Training Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   SAGEMAKER TRAINING FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────┐      ┌─────────────┐      ┌──────────┐          │
│   │ Training │─────▶│  Training   │─────▶│  Model   │          │
│   │   Data   │      │   Job       │      │ Artifacts│          │
│   │  (S3)    │      │ (Container) │      │  (S3)    │          │
│   └──────────┘      └─────────────┘      └──────────┘          │
│                            │                                     │
│   ┌──────────┐      ┌─────────────┐      ┌──────────┐          │
│   │Validation│      │  Compute    │      │CloudWatch│          │
│   │   Data   │      │  Instances  │      │  Metrics │          │
│   │  (S3)    │      │             │      │          │          │
│   └──────────┘      └─────────────┘      └──────────┘          │
│                                                                  │
│   Training Instance Lifecycle:                                  │
│   1. Download training image and data                           │
│   2. Start training container                                   │
│   3. Execute training algorithm                                 │
│   4. Upload model artifacts to S3                               │
│   5. Terminate instance                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Training Job Configuration

| Parameter | Description | Example |
|-----------|-------------|---------|
| **Algorithm Specification** | Container image | XGBoost, custom container |
| **Input Data Config** | S3 location, content type | s3://bucket/train/, CSV |
| **Output Data Config** | Model artifact location | s3://bucket/models/ |
| **Resource Config** | Instance type, count, volume size | ml.m5.xlarge, 1, 30GB |
| **Stopping Condition** | Max runtime | 3600 seconds |
| **Hyperparameters** | Algorithm-specific params | max_depth=5, eta=0.2 |
| **VPC Config** | Network isolation | Subnets, security groups |

### Training Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **File Mode** | Download entire dataset to training instance | Small to medium datasets |
| **Pipe Mode** | Stream data from S3 during training | Large datasets, faster startup |
| **Fast File Mode** | Lazy loading from S3 | Large datasets, random access needed |

### Data Input Channels

```python
# Multiple input channels example
from sagemaker.inputs import TrainingInput

train_input = TrainingInput(
    s3_data='s3://bucket/train/',
    content_type='text/csv'
)

validation_input = TrainingInput(
    s3_data='s3://bucket/validation/',
    content_type='text/csv'
)

estimator.fit({
    'train': train_input,
    'validation': validation_input
})
```

### Instance Types for Training

| Instance Family | Use Case | Cost |
|-----------------|----------|------|
| **ml.m5** | General purpose, balanced | Low |
| **ml.c5** | Compute optimized | Low-Medium |
| **ml.p3/p4** | GPU training (deep learning) | High |
| **ml.g4dn** | GPU training (cost-effective) | Medium |
| **ml.inf1** | AWS Inferentia (inference-optimized) | Low |

> **Exam Tip**: Use GPU instances (ml.p3, ml.g4dn) for deep learning and computer vision. Use CPU instances (ml.m5, ml.c5) for traditional ML algorithms like XGBoost.
{: .prompt-tip }

### Distributed Training

#### Data Parallelism

```
┌─────────────────────────────────────────────────────────────────┐
│                      DATA PARALLELISM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Model Replica 1     Model Replica 2     Model Replica 3       │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│   │   Model     │     │   Model     │     │   Model     │       │
│   │  (copy 1)   │     │  (copy 2)   │     │  (copy 3)   │       │
│   └─────────────┘     └─────────────┘     └─────────────┘       │
│         │                   │                   │                │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│   │  Data       │     │  Data       │     │  Data       │       │
│   │  Batch 1    │     │  Batch 2    │     │  Batch 3    │       │
│   └─────────────┘     └─────────────┘     └─────────────┘       │
│                                                                  │
│   Gradients synchronized across all replicas                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**When to use:** Large datasets, model fits in memory

#### Model Parallelism

```
┌─────────────────────────────────────────────────────────────────┐
│                      MODEL PARALLELISM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Instance 1          Instance 2          Instance 3            │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│   │   Layers    │     │   Layers    │     │   Layers    │       │
│   │    1-5      │────▶│    6-10     │────▶│   11-15     │       │
│   └─────────────┘     └─────────────┘     └─────────────┘       │
│                                                                  │
│   Same data batch flows through all instances                   │
│   Model partitioned across instances                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**When to use:** Large models that don't fit in single GPU memory

### SageMaker Distributed Training Libraries

| Library | Type | Use Case |
|---------|------|----------|
| **SageMaker Data Parallel** | Data parallelism | Distributed deep learning training |
| **SageMaker Model Parallel** | Model parallelism | Large models (billions of parameters) |

```python
# Data parallel training example
from sagemaker.pytorch import PyTorch

estimator = PyTorch(
    entry_point='train.py',
    role=role,
    framework_version='1.12',
    py_version='py38',
    instance_type='ml.p3.16xlarge',
    instance_count=4,
    distribution={
        'smdistributed': {
            'dataparallel': {
                'enabled': True
            }
        }
    }
)
```

### Spot Instances for Training

| Feature | Description |
|---------|-------------|
| **Cost Savings** | Up to 90% cost reduction |
| **Managed Spot** | SageMaker handles interruptions |
| **Checkpointing** | Save/resume from checkpoints |
| **Use Case** | Long-running, fault-tolerant training |

```python
estimator = XGBoost(
    ...,
    use_spot_instances=True,
    max_run=3600,
    max_wait=7200,  # Maximum time to wait for spot
    checkpoint_s3_uri='s3://bucket/checkpoints/'
)
```

> **Exam Tip**: Managed Spot Training can save up to 90% on training costs. Requires checkpointing to handle interruptions.
{: .prompt-warning }

---

## Task 2.3: Perform Hyperparameter Tuning and Optimization

### Hyperparameter Tuning Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              SAGEMAKER HYPERPARAMETER TUNING                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────────────────────────────────────────┐      │
│   │  Tuning Job (Orchestrator)                           │      │
│   │  • Define hyperparameter ranges                      │      │
│   │  • Select optimization strategy                      │      │
│   │  • Specify objective metric                          │      │
│   └──────────────────────────────────────────────────────┘      │
│                           │                                      │
│           ┌───────────────┼───────────────┐                     │
│           │               │               │                     │
│   ┌───────▼─────┐  ┌──────▼──────┐  ┌────▼────────┐            │
│   │ Training    │  │  Training   │  │  Training   │            │
│   │  Job 1      │  │   Job 2     │  │   Job 3     │            │
│   │  (config 1) │  │  (config 2) │  │  (config 3) │            │
│   └─────────────┘  └─────────────┘  └─────────────┘            │
│          │                │                │                    │
│   ┌──────▼────────────────▼────────────────▼──────┐             │
│   │  Best Hyperparameter Configuration            │             │
│   └────────────────────────────────────────────────┘             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Hyperparameter Types

| Type | Description | Example |
|------|-------------|---------|
| **Continuous** | Numerical range | learning_rate: 0.001-0.1 |
| **Integer** | Discrete numbers | num_layers: 1-10 |
| **Categorical** | Fixed set of values | optimizer: ['sgd', 'adam', 'rmsprop'] |

### Tuning Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Bayesian** | Uses past results to choose next config | Most efficient, default |
| **Random** | Random sampling | Large search space, parallel jobs |
| **Grid** | Exhaustive search (not supported by SageMaker) | Small search space |
| **Hyperband** | Resource-efficient, early stopping | Limited budget |

### Objective Metrics

| Metric Type | Examples | Goal |
|-------------|----------|------|
| **Maximization** | Accuracy, F1, AUC, Precision, Recall | Maximize |
| **Minimization** | Loss, Error, RMSE, MAE | Minimize |

### Tuning Job Configuration

```python
from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter, CategoricalParameter

# Define hyperparameter ranges
hyperparameter_ranges = {
    'max_depth': IntegerParameter(3, 10),
    'eta': ContinuousParameter(0.01, 0.3),
    'subsample': ContinuousParameter(0.5, 1.0),
    'num_round': IntegerParameter(50, 300),
    'objective': CategoricalParameter(['binary:logistic', 'binary:hinge'])
}

# Define objective metric
objective_metric_name = 'validation:auc'

# Create tuner
tuner = HyperparameterTuner(
    estimator=xgb_estimator,
    objective_metric_name=objective_metric_name,
    hyperparameter_ranges=hyperparameter_ranges,
    metric_definitions=[
        {'Name': 'validation:auc', 'Regex': 'validation-auc:([0-9\\.]+)'}
    ],
    max_jobs=20,
    max_parallel_jobs=3,
    strategy='Bayesian'
)

# Start tuning
tuner.fit({'train': s3_train, 'validation': s3_val})
```

### Tuning Job Limits

| Parameter | Description | Consideration |
|-----------|-------------|---------------|
| `max_jobs` | Total training jobs | Budget constraint |
| `max_parallel_jobs` | Concurrent jobs | Account limits, faster completion |
| `max_runtime_in_seconds` | Per-job timeout | Prevent runaway jobs |

### Early Stopping

**Automatic Model Tuning with Early Stopping** terminates underperforming training jobs early.

| Type | Description |
|------|-------------|
| **Auto** | SageMaker decides when to stop |
| **Off** | All jobs run to completion |

Benefits:
- Reduces training time
- Lowers costs
- Improves efficiency

```python
tuner = HyperparameterTuner(
    ...,
    early_stopping_type='Auto'
)
```

### Warm Start Tuning

**Warm Start** uses results from previous tuning jobs to initialize new jobs.

```python
from sagemaker.tuner import WarmStartConfig, WarmStartTypes

warm_start_config = WarmStartConfig(
    warm_start_type=WarmStartTypes.IDENTICAL_DATA_AND_ALGORITHM,
    parents={'previous-tuning-job-name'}
)

tuner = HyperparameterTuner(
    ...,
    warm_start_config=warm_start_config
)
```

**Warm Start Types:**
- `IDENTICAL_DATA_AND_ALGORITHM`: Same data and algorithm
- `TRANSFER_LEARNING`: Different dataset, same algorithm

> **Exam Tip**: Bayesian optimization is the default and most efficient tuning strategy. Use warm start to leverage previous tuning results.
{: .prompt-tip }

---

## Task 2.4: Evaluate and Compare Model Performance

### Model Evaluation Metrics

#### Classification Metrics

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONFUSION MATRIX                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                      Predicted                                   │
│                   Positive    Negative                           │
│   Actual  Positive   TP          FN                              │
│           Negative   FP          TN                              │
│                                                                  │
│   Accuracy = (TP + TN) / (TP + TN + FP + FN)                    │
│   Precision = TP / (TP + FP)                                    │
│   Recall = TP / (TP + FN)                                       │
│   F1 Score = 2 * (Precision * Recall) / (Precision + Recall)    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Metric | Formula | When to Use |
|--------|---------|-------------|
| **Accuracy** | Correct predictions / Total predictions | Balanced datasets |
| **Precision** | TP / (TP + FP) | When false positives are costly |
| **Recall** | TP / (TP + FN) | When false negatives are costly |
| **F1 Score** | Harmonic mean of precision and recall | Balance between precision and recall |
| **AUC-ROC** | Area under ROC curve | Binary classification, imbalanced data |
| **AUC-PR** | Area under precision-recall curve | Imbalanced datasets |

#### Regression Metrics

| Metric | Description | Range |
|--------|-------------|-------|
| **MSE** | Mean Squared Error | 0 to ∞ (lower is better) |
| **RMSE** | Root Mean Squared Error | 0 to ∞ (lower is better) |
| **MAE** | Mean Absolute Error | 0 to ∞ (lower is better) |
| **R²** | Coefficient of determination | -∞ to 1 (higher is better) |
| **MAPE** | Mean Absolute Percentage Error | 0% to ∞% (lower is better) |

#### Clustering Metrics

| Metric | Description |
|--------|-------------|
| **Silhouette Score** | How similar points are to their cluster vs other clusters |
| **Inertia** | Sum of squared distances to cluster centers |
| **Davies-Bouldin Index** | Average similarity between clusters |

### Cross-Validation

```
┌─────────────────────────────────────────────────────────────────┐
│                    K-FOLD CROSS-VALIDATION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Fold 1:  [Test] [Train] [Train] [Train] [Train]              │
│   Fold 2:  [Train] [Test] [Train] [Train] [Train]              │
│   Fold 3:  [Train] [Train] [Test] [Train] [Train]              │
│   Fold 4:  [Train] [Train] [Train] [Test] [Train]              │
│   Fold 5:  [Train] [Train] [Train] [Train] [Test]              │
│                                                                  │
│   Final Score = Average of all fold scores                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Model Comparison Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Holdout Validation** | Single train/test split | Large datasets |
| **K-Fold CV** | Multiple train/test splits | Limited data |
| **Stratified K-Fold** | Preserve class distribution | Imbalanced data |
| **Time Series Split** | Chronological splits | Time series data |
| **A/B Testing** | Production comparison | Live traffic |

### SageMaker Experiments

**SageMaker Experiments** automatically tracks and compares training runs.

```python
from sagemaker.experiments import Run

with Run(
    experiment_name='image-classification-exp',
    run_name='xgboost-run-1',
    sagemaker_session=sagemaker_session
) as run:
    # Log parameters
    run.log_parameter('max_depth', 5)
    run.log_parameter('eta', 0.2)

    # Train model
    estimator.fit({'train': train_input})

    # Log metrics
    run.log_metric('train:accuracy', 0.95)
    run.log_metric('validation:accuracy', 0.92)
```

### SageMaker Model Registry

Track and version models in a central repository.

| Feature | Description |
|---------|-------------|
| **Model Groups** | Logical grouping of model versions |
| **Model Versions** | Individual trained models |
| **Approval Status** | Pending, Approved, Rejected |
| **Metadata** | Metrics, hyperparameters, lineage |

```python
from sagemaker.model import Model

model = Model(
    image_uri=training_image,
    model_data=model_artifacts,
    role=role
)

model_package = model.register(
    model_package_group_name='customer-churn-models',
    approval_status='PendingManualApproval',
    inference_instances=['ml.m5.xlarge'],
    transform_instances=['ml.m5.xlarge'],
    model_metrics={
        'validation:accuracy': 0.92,
        'validation:f1': 0.88
    }
)
```

### Bias Detection and Model Explainability

#### SageMaker Clarify

**SageMaker Clarify** detects bias and explains model predictions.

| Feature | Description |
|---------|-------------|
| **Pre-training Bias** | Bias in training data |
| **Post-training Bias** | Bias in model predictions |
| **Feature Importance** | SHAP values for explainability |
| **Partial Dependence Plots** | Feature impact visualization |

**Bias Metrics:**
- **Class Imbalance (CI)**: Imbalance in dataset labels
- **Difference in Positive Proportions (DPL)**: Difference in positive outcomes
- **Disparate Impact (DI)**: Ratio of positive outcomes between groups
- **Conditional Demographic Disparity (CDD)**: Outcome differences within subgroups

```python
from sagemaker.clarify import (
    SageMakerClarifyProcessor,
    BiasConfig,
    DataConfig,
    ModelConfig
)

clarify_processor = SageMakerClarifyProcessor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge'
)

bias_config = BiasConfig(
    label_values_or_threshold=[1],
    facet_name='gender',
    facet_values_or_threshold=[0]
)

clarify_processor.run_bias(
    data_config=data_config,
    bias_config=bias_config,
    model_config=model_config
)
```

> **Exam Tip**: SageMaker Clarify provides both bias detection and model explainability (SHAP values) in a single service.
{: .prompt-tip }

---

## Key Definitions

| Term | Definition |
|------|------------|
| **Hyperparameter** | Configuration external to the model, set before training |
| **Epoch** | One complete pass through the entire training dataset |
| **Batch Size** | Number of samples processed before updating model weights |
| **Learning Rate** | Step size for weight updates during training |
| **Regularization** | Technique to prevent overfitting (L1, L2, dropout) |
| **Gradient Descent** | Optimization algorithm to minimize loss function |
| **Bayesian Optimization** | Sequential model-based optimization for hyperparameters |
| **SHAP Values** | Shapley Additive exPlanations for feature importance |

---

## Practice Questions

### Question 1
A data scientist needs to classify customer reviews as positive, negative, or neutral. The dataset contains 50,000 text reviews. Which SageMaker algorithm is most appropriate?

A) Linear Learner
B) XGBoost
C) BlazingText
D) Image Classification

<details>
<summary>View Answer</summary>

**Answer: C**

BlazingText is designed for text classification tasks and implements the FastText algorithm, which is optimized for text data. It can handle multi-class classification (positive/negative/neutral) efficiently.

Linear Learner (A) works with numerical features, not raw text. XGBoost (B) requires feature engineering for text. Image Classification (D) is for images, not text.

</details>

### Question 2
A machine learning engineer wants to train a large deep learning model that doesn't fit in the memory of a single GPU. Which training approach should they use?

A) Data parallelism with SageMaker Data Parallel library
B) Model parallelism with SageMaker Model Parallel library
C) Distributed training with Horovod
D) Spot instance training

<details>
<summary>View Answer</summary>

**Answer: B**

Model parallelism splits the model across multiple GPUs/instances, allowing models larger than single GPU memory to be trained. SageMaker Model Parallel library handles this automatically.

Data parallelism (A) replicates the entire model on each instance, so it won't help if the model doesn't fit in memory. Horovod (C) is primarily for data parallelism. Spot instances (D) address cost, not memory limitations.

</details>

### Question 3
A team is performing hyperparameter tuning with a budget constraint. They want to run 50 training jobs but can only afford 10 parallel jobs at a time. Which configuration should they use?

A) max_jobs=10, max_parallel_jobs=50
B) max_jobs=50, max_parallel_jobs=10
C) max_jobs=50, max_parallel_jobs=50
D) max_jobs=10, max_parallel_jobs=10

<details>
<summary>View Answer</summary>

**Answer: B**

`max_jobs=50` sets the total number of training jobs, and `max_parallel_jobs=10` limits concurrent execution to 10, respecting the budget constraint while eventually running all 50 jobs.

Option A has the parameters reversed. Option C would try to run all 50 in parallel. Option D would only run 10 total jobs.

</details>

### Question 4
Which metric should be prioritized when false negatives are very costly, such as in cancer detection?

A) Precision
B) Accuracy
C) Recall
D) F1 Score

<details>
<summary>View Answer</summary>

**Answer: C**

Recall (sensitivity) measures the proportion of actual positives correctly identified. In cancer detection, missing a positive case (false negative) is very costly, so maximizing recall is critical.

Precision (A) focuses on minimizing false positives. Accuracy (B) can be misleading with imbalanced data. F1 Score (D) balances both but doesn't prioritize recall.

</details>

### Question 5
A data scientist wants to reduce training costs by 70% for a long-running training job. The job can tolerate interruptions if checkpoints are saved. Which approach should they use?

A) Use smaller instance types
B) Enable managed spot training with checkpointing
C) Reduce the number of epochs
D) Use Pipe mode instead of File mode

<details>
<summary>View Answer</summary>

**Answer: B**

Managed spot training can reduce costs by up to 90%. With checkpointing enabled, SageMaker automatically saves and resumes from checkpoints when spot instances are interrupted.

Smaller instances (A) may increase training time. Reducing epochs (C) affects model quality. Pipe mode (D) improves training speed but doesn't reduce costs significantly.

</details>

### Question 6
Which SageMaker built-in algorithm is best suited for time series forecasting with multiple related time series?

A) XGBoost
B) Linear Learner
C) DeepAR
D) Random Cut Forest

<details>
<summary>View Answer</summary>

**Answer: C**

DeepAR is specifically designed for time series forecasting and can learn patterns across multiple related time series, providing probabilistic forecasts.

XGBoost (A) can be used for time series but doesn't specialize in it. Linear Learner (B) is for classification/regression. Random Cut Forest (D) is for anomaly detection.

</details>

---

## Key Takeaways

1. **XGBoost is versatile**: Use for tabular classification and regression problems
2. **Choose the right algorithm**: Match algorithm to data type (tabular, text, images, time series)
3. **GPU for deep learning**: Use ml.p3 or ml.g4dn instances for neural networks and computer vision
4. **Data parallelism vs model parallelism**: Data parallel for large datasets, model parallel for large models
5. **Bayesian optimization**: Default and most efficient hyperparameter tuning strategy
6. **Early stopping**: Enable to reduce tuning costs and time
7. **Managed spot training**: Save up to 90% on training costs with checkpointing
8. **SageMaker Clarify**: Single service for bias detection and model explainability
9. **Recall for costly false negatives**: Prioritize recall in medical diagnosis, fraud detection
10. **SageMaker Experiments**: Automatically track and compare training runs

---

## Navigation

| Previous | Next |
|----------|------|
| [Domain 1: Data Preparation](/posts/aws-mla-c01-domain-1-data-preparation/) | [Domain 3: Deployment and Orchestration](/posts/aws-mla-c01-domain-3-deployment-orchestration/) |
