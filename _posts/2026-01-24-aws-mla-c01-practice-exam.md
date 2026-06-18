---
title: "AWS MLA-C01 - Full Practice Exam (65 Questions)"
author: thanhnv1808
date: 2026-01-24 16:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, practice-exam, mock-test, certification]
description: Full-length practice exam for AWS Machine Learning Engineer Associate (MLA-C01) with 65 questions covering all domains. Test your exam readiness!
pin: false
comments: true
---

## Practice Exam Instructions

This practice exam simulates the real AWS Certified Machine Learning Engineer - Associate (MLA-C01) exam.

| Exam Detail | Information |
|-------------|-------------|
| **Total Questions** | 65 |
| **Time Limit** | 170 minutes (2 hours 50 minutes) |
| **Passing Score** | 720/1000 |
| **Question Types** | Multiple choice, Multiple response |

### Domain Distribution

| Domain | Questions |
|--------|-----------|
| Domain 1: Data Preparation for ML | 18 (28%) |
| Domain 2: ML Model Development | 17 (26%) |
| Domain 3: Deployment and Orchestration | 14 (22%) |
| Domain 4: Monitoring, Maintenance, Security | 16 (24%) |

> **Tip**: Time yourself! Aim for about 2.5 minutes per question.
{: .prompt-tip }

---

## Domain 1: Data Preparation for ML (Questions 1-18)

### Question 1
A data engineer needs to ingest streaming data from IoT devices and store it in S3 for ML training. Which service provides the simplest fully managed solution with automatic scaling?

A. Amazon Kinesis Data Streams with Lambda consumers
B. Amazon Kinesis Data Firehose
C. Amazon MSK (Managed Streaming for Apache Kafka)
D. AWS Glue Streaming ETL

<details>
<summary>Show Answer</summary>
<b>B. Amazon Kinesis Data Firehose</b>

Kinesis Data Firehose is the simplest solution because it:
- Automatically delivers streaming data to S3
- Handles scaling automatically
- Requires no consumer management
- Supports data transformation via Lambda

Kinesis Data Streams (A) requires managing consumers. MSK (C) requires more operational overhead. Glue Streaming (D) is more complex than needed.
</details>

---

### Question 2
A company has training data stored in multiple formats: CSV, JSON, and Parquet. They want to convert everything to an optimized format for large-scale ML training in SageMaker. Which format should they choose?

A. CSV
B. JSON
C. Parquet
D. Avro

<details>
<summary>Show Answer</summary>
<b>C. Parquet</b>

Parquet is the optimal format for large-scale ML because it:
- Uses columnar storage (faster reads for analytics)
- Provides efficient compression
- Includes schema information
- Works well with SageMaker and Athena

CSV (A) is inefficient for large datasets. JSON (B) is verbose. Avro (D) is less commonly used with SageMaker.
</details>

---

### Question 3
A data scientist needs to perform feature engineering on a 500GB dataset using Spark transformations. Which AWS service should they use?

A. AWS Lambda
B. AWS Glue ETL Job (Spark)
C. Amazon SageMaker Processing with scikit-learn
D. AWS Batch

<details>
<summary>Show Answer</summary>
<b>B. AWS Glue ETL Job (Spark)</b>

AWS Glue ETL Jobs are ideal for large-scale Spark transformations:
- Native Spark support with auto-scaling DPUs
- Serverless (no infrastructure management)
- Integrated with Data Catalog
- Cost-effective for large datasets

Lambda (A) has time and memory limits. SageMaker Processing (C) with scikit-learn doesn't scale as well for 500GB. AWS Batch (D) requires more management.
</details>

---

### Question 4
A machine learning pipeline requires storing features for both real-time inference and batch training, with the ability to perform point-in-time queries to prevent data leakage. Which service should be used?

A. Amazon DynamoDB for real-time and S3 for batch
B. Amazon ElastiCache for real-time and Redshift for batch
C. Amazon SageMaker Feature Store
D. Amazon RDS for both real-time and batch

<details>
<summary>Show Answer</summary>
<b>C. Amazon SageMaker Feature Store</b>

SageMaker Feature Store provides:
- Online store for low-latency real-time access
- Offline store (S3) for batch training
- Automatic synchronization between stores
- Point-in-time queries to prevent data leakage
- Built-in versioning and metadata management

Other options (A, B, D) require manual synchronization and don't support point-in-time queries.
</details>

---

### Question 5
Which data validation rule language should be used with AWS Glue to check that customer age values are between 0 and 120?

A. SQL WHERE clause
B. AWS Glue Data Quality Definition Language (DQDL)
C. Python assertions
D. AWS Config Rules

<details>
<summary>Show Answer</summary>
<b>B. AWS Glue Data Quality Definition Language (DQDL)</b>

DQDL is specifically designed for data quality validation in Glue:
```
ColumnValues "age" between 0 and 120
```

SQL (A) is for querying, not validation rules. Python assertions (C) require custom code. Config Rules (D) are for AWS resource configuration compliance.
</details>

---

### Question 6
A dataset has a categorical variable "country" with 150 unique values. Which encoding method is most appropriate for a tree-based model like XGBoost?

A. One-hot encoding
B. Label encoding
C. Target encoding
D. Binary encoding

<details>
<summary>Show Answer</summary>
<b>C. Target encoding</b>

Target encoding is ideal for high cardinality categorical variables in tree-based models:
- Avoids dimensionality explosion (one-hot would create 150 columns)
- Captures relationship between category and target
- Works well with XGBoost and other tree algorithms

One-hot (A) would create too many features. Label encoding (B) implies ordering. Binary encoding (D) is less effective than target encoding for this case.
</details>

---

### Question 7
A data engineer needs to handle missing values in a time series dataset where the missingness pattern is random. Which imputation strategy is most appropriate?

A. Mean imputation
B. Forward fill
C. Backward fill
D. Delete rows with missing values

<details>
<summary>Show Answer</summary>
<b>B. Forward fill</b>

For time series data, forward fill (or backward fill) preserves temporal patterns:
- Uses last known value to fill gaps
- Maintains time series continuity
- Appropriate for random missingness in sequential data

Mean imputation (A) ignores temporal relationships. Backward fill (C) could use future information. Deleting rows (D) disrupts the time series.
</details>

---

### Question 8
Which SageMaker service provides a visual interface with 300+ built-in transformations for data preparation and can export workflows to SageMaker Processing or Pipelines?

A. SageMaker Studio
B. SageMaker Data Wrangler
C. SageMaker Feature Store
D. SageMaker Clarify

<details>
<summary>Show Answer</summary>
<b>B. SageMaker Data Wrangler</b>

Data Wrangler provides:
- Visual data preparation interface
- 300+ built-in transformations
- Custom Python/Spark transforms
- Export to Processing, Pipelines, Feature Store, or code

Studio (A) is the IDE environment. Feature Store (C) is for storage. Clarify (D) is for bias detection.
</details>

---

### Question 9
A company stores ML training data in S3 Standard. After 90 days, data is rarely accessed but must be retained for 3 years for compliance. What is the most cost-effective storage strategy?

A. Keep all data in S3 Standard
B. Use S3 lifecycle policy to transition to S3 Standard-IA after 90 days, then to Glacier after 1 year
C. Move all data to S3 Glacier immediately
D. Delete old data after 90 days

<details>
<summary>Show Answer</summary>
<b>B. Use S3 lifecycle policy to transition to S3 Standard-IA after 90 days, then to Glacier after 1 year</b>

This strategy balances cost and access:
- First 90 days: S3 Standard (frequent access for recent training)
- 90 days - 1 year: S3 Standard-IA (infrequent access, lower cost)
- After 1 year: Glacier (archival, lowest cost, compliance retention)

Option A is expensive. Option C prevents recent access. Option D violates compliance.
</details>

---

### Question 10
Which AWS service provides column-level and row-level access control for data lake resources?

A. AWS IAM
B. Amazon S3 bucket policies
C. AWS Lake Formation
D. Amazon Athena

<details>
<summary>Show Answer</summary>
<b>C. AWS Lake Formation</b>

Lake Formation provides fine-grained data access control:
- Column-level permissions
- Row-level security
- Tag-based access control
- Centralized permission management

IAM (A) and S3 policies (B) provide resource-level control, not column/row level. Athena (D) is a query service.
</details>

---

### Question 11
A dataset has 100,000 rows with a target variable distribution of 98% negative class and 2% positive class. How should the data be split for model training?

A. Random 80/20 split
B. Stratified split maintaining 98/2 distribution
C. Only use positive class examples
D. Oversample positive class before any splitting

<details>
<summary>Show Answer</summary>
<b>B. Stratified split maintaining 98/2 distribution</b>

Stratified splitting preserves the class distribution in both train and test sets, ensuring:
- Both sets have the same 98/2 ratio
- Model evaluation is representative
- No bias from unbalanced splits

Random split (A) might create different distributions. Option C loses valuable negative examples. Oversampling (D) should be done after splitting.
</details>

---

### Question 12
What is the main difference between AWS Glue Crawlers and AWS Glue Data Catalog?

A. Crawlers create jobs, Data Catalog stores metadata
B. Crawlers discover schema, Data Catalog stores metadata
C. Crawlers run transformations, Data Catalog stores data
D. They are the same service

<details>
<summary>Show Answer</summary>
<b>B. Crawlers discover schema, Data Catalog stores metadata</b>

- Crawlers: Scan data sources and automatically infer schemas
- Data Catalog: Central metadata repository that stores table definitions, schemas, and data locations

Crawlers populate the Data Catalog with discovered metadata.
</details>

---

### Question 13
A time series forecasting model needs train, validation, and test splits. How should the data be split?

A. Random 60/20/20 split
B. Chronological split: oldest 60% train, next 20% validation, newest 20% test
C. K-fold cross-validation
D. Stratified split

<details>
<summary>Show Answer</summary>
<b>B. Chronological split: oldest 60% train, next 20% validation, newest 20% test</b>

Time series data requires chronological splitting to:
- Prevent data leakage from future to past
- Simulate real-world forecasting scenario
- Maintain temporal dependencies

Random split (A) causes leakage. K-fold (C) mixes time periods. Stratified (D) doesn't preserve temporal order.
</details>

---

### Question 14
Which transformation should be applied to a feature with values ranging from 100 to 1,000,000 before training a neural network?

A. No transformation needed
B. Log transformation
C. One-hot encoding
D. Min-Max scaling

<details>
<summary>Show Answer</summary>
<b>D. Min-Max scaling</b>

Neural networks require normalized inputs:
- Min-Max scaling transforms to [0, 1] range
- Prevents features with large values from dominating
- Improves gradient descent convergence

Log transformation (B) helps with skewness but may not be sufficient alone. One-hot (C) is for categorical data.
</details>

---

### Question 15
A company wants to track data lineage from source to model training. Which service provides built-in lineage tracking for data transformations?

A. AWS CloudTrail
B. AWS Glue Data Catalog
C. Amazon SageMaker ML Lineage Tracking
D. AWS Config

<details>
<summary>Show Answer</summary>
<b>C. Amazon SageMaker ML Lineage Tracking</b>

SageMaker ML Lineage Tracking automatically tracks:
- Data sources and transformations
- Training jobs and models
- Endpoints and model deployments
- Complete ML workflow lineage

CloudTrail (A) tracks API calls. Glue Catalog (B) stores metadata. Config (D) tracks resource configuration.
</details>

---

### Question 16
Which AWS service would you use to run SQL queries on data stored in S3 without loading it into a database?

A. Amazon RDS
B. Amazon Redshift
C. Amazon Athena
D. Amazon EMR

<details>
<summary>Show Answer</summary>
<b>C. Amazon Athena</b>

Athena is a serverless query service that:
- Runs SQL queries directly on S3 data
- No infrastructure or data loading required
- Integrates with Glue Data Catalog
- Pay per query (scanned data)

RDS (A) and Redshift (B) require data loading. EMR (C) requires cluster management.
</details>

---

### Question 17
A feature has values: [5, 10, 15, 1000]. What problem does the outlier (1000) potentially cause, and how should it be handled?

A. No problem; keep the value
B. May skew model training; use robust scaling or cap at 99th percentile
C. Always remove outliers
D. Replace with mean value

<details>
<summary>Show Answer</summary>
<b>B. May skew model training; use robust scaling or cap at 99th percentile</b>

Outliers can:
- Skew statistical measures (mean, standard deviation)
- Dominate gradient updates in neural networks
- Affect model performance

Robust scaling (uses median and IQR) or capping (Winsorization) handles outliers while preserving information. Always removing (C) may lose valuable anomalies. Mean replacement (D) doesn't address the skew.
</details>

---

### Question 18
Which SageMaker built-in algorithm is specifically designed for dimensionality reduction?

A. XGBoost
B. Linear Learner
C. PCA (Principal Component Analysis)
D. k-Means

<details>
<summary>Show Answer</summary>
<b>C. PCA (Principal Component Analysis)</b>

PCA is SageMaker's built-in algorithm for dimensionality reduction:
- Reduces feature count while preserving variance
- Creates uncorrelated principal components
- Useful for visualization and noise reduction

XGBoost (A) and Linear Learner (B) are for classification/regression. k-Means (D) is for clustering.
</details>

---

## Domain 2: ML Model Development (Questions 19-35)

### Question 19
Which SageMaker built-in algorithm is the most versatile for tabular data classification and regression?

A. Linear Learner
B. XGBoost
C. k-NN
D. Factorization Machines

<details>
<summary>Show Answer</summary>
<b>B. XGBoost</b>

XGBoost is the go-to algorithm for tabular data:
- Handles classification and regression
- Works with mixed data types
- Handles missing values automatically
- Excellent performance for most tabular problems
- Supports CPU and GPU training

Linear Learner (A) is for linear relationships. k-NN (C) is instance-based. Factorization Machines (D) is for sparse data.
</details>

---

### Question 20
A deep learning model is too large to fit in a single GPU's memory. Which distributed training strategy should be used?

A. Data parallelism
B. Model parallelism
C. Increase batch size
D. Use CPU instead of GPU

<details>
<summary>Show Answer</summary>
<b>B. Model parallelism</b>

Model parallelism splits the model across multiple GPUs:
- Different layers on different GPUs
- Enables training models larger than single GPU memory
- SageMaker Model Parallel library handles this automatically

Data parallelism (A) replicates the entire model, so it doesn't help with memory constraints. Increasing batch size (C) increases memory usage. CPU (D) would be very slow.
</details>

---

### Question 21
Which hyperparameter tuning strategy uses past training results to intelligently choose the next hyperparameter configuration?

A. Random search
B. Grid search
C. Bayesian optimization
D. Manual tuning

<details>
<summary>Show Answer</summary>
<b>C. Bayesian optimization</b>

Bayesian optimization:
- Uses probabilistic model of objective function
- Learns from previous training jobs
- Intelligently selects next hyperparameters
- Most efficient strategy (default in SageMaker)

Random (A) samples randomly. Grid (B) exhaustive search (not supported by SageMaker). Manual (D) is inefficient.
</details>

---

### Question 22
A hyperparameter tuning job has `max_jobs=30` and `max_parallel_jobs=5`. What does this configuration mean?

A. Run 30 jobs in parallel, maximum 5 total
B. Run 5 jobs in parallel, 30 jobs total
C. Run 30 jobs sequentially
D. Run 5 jobs total

<details>
<summary>Show Answer</summary>
<b>B. Run 5 jobs in parallel, 30 jobs total</b>

- `max_jobs`: Total number of training jobs (30)
- `max_parallel_jobs`: Maximum concurrent jobs (5)

This means 30 jobs will run total, with up to 5 running simultaneously.
</details>

---

### Question 23
Which metric should be maximized when the cost of false positives is very high, such as in spam email detection?

A. Recall
B. Precision
C. Accuracy
D. F1 Score

<details>
<summary>Show Answer</summary>
<b>B. Precision</b>

Precision = TP / (TP + FP) measures accuracy of positive predictions:
- High precision = low false positive rate
- Important when false positives are costly
- In spam detection: Don't want legitimate emails marked as spam

Recall (A) minimizes false negatives. Accuracy (C) can be misleading. F1 (D) balances both.
</details>

---

### Question 24
What is the primary purpose of using managed spot training in SageMaker?

A. Increase training speed
B. Reduce training costs by up to 90%
C. Improve model accuracy
D. Enable distributed training

<details>
<summary>Show Answer</summary>
<b>B. Reduce training costs by up to 90%</b>

Managed Spot Training:
- Uses EC2 Spot Instances for up to 90% cost savings
- SageMaker handles interruptions automatically
- Requires checkpointing to resume from interruptions
- Best for long-running, fault-tolerant training jobs

Speed (A), accuracy (C), and distributed training (D) are not the primary benefits.
</details>

---

### Question 25
Which SageMaker built-in algorithm should be used for time series forecasting with multiple related time series?

A. XGBoost
B. Linear Learner
C. DeepAR
D. Random Cut Forest

<details>
<summary>Show Answer</summary>
<b>C. DeepAR</b>

DeepAR is specifically designed for time series forecasting:
- Uses RNN architecture
- Learns patterns across multiple related time series
- Provides probabilistic forecasts
- Handles missing data and varying lengths

XGBoost (A) can do time series but isn't specialized. Linear Learner (B) is for classification/regression. RCF (D) is for anomaly detection.
</details>

---

### Question 26
What does enabling `early_stopping_type='Auto'` in hyperparameter tuning do?

A. Stops all jobs after a fixed time
B. Automatically stops poorly performing jobs to save time and cost
C. Stops the best performing job early
D. Prevents any jobs from running

<details>
<summary>Show Answer</summary>
<b>B. Automatically stops poorly performing jobs to save time and cost</b>

Early stopping:
- SageMaker monitors job performance
- Terminates jobs unlikely to outperform current best
- Reduces tuning time and cost
- Uses statistical analysis to make stopping decisions

Does not stop after fixed time (A), stop best jobs (C), or prevent execution (D).
</details>

---

### Question 27
Which training mode streams data from S3 during training instead of downloading the entire dataset first?

A. File mode
B. Pipe mode
C. Fast File mode
D. Stream mode

<details>
<summary>Show Answer</summary>
<b>B. Pipe mode</b>

Pipe mode:
- Streams data from S3 during training
- Reduces training startup time
- Better for large datasets
- Lower disk space requirements

File mode (A) downloads entire dataset. Fast File mode (C) uses lazy loading. Stream mode (D) is not a SageMaker mode.
</details>

---

### Question 28
What is the purpose of SageMaker Experiments?

A. Run A/B tests in production
B. Track and compare multiple training runs
C. Deploy models to different endpoints
D. Monitor model drift

<details>
<summary>Show Answer</summary>
<b>B. Track and compare multiple training runs</b>

SageMaker Experiments:
- Organizes training runs into experiments
- Tracks metrics, parameters, and artifacts
- Enables comparison of different runs
- Provides visualization and lineage tracking

A/B testing (A) is for deployment. Deployment (C) is handled by endpoints. Drift monitoring (D) is Model Monitor.
</details>

---

### Question 29
Which instance type should be used for training a computer vision model with a ResNet architecture on a large dataset?

A. ml.m5.xlarge (CPU)
B. ml.c5.2xlarge (compute optimized CPU)
C. ml.p3.2xlarge (GPU)
D. ml.t3.medium (burstable CPU)

<details>
<summary>Show Answer</summary>
<b>C. ml.p3.2xlarge (GPU)</b>

Deep learning for computer vision requires GPU:
- ResNet is a deep convolutional neural network
- GPUs accelerate matrix operations
- ml.p3 instances have NVIDIA V100 GPUs
- Much faster than CPU for deep learning

CPU instances (A, B, D) would be very slow for deep learning.
</details>

---

### Question 30
What is the difference between training data and validation data?

A. They are the same
B. Training data trains the model, validation data tunes hyperparameters and monitors overfitting
C. Validation data is always larger than training data
D. Training data is for testing, validation data is for training

<details>
<summary>Show Answer</summary>
<b>B. Training data trains the model, validation data tunes hyperparameters and monitors overfitting</b>

- Training data: Used to fit model parameters (weights, biases)
- Validation data: Used during training to evaluate performance and tune hyperparameters
- Test data: Final evaluation on unseen data

Validation is typically smaller (C is wrong). Roles are not reversed (D is wrong).
</details>

---

### Question 31
Which SageMaker service provides bias detection and model explainability using SHAP values?

A. SageMaker Debugger
B. SageMaker Clarify
C. SageMaker Model Monitor
D. SageMaker Autopilot

<details>
<summary>Show Answer</summary>
<b>B. SageMaker Clarify</b>

SageMaker Clarify provides:
- Bias detection (pre-training and post-training)
- Model explainability using SHAP values
- Feature importance analysis
- Fairness metrics

Debugger (A) is for training debugging. Model Monitor (C) is for drift detection. Autopilot (D) is for AutoML.
</details>

---

### Question 32
What does the learning rate hyperparameter control in model training?

A. How many epochs to train
B. The step size for weight updates during gradient descent
C. The size of the training dataset
D. The number of layers in the network

<details>
<summary>Show Answer</summary>
<b>B. The step size for weight updates during gradient descent</b>

Learning rate controls:
- How much weights are updated in each iteration
- Small learning rate: slow convergence, stable
- Large learning rate: fast convergence, may overshoot
- Critical hyperparameter for training success

Does not control epochs (A), dataset size (C), or architecture (D).
</details>

---

### Question 33
A model achieves 99% accuracy on the training set but only 70% on the test set. What problem does this indicate?

A. Underfitting
B. Overfitting
C. Data drift
D. Concept drift

<details>
<summary>Show Answer</summary>
<b>B. Overfitting</b>

Overfitting occurs when:
- Model learns training data too well (99% accuracy)
- Fails to generalize to new data (70% test accuracy)
- High variance problem

Solutions: Regularization, more data, simpler model, dropout.

Underfitting (A) would show poor performance on both sets. Drift (C, D) is for production models.
</details>

---

### Question 34
Which regularization technique randomly drops neurons during training to prevent overfitting in neural networks?

A. L1 regularization
B. L2 regularization
C. Dropout
D. Batch normalization

<details>
<summary>Show Answer</summary>
<b>C. Dropout</b>

Dropout:
- Randomly deactivates neurons during training
- Forces network to learn robust features
- Prevents co-adaptation of neurons
- Common in deep learning

L1 (A) and L2 (B) add penalty to loss function. Batch normalization (D) normalizes activations.
</details>

---

### Question 35
What is the purpose of the validation set during hyperparameter tuning?

A. Train the model
B. Evaluate different hyperparameter configurations
C. Final model evaluation
D. Deploy the model

<details>
<summary>Show Answer</summary>
<b>B. Evaluate different hyperparameter configurations</b>

Validation set is used to:
- Compare models with different hyperparameters
- Select best hyperparameter configuration
- Prevent overfitting to test set

Training set (A) fits the model. Test set (C) for final evaluation. Deployment (D) uses endpoints.
</details>

---

## Domain 3: Deployment and Orchestration (Questions 36-49)

### Question 36
Which SageMaker deployment option is most cost-effective for intermittent traffic with unpredictable patterns?

A. Real-time endpoint with fixed instances
B. Serverless inference
C. Batch Transform
D. Asynchronous inference

<details>
<summary>Show Answer</summary>
<b>B. Serverless inference</b>

Serverless inference:
- Scales to zero when not in use (no idle cost)
- Pay only for compute time and data processed
- Ideal for intermittent, unpredictable traffic
- Auto-scales for traffic spikes

Real-time endpoints (A) cost money when idle. Batch Transform (C) is for scheduled batch jobs. Async (D) is for large payloads.
</details>

---

### Question 37
A company needs to perform predictions on 5 million records once per week. Which deployment pattern is most appropriate?

A. Real-time endpoint
B. Serverless inference
C. Batch Transform
D. Asynchronous inference

<details>
<summary>Show Answer</summary>
<b>C. Batch Transform</b>

Batch Transform is ideal for:
- Large-scale, periodic inference
- Processing millions of records
- No need for persistent endpoint
- Cost-effective (no always-on infrastructure)

Real-time (A) would be expensive for weekly use. Serverless (B) better for request/response. Async (D) for individual large payloads.
</details>

---

### Question 38
What is the primary benefit of using a multi-model endpoint?

A. Faster inference speed
B. Better model accuracy
C. Cost reduction by hosting multiple models on shared infrastructure
D. Automatic model retraining

<details>
<summary>Show Answer</summary>
<b>C. Cost reduction by hosting multiple models on shared infrastructure</b>

Multi-model endpoints:
- Host hundreds or thousands of models on one endpoint
- Share compute resources across models
- Dynamically load/unload models
- Significant cost savings for many-model scenarios

Does not improve speed (A), accuracy (B), or enable retraining (D).
</details>

---

### Question 39
Which deployment strategy allows instant rollback by switching traffic back to the previous version?

A. All-at-once deployment
B. Blue/Green deployment
C. In-place update
D. Canary deployment

<details>
<summary>Show Answer</summary>
<b>B. Blue/Green deployment</b>

Blue/Green deployment:
- New version deployed alongside old version
- Traffic switched when new version is validated
- Instant rollback by switching traffic back
- Zero downtime

All-at-once (A) has downtime and difficult rollback. In-place (C) overwrites current version. Canary (D) gradually shifts traffic.
</details>

---

### Question 40
What is the maximum payload size supported by asynchronous inference?

A. 6 MB
B. 100 MB
C. 1 GB
D. 10 GB

<details>
<summary>Show Answer</summary>
<b>C. 1 GB</b>

Asynchronous inference supports:
- Up to 1 GB payload size
- Large inputs (images, videos, documents)
- Queued processing via SQS
- SNS notifications on completion

Real-time endpoints have 6 MB limit (A).
</details>

---

### Question 41
Which SageMaker service automatically benchmarks models on different instance types and provides cost-performance recommendations?

A. SageMaker Debugger
B. SageMaker Model Monitor
C. SageMaker Inference Recommender
D. SageMaker Neo

<details>
<summary>Show Answer</summary>
<b>C. SageMaker Inference Recommender</b>

Inference Recommender:
- Automatically benchmarks instance types
- Measures latency, throughput, cost
- Provides ranked recommendations
- Helps optimize deployment configuration

Debugger (A) for training. Model Monitor (B) for drift. Neo (D) for compilation.
</details>

---

### Question 42
What is the purpose of SageMaker Pipelines?

A. Data ingestion only
B. Model deployment only
C. Orchestrate end-to-end ML workflows
D. Monitor model performance

<details>
<summary>Show Answer</summary>
<b>C. Orchestrate end-to-end ML workflows</b>

SageMaker Pipelines:
- Automate ML workflow from data to deployment
- Native SageMaker integration
- Steps: Processing, Training, Evaluation, Condition, Register, Deploy
- Lineage tracking and versioning

Not limited to ingestion (A) or deployment (B). Monitoring (D) is Model Monitor.
</details>

---

### Question 43
Which Step type in SageMaker Pipelines allows conditional execution based on model performance?

A. Processing Step
B. Training Step
C. Condition Step
D. Transform Step

<details>
<summary>Show Answer</summary>
<b>C. Condition Step</b>

Condition Step:
- Evaluates conditions (e.g., accuracy > 0.9)
- Executes different steps based on result
- Enables approval workflows
- Common for model registration gates

Other steps (A, B, D) are not conditional.
</details>

---

### Question 44
What does endpoint auto-scaling based on `InvocationsPerInstance` do?

A. Scales based on model accuracy
B. Scales based on number of requests per instance
C. Scales based on data drift
D. Scales based on training time

<details>
<summary>Show Answer</summary>
<b>B. Scales based on number of requests per instance</b>

InvocationsPerInstance metric:
- Counts requests to each instance
- Auto-scaling adjusts instance count to maintain target
- Example: Scale up if > 1000 invocations/instance
- Balances cost and performance

Not based on accuracy (A), drift (C), or training (D).
</details>

---

### Question 45
Which service optimizes ML models for deployment on specific hardware (CPU, GPU, edge devices)?

A. SageMaker Debugger
B. SageMaker Neo
C. SageMaker Autopilot
D. SageMaker Clarify

<details>
<summary>Show Answer</summary>
<b>B. SageMaker Neo</b>

SageMaker Neo:
- Compiles models for specific hardware
- Optimizes for cloud instances or edge devices
- Supports TensorFlow, PyTorch, MXNet, etc.
- Can improve inference speed up to 2x

Debugger (A) for training. Autopilot (C) for AutoML. Clarify (D) for bias/explainability.
</details>

---

### Question 46
What is an inference pipeline in SageMaker?

A. Multiple models in sequence
B. Multiple endpoints for load balancing
C. Chain of containers for preprocessing and inference
D. Batch processing workflow

<details>
<summary>Show Answer</summary>
<b>C. Chain of containers for preprocessing and inference</b>

Inference Pipeline:
- Chains up to 15 containers in sequence
- Preprocessing → Feature engineering → Inference
- Example: SparkML → scikit-learn → XGBoost
- Simplifies deployment of complex workflows

Not just multiple models (A), not load balancing (B), not batch processing (D).
</details>

---

### Question 47
When should you use AWS Step Functions instead of SageMaker Pipelines for ML workflows?

A. Always use Step Functions
B. When you need integration with many non-SageMaker AWS services
C. Never use Step Functions for ML
D. Only for training jobs

<details>
<summary>Show Answer</summary>
<b>B. When you need integration with many non-SageMaker AWS services</b>

Use Step Functions when:
- Complex workflows with Lambda, EMR, Glue, etc.
- Need fine-grained control over state transitions
- Integration with many AWS services

Use SageMaker Pipelines for:
- ML-specific workflows
- Native SageMaker integration
- Built-in lineage tracking
</details>

---

### Question 48
What does the Model Registry approval status "PendingManualApproval" indicate?

A. Model is automatically approved
B. Model is waiting for human approval before deployment
C. Model is rejected
D. Model is already deployed

<details>
<summary>Show Answer</summary>
<b>B. Model is waiting for human approval before deployment</b>

Approval statuses:
- PendingManualApproval: Requires human review
- Approved: Ready for deployment
- Rejected: Not approved for deployment

Enables governance and review processes.
</details>

---

### Question 49
Which deployment strategy gradually increases traffic to a new model version while monitoring performance?

A. All-at-once
B. Blue/Green
C. Canary
D. Rolling

<details>
<summary>Show Answer</summary>
<b>C. Canary</b>

Canary deployment:
- Starts with small traffic percentage (e.g., 5%)
- Monitors new version performance
- Gradually increases traffic if successful
- Reduces risk of widespread issues

All-at-once (A) switches immediately. Blue/Green (B) switches 100% at once. Rolling (D) updates instances sequentially.
</details>

---

## Domain 4: Monitoring, Maintenance, Security (Questions 50-65)

### Question 50
Which type of Model Monitor detects changes in input feature distributions?

A. Model Quality Monitoring
B. Data Quality Monitoring
C. Bias Drift Monitoring
D. Feature Attribution Monitoring

<details>
<summary>Show Answer</summary>
<b>B. Data Quality Monitoring</b>

Data Quality Monitoring:
- Compares production input data to baseline
- Detects distribution changes
- Monitors missing values, data types
- Alerts on constraint violations

Model Quality (A) monitors prediction accuracy. Bias Drift (C) monitors fairness. Feature Attribution (D) monitors importance.
</details>

---

### Question 51
What must be enabled on a SageMaker endpoint before Model Monitor can analyze production data?

A. Auto-scaling
B. Data capture
C. VPC mode
D. Encryption

<details>
<summary>Show Answer</summary>
<b>B. Data capture</b>

Data capture:
- Records inference requests and responses
- Stores in S3 for analysis
- Required for all Model Monitor types
- Configure sampling percentage

Auto-scaling (A) is for performance. VPC (C) and encryption (D) are security features.
</details>

---

### Question 52
A model's accuracy on production data has dropped from 92% to 78%, but input feature distributions remain similar to training data. What type of drift is this?

A. Data drift
B. Concept drift
C. Label drift
D. No drift

<details>
<summary>Show Answer</summary>
<b>B. Concept drift</b>

Concept drift:
- Relationship between features and target changes
- Input distributions stay the same
- Causes performance degradation
- Requires model retraining

Data drift (A) would show input distribution changes. Label drift (C) is target distribution change.
</details>

---

### Question 53
What is the recommended retraining strategy when drift is detected in a production model?

A. Ignore the drift
B. Immediately retrain with all historical data
C. Retrain with recent data, evaluate performance, deploy if better than current model
D. Delete the current model

<details>
<summary>Show Answer</summary>
<b>C. Retrain with recent data, evaluate performance, deploy if better than current model</b>

Best practice workflow:
1. Detect drift
2. Retrain with recent/relevant data
3. Evaluate new model
4. Deploy only if performance improves
5. Use champion/challenger or canary deployment

Always evaluate before deploying (B is incomplete). Never ignore drift (A). Don't delete without replacement (D).
</details>

---

### Question 54
Which IAM best practice should be applied to SageMaker execution roles?

A. Grant full administrator access
B. Use root account credentials
C. Apply principle of least privilege
D. Share roles across all users

<details>
<summary>Show Answer</summary>
<b>C. Apply principle of least privilege</b>

Least privilege means:
- Grant minimum permissions needed
- Only necessary S3 buckets, ECR repos
- Specific actions, not wildcards
- Reduces security risk

Never use admin (A) or root (B). Sharing roles (D) violates least privilege.
</details>

---

### Question 55
What does enabling VPC mode with `enable_network_isolation=True` do?

A. Enables internet access
B. Provides complete network isolation with no inbound/outbound calls
C. Only blocks inbound traffic
D. Only blocks outbound traffic

<details>
<summary>Show Answer</summary>
<b>B. Provides complete network isolation with no inbound/outbound calls</b>

Network isolation mode:
- No network access during training or inference
- Training data must be in S3
- Maximum security for sensitive data
- Prevents data exfiltration

Not partial isolation (C, D). Blocks all network access (not enables, A).
</details>

---

### Question 56
Which AWS service tracks all SageMaker API calls for audit and compliance?

A. Amazon CloudWatch
B. AWS CloudTrail
C. AWS Config
D. Amazon Inspector

<details>
<summary>Show Answer</summary>
<b>B. AWS CloudTrail</b>

CloudTrail:
- Logs all API calls
- Tracks who, what, when, where
- Compliance and audit trail
- Integrates with CloudWatch Logs

CloudWatch (A) for metrics. Config (C) for resource configuration. Inspector (D) for security assessments.
</details>

---

### Question 57
What type of encryption does SageMaker use for data in transit?

A. AES-256
B. TLS 1.2 or higher
C. RSA
D. No encryption

<details>
<summary>Show Answer</summary>
<b>B. TLS 1.2 or higher</b>

SageMaker encryption:
- In transit: TLS 1.2+ for all communications
- At rest: AES-256 (S3, EBS)
- Inter-container: Optional encryption

AES-256 (A) is for at-rest. TLS is the in-transit protocol.
</details>

---

### Question 58
Which metric indicates the latency of the model inference itself, excluding SageMaker overhead?

A. Invocations
B. ModelLatency
C. OverheadLatency
D. Invocation4XXErrors

<details>
<summary>Show Answer</summary>
<b>B. ModelLatency</b>

SageMaker endpoint metrics:
- ModelLatency: Time for model inference
- OverheadLatency: SageMaker processing overhead
- Total latency = ModelLatency + OverheadLatency

Invocations (A) counts requests. Errors (D) count failures.
</details>

---

### Question 59
What is the purpose of point-in-time queries in SageMaker Feature Store?

A. Real-time predictions
B. Prevent data leakage by retrieving features as they existed at a specific time
C. Delete old features
D. Batch processing

<details>
<summary>Show Answer</summary>
<b>B. Prevent data leakage by retrieving features as they existed at a specific time</b>

Point-in-time queries:
- Retrieve features as they existed historically
- Prevent using future data for past predictions
- Critical for time-series and temporal models
- Ensures training data integrity

Not for predictions (A), deletion (C), or batch processing (D).
</details>

---

### Question 60
Which deployment pattern maintains two versions of a model with weighted traffic distribution for comparison?

A. Blue/Green
B. Canary
C. Champion/Challenger
D. All-at-once

<details>
<summary>Show Answer</summary>
<b>C. Champion/Challenger</b>

Champion/Challenger:
- Two model versions (current vs new)
- Split traffic (e.g., 90/10)
- Compare performance metrics
- Promote challenger if better

Blue/Green (A) switches traffic. Canary (B) gradually increases. All-at-once (D) replaces immediately.
</details>

---

### Question 61
What does the baseline in Model Monitor represent?

A. Production data statistics
B. Statistics from training data for comparison
C. Model accuracy threshold
D. Cost budget

<details>
<summary>Show Answer</summary>
<b>B. Statistics from training data for comparison</b>

Baseline:
- Created from training data
- Statistical reference (mean, std, distributions)
- Production data compared against baseline
- Deviations trigger violations

Not production stats (A), accuracy threshold (C), or cost (D).
</details>

---

### Question 62
Which KMS feature allows SageMaker to encrypt model artifacts and output data?

A. Customer Managed Keys (CMK)
B. AWS Managed Keys only
C. No encryption available
D. Public keys

<details>
<summary>Show Answer</summary>
<b>A. Customer Managed Keys (CMK)</b>

KMS integration:
- Use Customer Managed Keys for encryption
- Encrypt training volumes, model artifacts, outputs
- Full control over key policies and rotation
- Can also use AWS managed keys

Not limited to AWS managed (B). Encryption is available (C). Not public keys (D).
</details>

---

### Question 63
What is the difference between data drift and concept drift?

A. They are the same thing
B. Data drift is input distribution change; concept drift is relationship change between inputs and outputs
C. Data drift is worse than concept drift
D. Concept drift only affects regression models

<details>
<summary>Show Answer</summary>
<b>B. Data drift is input distribution change; concept drift is relationship change between inputs and outputs</b>

- Data drift: Feature distributions change (e.g., customer age increases)
- Concept drift: Input-output relationship changes (e.g., income threshold for approval changes)

Both require different detection and remediation strategies. Neither is universally worse (C). Affects all model types (D).
</details>

---

### Question 64
Which SageMaker feature provides pre-built MLOps templates with CI/CD pipelines?

A. SageMaker Pipelines
B. SageMaker Projects
C. SageMaker Studio
D. SageMaker Experiments

<details>
<summary>Show Answer</summary>
<b>B. SageMaker Projects</b>

SageMaker Projects:
- Pre-built MLOps templates
- Integrates CodePipeline, CodeCommit, CodeBuild
- Automated model building and deployment
- Best practice workflows

Pipelines (A) orchestrates workflows. Studio (C) is IDE. Experiments (D) tracks runs.
</details>

---

### Question 65
For HIPAA compliance with highly sensitive healthcare data, which SageMaker configuration is recommended?

A. Default configuration
B. VPC mode with network isolation, KMS encryption, and private subnets
C. Public endpoints with encryption
D. No special configuration needed

<details>
<summary>Show Answer</summary>
<b>B. VPC mode with network isolation, KMS encryption, and private subnets</b>

HIPAA-compliant configuration:
- VPC mode isolates resources
- Network isolation prevents data exfiltration
- KMS encryption for all data at rest and in transit
- Private subnets with no internet access
- VPC endpoints for AWS service access

Default config (A) lacks controls. Public endpoints (C) increase risk. Special config required (D is wrong).
</details>

---

## Exam Summary

Congratulations on completing the practice exam!

### Scoring Guide

| Score Range | Assessment |
|-------------|------------|
| 58-65 (90%+) | Excellent! You're well-prepared |
| 52-57 (80-89%) | Good! Review weak areas |
| 47-51 (72-79%) | Passing range - more study needed |
| Below 47 | Study all domains thoroughly |

### Study Recommendations

Focus on these key areas based on your performance:

1. **Data Preparation**: Feature Store, Glue, data formats, Feature engineering
2. **Model Development**: XGBoost, hyperparameter tuning, distributed training
3. **Deployment**: Endpoint types, deployment strategies, inference optimization
4. **Monitoring & Security**: Model Monitor types, drift detection, VPC, encryption, IAM

### Next Steps

1. Review incorrect answers and understand why
2. Study official AWS documentation for weak areas
3. Practice hands-on labs in AWS Console/SageMaker Studio
4. Review the domain lesson pages
5. Take this practice exam again after additional study

Good luck with your AWS Certified Machine Learning Engineer - Associate exam!

---

## Navigation

| Previous | Next |
|----------|------|
| [Domain 4: Monitoring and Security](/posts/aws-mla-c01-domain-4-monitoring-security/) | [Hands-on Labs](/posts/aws-mla-c01-hands-on-labs/) |
