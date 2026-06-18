---
title: "AWS MLA-C01 - Domain 1: Data Preparation for Machine Learning"
author: thanhnv1808
date: 2026-01-24 10:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, data-preparation, sagemaker, etl]
description: "Domain 1 covers 28% of the exam (~18 questions). Master data ingestion, transformation, feature engineering, and data validation for ML workloads."
pin: false
comments: true
---

## Domain 1 Overview

**Exam Weight: 28% (~18 questions)**

This domain focuses on preparing data for machine learning workloads, including ingestion, transformation, feature engineering, and data validation.

### Task Statements

| Task | Description |
|------|-------------|
| 1.1 | Ingest and store data for ML workloads |
| 1.2 | Transform data and perform feature engineering |
| 1.3 | Ensure data quality and manage data lifecycle |

---

## Task 1.1: Ingest and Store Data for ML Workloads

### Data Ingestion Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA INGESTION PATTERNS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Batch Ingestion          Streaming Ingestion     Real-time    │
│   ┌─────────────┐          ┌─────────────┐        ┌──────────┐  │
│   │   S3       │          │  Kinesis    │        │ API      │  │
│   │   Glue     │          │  Data       │        │ Gateway  │  │
│   │   EMR      │          │  Streams    │        │ Lambda   │  │
│   │   Batch    │          │  Firehose   │        │ AppSync  │  │
│   └─────────────┘          └─────────────┘        └──────────┘  │
│         │                        │                     │        │
│         └────────────────────────┴─────────────────────┘        │
│                              │                                   │
│                    ┌─────────▼─────────┐                        │
│                    │   Data Lake/      │                        │
│                    │   Warehouse       │                        │
│                    │   (S3/Redshift)   │                        │
│                    └───────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

### AWS Data Storage Services for ML

| Service | Use Case | Data Format | Best For |
|---------|----------|-------------|----------|
| **Amazon S3** | Data lake storage | Any format | Raw data, training datasets, model artifacts |
| **Amazon Redshift** | Data warehousing | Structured | Analytics, aggregated features |
| **Amazon DynamoDB** | NoSQL database | Key-value, document | Real-time feature serving |
| **Amazon RDS/Aurora** | Relational database | Structured | Transactional data |
| **Amazon ElastiCache** | In-memory cache | Key-value | Low-latency feature retrieval |
| **Amazon FSx for Lustre** | High-performance file system | Any | High-throughput training data |

### S3 Data Organization for ML

```
s3://ml-data-bucket/
├── raw/                          # Original unprocessed data
│   ├── ingestion_date=2024-01-01/
│   └── ingestion_date=2024-01-02/
├── processed/                    # Cleaned and transformed data
│   ├── train/
│   ├── validation/
│   └── test/
├── features/                     # Feature store exports
│   └── feature_group_name/
├── models/                       # Trained model artifacts
│   └── model_version/
└── inference/                    # Inference results
    └── batch_predictions/
```

### Amazon Kinesis Family

| Service | Purpose | Key Features |
|---------|---------|--------------|
| **Kinesis Data Streams** | Real-time data streaming | Custom consumers, multiple consumers, 24h-365 day retention |
| **Kinesis Data Firehose** | Data delivery | Near real-time, auto-scaling, built-in transformations |
| **Kinesis Data Analytics** | Stream processing | SQL/Flink, real-time analytics, anomaly detection |
| **Kinesis Video Streams** | Video streaming | WebRTC, playback, ML integration |

### Data Format Considerations

| Format | Pros | Cons | Best For |
|--------|------|------|----------|
| **Parquet** | Columnar, compressed, schema | Not human-readable | Large datasets, analytics |
| **CSV** | Human-readable, universal | No schema, inefficient | Small datasets, data exchange |
| **JSON** | Flexible, semi-structured | Verbose, slow parsing | API data, logs |
| **Avro** | Schema evolution, compact | Less tool support | Event streaming |
| **ORC** | Optimized for Hive | Limited ecosystem | Hadoop workloads |
| **RecordIO** | Efficient for SageMaker | Proprietary format | SageMaker training |

> **Exam Tip**: Parquet is the preferred format for large-scale ML workloads due to columnar storage, efficient compression, and schema support.
{: .prompt-tip }

### AWS Glue for Data Ingestion

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS GLUE COMPONENTS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐     │
│   │   Glue      │    │   Glue      │    │   Glue ETL      │     │
│   │   Crawlers  │───▶│   Data      │───▶│   Jobs          │     │
│   │             │    │   Catalog   │    │   (Spark/Python)│     │
│   └─────────────┘    └─────────────┘    └─────────────────┘     │
│                                                │                 │
│   ┌─────────────┐    ┌─────────────┐          ▼                 │
│   │   Glue      │    │   Glue      │    ┌─────────────────┐     │
│   │   DataBrew  │    │   Workflows │    │   Target Data   │     │
│   │   (No-code) │    │             │    │   Store         │     │
│   └─────────────┘    └─────────────┘    └─────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Glue Job Types

| Job Type | Language | Use Case | Scalability |
|----------|----------|----------|-------------|
| **Spark ETL** | PySpark/Scala | Large-scale transformations | Auto-scaling DPUs |
| **Python Shell** | Python | Light transformations, APIs | Fixed resources |
| **Streaming** | PySpark | Real-time processing | Continuous |
| **Ray** | Python | ML workloads, distributed | Auto-scaling |

---

## Task 1.2: Transform Data and Perform Feature Engineering

### Data Transformation Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                  DATA TRANSFORMATION PIPELINE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Raw Data → Cleaning → Transformation → Feature Engineering     │
│                                                                  │
│  ┌─────────┐   ┌─────────────┐   ┌──────────────┐   ┌────────┐  │
│  │ Missing │   │ Normalization│   │ Encoding     │   │Feature │  │
│  │ Values  │   │ Scaling      │   │ Categorical  │   │Store   │  │
│  │ Outliers│   │ Log Transform│   │ Text/Image   │   │        │  │
│  └─────────┘   └─────────────┘   └──────────────┘   └────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Handling Missing Values

| Strategy | When to Use | AWS Implementation |
|----------|-------------|-------------------|
| **Mean/Median Imputation** | Numerical, random missing | SageMaker Data Wrangler, Glue DataBrew |
| **Mode Imputation** | Categorical variables | SageMaker Data Wrangler |
| **Forward/Backward Fill** | Time series data | Glue ETL, Pandas |
| **KNN Imputation** | Complex patterns | SageMaker Processing |
| **Drop Rows** | Small % missing, MCAR | Any ETL tool |
| **Indicator Variable** | Missingness is informative | Custom transformation |

### Feature Scaling Techniques

| Technique | Formula | When to Use |
|-----------|---------|-------------|
| **Min-Max Scaling** | (x - min) / (max - min) | Neural networks, bounded features |
| **Standard Scaling** | (x - mean) / std | Linear models, SVM, PCA |
| **Robust Scaling** | (x - median) / IQR | Data with outliers |
| **Log Transformation** | log(x + 1) | Right-skewed distributions |
| **Box-Cox** | Power transformation | Normalize distributions |

### Categorical Encoding Methods

| Method | Output | Use Case | Cardinality |
|--------|--------|----------|-------------|
| **One-Hot Encoding** | Binary columns | Nominal categories | Low |
| **Label Encoding** | Integer values | Ordinal categories | Any |
| **Target Encoding** | Target mean | High cardinality | High |
| **Frequency Encoding** | Count/frequency | Tree-based models | High |
| **Embedding** | Dense vectors | Deep learning, NLP | Very High |
| **Binary Encoding** | Binary representation | Medium cardinality | Medium |

### Amazon SageMaker Data Wrangler

```
┌─────────────────────────────────────────────────────────────────┐
│                   SAGEMAKER DATA WRANGLER                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Data Sources          Transformations           Export Options │
│  ─────────────         ───────────────           ────────────── │
│  • S3                  • 300+ built-in           • S3           │
│  • Athena              • Custom (Python/Spark)   • Feature Store│
│  • Redshift            • ML transforms           • Pipeline     │
│  • Snowflake           • Data quality checks     • Python code  │
│  • Databricks          • Visualization           • Spark code   │
│                                                                  │
│  Features:                                                       │
│  • Visual interface for data preparation                        │
│  • Quick Model for feature importance analysis                  │
│  • Data quality and insights reports                            │
│  • Export to SageMaker Processing, Pipeline, or Feature Store   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Amazon SageMaker Feature Store

| Component | Description |
|-----------|-------------|
| **Feature Group** | Collection of features with shared metadata |
| **Online Store** | Low-latency retrieval for real-time inference |
| **Offline Store** | S3-based storage for batch training |
| **Feature Definition** | Schema defining feature name and type |
| **Record Identifier** | Unique key for each record |
| **Event Time** | Timestamp for point-in-time queries |

```python
# Creating a Feature Group
from sagemaker.feature_store.feature_group import FeatureGroup

feature_group = FeatureGroup(
    name="customer-features",
    sagemaker_session=session
)

feature_group.load_feature_definitions(data_frame=df)

feature_group.create(
    s3_uri=f"s3://{bucket}/feature-store/",
    record_identifier_name="customer_id",
    event_time_feature_name="event_time",
    role_arn=role,
    enable_online_store=True
)
```

### Feature Engineering Techniques

| Technique | Description | Example |
|-----------|-------------|---------|
| **Binning** | Convert continuous to categorical | Age → Age groups |
| **Polynomial Features** | Create interaction terms | x₁ * x₂, x₁² |
| **Date/Time Extraction** | Extract components | Day of week, month, hour |
| **Aggregations** | Statistical summaries | Mean, sum, count by group |
| **Window Functions** | Rolling calculations | 7-day moving average |
| **Text Vectorization** | Convert text to numbers | TF-IDF, word embeddings |
| **Dimensionality Reduction** | Reduce features | PCA, t-SNE |

### SageMaker Built-in Algorithms for Feature Engineering

| Algorithm | Purpose | Output |
|-----------|---------|--------|
| **PCA** | Dimensionality reduction | Principal components |
| **Random Cut Forest** | Anomaly detection | Anomaly scores |
| **Object2Vec** | Embedding generation | Dense vectors |
| **BlazingText** | Word embeddings | Word vectors |
| **IP Insights** | IP address analysis | Risk scores |

> **Exam Tip**: SageMaker Feature Store provides both online (real-time) and offline (batch) feature retrieval with automatic synchronization.
{: .prompt-tip }

---

## Task 1.3: Ensure Data Quality and Manage Data Lifecycle

### Data Quality Dimensions

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA QUALITY DIMENSIONS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │ Completeness│   │  Accuracy   │   │ Consistency │          │
│   │ No missing  │   │ Correct     │   │ Same across │          │
│   │ values      │   │ values      │   │ systems     │          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│   │  Timeliness │   │  Validity   │   │  Uniqueness │          │
│   │ Up-to-date  │   │ Conforms to │   │ No duplicate│          │
│   │ data        │   │ rules       │   │ records     │          │
│   └─────────────┘   └─────────────┘   └─────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### AWS Data Quality Tools

| Tool | Purpose | Key Features |
|------|---------|--------------|
| **AWS Glue Data Quality** | Rule-based validation | DQDL rules, automatic recommendations |
| **SageMaker Data Wrangler** | Visual data analysis | Quality reports, statistics |
| **Amazon Deequ** | Statistical validation | Unit tests for data, anomaly detection |
| **Great Expectations** | Data validation framework | Expectations, documentation |

### Glue Data Quality Definition Language (DQDL)

```sql
-- Example DQDL Rules
Rules = [
    -- Completeness checks
    Completeness "customer_id" > 0.99,
    IsComplete "email",

    -- Uniqueness checks
    IsPrimaryKey "order_id",
    IsUnique "transaction_id",

    -- Validity checks
    ColumnDataType "age" = "INT",
    ColumnValues "status" in ["active", "inactive", "pending"],
    ColumnLength "phone" = 10,

    -- Range checks
    ColumnValues "age" between 0 and 120,
    ColumnValues "price" > 0,

    -- Pattern matching
    ColumnValues "email" matches "^[a-zA-Z0-9+_.-]+@[a-zA-Z0-9.-]+$",

    -- Statistical checks
    Mean "order_value" between 50 and 200,
    StandardDeviation "price" < 100,

    -- Referential integrity
    ReferentialIntegrity "customer_id" "customers"."id" > 0.95
]
```

### Data Validation Strategies

| Strategy | Description | Implementation |
|----------|-------------|----------------|
| **Schema Validation** | Verify data structure | Glue Schema Registry, Athena |
| **Statistical Validation** | Check distributions | Deequ, Data Wrangler |
| **Business Rule Validation** | Apply business logic | DQDL, custom Lambda |
| **Anomaly Detection** | Identify outliers | Random Cut Forest, custom models |
| **Drift Detection** | Monitor distribution changes | SageMaker Model Monitor |

### Data Lineage and Versioning

```
┌─────────────────────────────────────────────────────────────────┐
│                      DATA LINEAGE TRACKING                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Source Data         Transformations          Output            │
│   ───────────         ───────────────          ──────            │
│   ┌─────────┐         ┌─────────────┐         ┌─────────┐       │
│   │ Raw S3  │────────▶│ Glue Job    │────────▶│Processed│       │
│   │ Data    │         │ v1.2        │         │ Data    │       │
│   └─────────┘         └─────────────┘         └─────────┘       │
│        │                    │                       │            │
│        ▼                    ▼                       ▼            │
│   ┌─────────────────────────────────────────────────────┐       │
│   │             AWS Glue Data Catalog                    │       │
│   │    - Table metadata    - Column lineage              │       │
│   │    - Schema versions   - Transform history           │       │
│   └─────────────────────────────────────────────────────┘       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### AWS Services for Data Governance

| Service | Purpose | Features |
|---------|---------|----------|
| **AWS Lake Formation** | Data lake governance | Fine-grained access, data sharing |
| **AWS Glue Data Catalog** | Metadata management | Schema registry, versioning |
| **Amazon DataZone** | Data discovery | Business catalog, data sharing |
| **AWS CloudTrail** | Audit logging | API call history |

### Data Lifecycle Management

| Stage | Actions | AWS Services |
|-------|---------|--------------|
| **Ingestion** | Collect, validate | Kinesis, Glue, S3 |
| **Storage** | Store, organize | S3, Redshift, RDS |
| **Processing** | Transform, enrich | Glue, EMR, SageMaker |
| **Analysis** | Query, visualize | Athena, QuickSight |
| **Archive** | Move to cold storage | S3 Glacier, lifecycle policies |
| **Deletion** | Secure removal | S3 lifecycle, retention policies |

### S3 Lifecycle Policies for ML Data

```json
{
    "Rules": [
        {
            "ID": "ArchiveOldTrainingData",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "training-data/"
            },
            "Transitions": [
                {
                    "Days": 90,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 365,
                    "StorageClass": "GLACIER"
                }
            ]
        },
        {
            "ID": "DeleteTempData",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "temp/"
            },
            "Expiration": {
                "Days": 7
            }
        }
    ]
}
```

> **Exam Tip**: Lake Formation provides column-level and row-level security for fine-grained access control to data lake resources.
{: .prompt-tip }

### Train/Validation/Test Split Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Random Split** | Random sampling | IID data |
| **Stratified Split** | Preserve class distribution | Imbalanced datasets |
| **Time-based Split** | Chronological ordering | Time series, avoid data leakage |
| **Group Split** | Keep groups together | Customer-level predictions |
| **K-Fold CV** | Multiple train/validation splits | Limited data, robust evaluation |

---

## Key Definitions

| Term | Definition |
|------|------------|
| **ETL** | Extract, Transform, Load - process of moving and transforming data |
| **Feature Engineering** | Creating new features from raw data to improve model performance |
| **Feature Store** | Centralized repository for storing and serving ML features |
| **Data Lineage** | Tracking data origin and transformations through the pipeline |
| **Data Drift** | Changes in data distribution over time |
| **Point-in-Time Query** | Retrieve features as they existed at a specific time |
| **Cardinality** | Number of unique values in a categorical variable |
| **Imputation** | Filling in missing values with estimated values |

---

## Practice Questions

### Question 1
A data scientist needs to prepare a large dataset with 500 columns for training. Many columns have high cardinality categorical variables. Which approach should they use for encoding?

A) One-hot encoding for all categorical variables
B) Label encoding for all categorical variables
C) Target encoding for high cardinality variables
D) Drop all categorical variables

<details>
<summary>View Answer</summary>

**Answer: C**

Target encoding is appropriate for high cardinality categorical variables as it:
- Avoids the dimensionality explosion of one-hot encoding
- Captures the relationship between the category and target
- Works well with tree-based models

One-hot encoding (A) would create too many columns for high cardinality variables. Label encoding (B) assumes ordinal relationships. Dropping variables (D) loses valuable information.

</details>

### Question 2
A company wants to serve features in real-time for their fraud detection model while also using the same features for batch training. Which AWS service should they use?

A) Amazon ElastiCache
B) Amazon DynamoDB
C) Amazon SageMaker Feature Store
D) Amazon Redshift

<details>
<summary>View Answer</summary>

**Answer: C**

SageMaker Feature Store provides:
- Online store for real-time feature retrieval (low-latency)
- Offline store in S3 for batch training
- Automatic synchronization between online and offline stores
- Point-in-time queries to prevent data leakage

Other options provide either real-time (ElastiCache, DynamoDB) or batch (Redshift) but not both with automatic synchronization.

</details>

### Question 3
A data engineer needs to validate that incoming data meets quality standards before processing. They want to use rules like "customer_age should be between 0 and 120" and "email should match a regex pattern". Which AWS service should they use?

A) Amazon SageMaker Model Monitor
B) AWS Glue Data Quality
C) Amazon Macie
D) AWS Config

<details>
<summary>View Answer</summary>

**Answer: B**

AWS Glue Data Quality with DQDL (Data Quality Definition Language) allows you to:
- Define rules for data validation
- Check completeness, uniqueness, and validity
- Apply regex patterns and range checks
- Integrate with Glue ETL jobs

SageMaker Model Monitor (A) is for monitoring deployed models, not data validation. Macie (C) is for sensitive data discovery. AWS Config (D) is for resource configuration compliance.

</details>

### Question 4
A team needs to process streaming data from IoT sensors and write the results to S3 for ML training. They want the simplest solution with automatic scaling. Which service should they use?

A) Amazon Kinesis Data Streams with Lambda
B) Amazon Kinesis Data Firehose
C) Amazon MSK (Managed Streaming for Kafka)
D) Amazon SQS with EC2 consumers

<details>
<summary>View Answer</summary>

**Answer: B**

Kinesis Data Firehose is the simplest solution because it:
- Automatically handles scaling
- Has built-in delivery to S3
- Supports data transformation with Lambda
- Requires no consumer management
- Is fully serverless

Other options require more operational overhead for consumer management and scaling.

</details>

### Question 5
A data scientist is working with time series data for demand forecasting. How should they split the data for training and validation?

A) Random 80/20 split
B) Stratified sampling based on demand levels
C) Chronological split with earlier data for training
D) K-fold cross-validation

<details>
<summary>View Answer</summary>

**Answer: C**

For time series data, chronological splitting is essential because:
- It prevents data leakage from future to past
- It simulates real-world deployment conditions
- It tests the model's ability to forecast unseen future data

Random splits (A) would cause data leakage. Stratified sampling (B) doesn't preserve temporal order. K-fold CV (D) would also cause leakage in time series context.

</details>

---

## Key Takeaways

1. **Choose the right storage**: S3 for data lakes, Feature Store for ML features, FSx for Lustre for high-throughput training
2. **Prefer Parquet format**: Columnar storage with compression for large ML datasets
3. **Use SageMaker Feature Store**: Unified online and offline feature serving with point-in-time queries
4. **Implement data quality checks**: AWS Glue Data Quality with DQDL for validation rules
5. **Handle missing values appropriately**: Choose imputation strategy based on data characteristics
6. **Prevent data leakage**: Use time-based splits for temporal data, be careful with feature engineering
7. **Manage data lifecycle**: S3 lifecycle policies for cost optimization

---

## Navigation

| Previous | Next |
|----------|------|
| [Series Overview](/posts/aws-mla-c01-series-overview/) | [Domain 2: ML Model Development](/posts/aws-mla-c01-domain-2-model-development/) |
