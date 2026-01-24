---
title: "AWS MLA-C01 - Hands-On Labs: Practice with SageMaker and MLOps"
author: thanhnv1808
date: 2026-01-24 10:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, sagemaker, hands-on, labs]
description: Hands-on labs to practice AWS ML services for the MLA-C01 exam. Learn by doing with SageMaker Studio, training jobs, pipelines, and deployment.
pin: false
comments: true
---

## Introduction

This post provides comprehensive hands-on labs to practice with AWS ML services covered in the MLA-C01 exam. These labs focus on practical implementation of machine learning engineering workflows.

> **Cost Warning**: These labs will incur AWS charges. Use AWS Free Tier when available (250 hours t3.medium for 2 months) and clean up resources immediately after labs.
{: .prompt-warning }

### Prerequisites

- AWS Account with appropriate permissions
- Basic Python and ML knowledge
- Familiarity with Jupyter notebooks
- Completed the MLA-C01 theory lessons

### AWS Free Tier for ML Services

| Service | Free Tier |
|---------|-----------|
| Amazon SageMaker | 250 hours t3.medium notebook (2 months) |
| Amazon SageMaker Studio | 25 hours ml.t3.medium (2 months) |
| Amazon S3 | 5 GB storage, 20,000 GET requests |
| AWS Lambda | 1M free requests/month |
| Amazon ECR | 500 MB storage for 12 months |

---

## Lab 1: SageMaker Studio Setup and Configuration

### Objective
Set up SageMaker Studio and understand the development environment.

### Duration
30 minutes

### Steps

#### Step 1: Create a SageMaker Domain

1. Open **AWS Console** → Search for **SageMaker**
2. Click **Amazon SageMaker Studio**
3. Click **Get started** or **Create domain**

**Choose Setup Method:**
- **Quick setup**: Simplest option (recommended for learning)
- **Standard setup**: More control over configuration

For this lab, use **Quick setup**:

1. **Domain name**: `mla-c01-studio-domain`
2. **Execution role**: Create new role
3. In role creation:
   - S3 buckets: **Any S3 bucket**
   - Click **Create role**
4. Click **Submit**

Wait 5-10 minutes for domain creation.

> **Note**: Domain creation is a one-time setup. You can create multiple user profiles within a domain.
{: .prompt-info }

#### Step 2: Create a User Profile

1. Once domain is ready, click **Add user**
2. **User profile name**: `ml-engineer`
3. **Execution role**: Use existing role from domain
4. Click **Submit**

#### Step 3: Launch Studio

1. Select your user profile
2. Click **Open Studio**
3. Wait for Studio to load (first launch takes 2-3 minutes)

#### Step 4: Explore Studio Interface

**Key Components:**

| Component | Purpose |
|-----------|---------|
| **File browser** | Navigate files and folders |
| **Launcher** | Create notebooks, terminals, files |
| **Running terminals** | Active kernels and terminals |
| **Git** | Clone repositories |
| **SageMaker resources** | View training jobs, endpoints, etc. |

#### Step 5: Create Your First Notebook

1. Click **File** → **New** → **Notebook**
2. Select kernel: **Python 3 (Data Science)**
3. Wait for kernel to start
4. Rename notebook: Right-click → Rename → `lab1-introduction.ipynb`

#### Step 6: Test the Environment

In your notebook, run:

```python
# Cell 1: Import libraries
import sagemaker
import boto3
import pandas as pd
import numpy as np
from sagemaker import get_execution_role

print(f"SageMaker version: {sagemaker.__version__}")

# Cell 2: Get session and role
session = sagemaker.Session()
role = get_execution_role()
region = session.boto_region_name
bucket = session.default_bucket()

print(f"Region: {region}")
print(f"Role ARN: {role}")
print(f"Default bucket: {bucket}")

# Cell 3: Test S3 access
s3 = boto3.client('s3')
response = s3.list_buckets()
print(f"\nYou have access to {len(response['Buckets'])} S3 buckets")

# Cell 4: Create a test file in S3
test_data = pd.DataFrame({
    'feature1': np.random.rand(10),
    'feature2': np.random.rand(10),
    'label': np.random.randint(0, 2, 10)
})

test_file = 'test-data/sample.csv'
test_data.to_csv('sample.csv', index=False)

s3_path = f's3://{bucket}/mla-c01/{test_file}'
session.upload_data('sample.csv', bucket=bucket, key_prefix='mla-c01/test-data')

print(f"\nUploaded test data to: {s3_path}")
```

### Expected Output

```
SageMaker version: 2.x.x
Region: us-east-1
Role ARN: arn:aws:iam::123456789012:role/service-role/...
Default bucket: sagemaker-us-east-1-123456789012

You have access to X S3 buckets

Uploaded test data to: s3://sagemaker-us-east-1-123456789012/mla-c01/test-data/sample.csv
```

### Lab Questions

1. What is the difference between SageMaker Domain and User Profile?
2. What is the default S3 bucket naming convention?
3. What permissions does the SageMaker execution role have?

---

## Lab 2: Training Your First Model with SageMaker

### Objective
Train a machine learning model using SageMaker's built-in algorithms.

### Duration
45 minutes

### Use Case
Build a binary classifier to predict customer churn using XGBoost.

### Steps

#### Step 1: Prepare Training Data

Create a new notebook: `lab2-training.ipynb`

```python
# Import libraries
import sagemaker
from sagemaker import get_execution_role
from sagemaker.inputs import TrainingInput
import boto3
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split

# Setup
session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
prefix = 'mla-c01/churn-prediction'

# Create synthetic customer churn dataset
np.random.seed(42)
n_samples = 1000

data = pd.DataFrame({
    'tenure': np.random.randint(1, 72, n_samples),
    'monthly_charges': np.random.uniform(20, 120, n_samples),
    'total_charges': np.random.uniform(100, 8000, n_samples),
    'contract_type': np.random.choice([0, 1, 2], n_samples),  # 0=month-to-month, 1=1year, 2=2year
    'payment_method': np.random.choice([0, 1, 2, 3], n_samples),
    'internet_service': np.random.choice([0, 1, 2], n_samples),
    'tech_support': np.random.choice([0, 1], n_samples),
})

# Create target (churn is more likely with month-to-month contracts and high charges)
churn_probability = 0.3 + (data['contract_type'] == 0) * 0.3 - (data['tenure'] / 72) * 0.2
data['churn'] = (np.random.random(n_samples) < churn_probability).astype(int)

print(f"Dataset shape: {data.shape}")
print(f"Churn rate: {data['churn'].mean():.2%}")
print("\nFirst few rows:")
print(data.head())
```

#### Step 2: Split and Format Data for XGBoost

```python
# XGBoost expects label in the first column
train_data, test_data = train_test_split(data, test_size=0.2, random_state=42)

# Reorder columns: label first, then features
train_data = train_data[['churn', 'tenure', 'monthly_charges', 'total_charges',
                          'contract_type', 'payment_method', 'internet_service', 'tech_support']]
test_data = test_data[['churn', 'tenure', 'monthly_charges', 'total_charges',
                        'contract_type', 'payment_method', 'internet_service', 'tech_support']]

# Save to CSV (no headers, no index)
train_data.to_csv('train.csv', index=False, header=False)
test_data.to_csv('test.csv', index=False, header=False)

print(f"Training set: {train_data.shape}")
print(f"Test set: {test_data.shape}")
```

#### Step 3: Upload Data to S3

```python
# Upload training and test data
train_s3_path = session.upload_data('train.csv', bucket=bucket, key_prefix=f'{prefix}/train')
test_s3_path = session.upload_data('test.csv', bucket=bucket, key_prefix=f'{prefix}/test')

print(f"Training data uploaded to: {train_s3_path}")
print(f"Test data uploaded to: {test_s3_path}")

# Create TrainingInput objects
s3_input_train = TrainingInput(s3_data=train_s3_path, content_type='csv')
s3_input_test = TrainingInput(s3_data=test_s3_path, content_type='csv')
```

#### Step 4: Configure XGBoost Estimator

```python
from sagemaker.estimator import Estimator

# Get XGBoost container image
container = sagemaker.image_uris.retrieve('xgboost', session.boto_region_name, version='1.7-1')

# Create estimator
xgb_estimator = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/{prefix}/output',
    sagemaker_session=session,
    base_job_name='mla-c01-xgb-churn'
)

# Set hyperparameters
xgb_estimator.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    max_depth=5,
    eta=0.2,
    gamma=4,
    min_child_weight=6,
    subsample=0.8,
    verbosity=0
)

print("XGBoost estimator configured")
print(f"Training instance: {xgb_estimator.instance_type}")
print(f"Output path: {xgb_estimator.output_path}")
```

#### Step 5: Train the Model

```python
# Start training job
xgb_estimator.fit({
    'train': s3_input_train,
    'validation': s3_input_test
}, wait=True)

print("\nTraining completed!")
print(f"Model artifacts saved to: {xgb_estimator.model_data}")
```

#### Step 6: View Training Job in Console

1. Go to **SageMaker Console**
2. Click **Training** → **Training jobs**
3. Find your job (starts with `mla-c01-xgb-churn`)
4. Review:
   - Status
   - Training time
   - Instance type
   - Billable seconds
   - Metrics (validation:error)

### Expected Output

```
Training seconds: 120
Billable seconds: 120
Training job completed successfully

Model artifacts saved to: s3://bucket/prefix/output/model.tar.gz
```

### Lab Questions

1. Why does XGBoost expect the label in the first column?
2. What is the purpose of the validation dataset?
3. How would you optimize training costs?

---

## Lab 3: SageMaker Feature Store

### Objective
Create and use a Feature Store for managing ML features.

### Duration
45 minutes

### Steps

#### Step 1: Create Feature Definitions

Create notebook: `lab3-feature-store.ipynb`

```python
import sagemaker
from sagemaker.feature_store.feature_group import FeatureGroup
from sagemaker import get_execution_role
import pandas as pd
import time
from time import gmtime, strftime

session = sagemaker.Session()
role = get_execution_role()
region = session.boto_region_name
bucket = session.default_bucket()

# Create sample customer features
customer_data = pd.DataFrame({
    'customer_id': [f'C{i:05d}' for i in range(100)],
    'age': np.random.randint(18, 80, 100),
    'tenure_months': np.random.randint(1, 72, 100),
    'monthly_spend': np.random.uniform(20, 200, 100),
    'support_calls': np.random.randint(0, 10, 100),
    'contract_type': np.random.choice(['month', 'year', '2year'], 100),
    'event_time': pd.Timestamp.now().timestamp()
})

# Convert contract_type to numeric for Feature Store
customer_data['contract_type_code'] = customer_data['contract_type'].map({
    'month': 0, 'year': 1, '2year': 2
})

customer_data = customer_data.drop('contract_type', axis=1)

print(customer_data.head())
```

#### Step 2: Create Feature Group

```python
from sagemaker.feature_store.feature_definition import (
    FeatureDefinition,
    FeatureTypeEnum,
)

# Define feature group
feature_group_name = f'mla-c01-customer-features-{strftime("%Y-%m-%d-%H-%M-%S", gmtime())}'

feature_group = FeatureGroup(
    name=feature_group_name,
    sagemaker_session=session
)

# Load feature definitions from DataFrame
feature_group.load_feature_definitions(data_frame=customer_data)

print(f"\nFeature Group Name: {feature_group_name}")
print(f"Number of features: {len(customer_data.columns)}")
```

#### Step 3: Create Online and Offline Stores

```python
# Create feature group with online and offline stores
feature_group.create(
    s3_uri=f's3://{bucket}/mla-c01/feature-store',
    record_identifier_name='customer_id',
    event_time_feature_name='event_time',
    role_arn=role,
    enable_online_store=True
)

print("Feature Group created successfully!")
print("Waiting for Feature Group to become active...")

# Wait for feature group to be created
status = feature_group.describe().get('FeatureGroupStatus')
while status == 'Creating':
    print(f"Status: {status}")
    time.sleep(5)
    status = feature_group.describe().get('FeatureGroupStatus')

print(f"Final Status: {status}")
```

#### Step 4: Ingest Data into Feature Store

```python
# Ingest data
feature_group.ingest(
    data_frame=customer_data,
    max_workers=3,
    wait=True
)

print(f"Ingested {len(customer_data)} records into Feature Store")
```

#### Step 5: Retrieve Features from Online Store

```python
# Get a single record
customer_id_to_fetch = 'C00001'

record = session.boto_session.client('sagemaker-featurestore-runtime').get_record(
    FeatureGroupName=feature_group_name,
    RecordIdentifierValueAsString=customer_id_to_fetch
)

print(f"\nFeatures for {customer_id_to_fetch}:")
for feature in record['Record']:
    print(f"  {feature['FeatureName']}: {feature['ValueAsString']}")
```

#### Step 6: Query Offline Store with Athena

```python
# Build Athena query
query_string = f"""
SELECT customer_id, age, tenure_months, monthly_spend, support_calls
FROM "{feature_group.athena_query().table_name}"
WHERE support_calls > 5
LIMIT 10
"""

# Run query
output_location = f's3://{bucket}/mla-c01/athena-results/'

feature_group.athena_query().run(
    query_string=query_string,
    output_location=output_location
)

# Wait for query to complete
feature_group.athena_query().wait()

# Get results
results = feature_group.athena_query().as_dataframe()
print("\nCustomers with more than 5 support calls:")
print(results)
```

### Lab Questions

1. What is the difference between online and offline Feature Store?
2. When would you use each type of store?
3. How does Feature Store help with feature consistency?

---

## Lab 4: Hyperparameter Tuning

### Objective
Use SageMaker Automatic Model Tuning to find optimal hyperparameters.

### Duration
60 minutes

### Steps

#### Step 1: Setup and Data Preparation

Create notebook: `lab4-hyperparameter-tuning.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.tuner import (
    IntegerParameter,
    ContinuousParameter,
    CategoricalParameter,
    HyperparameterTuner
)

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()

# Use data from Lab 2
train_s3_path = f's3://{bucket}/mla-c01/churn-prediction/train/train.csv'
test_s3_path = f's3://{bucket}/mla-c01/churn-prediction/test/test.csv'
```

#### Step 2: Define Hyperparameter Ranges

```python
hyperparameter_ranges = {
    'max_depth': IntegerParameter(3, 10),
    'eta': ContinuousParameter(0.01, 0.5),
    'min_child_weight': IntegerParameter(1, 10),
    'subsample': ContinuousParameter(0.5, 1.0),
    'gamma': ContinuousParameter(0, 5),
    'alpha': ContinuousParameter(0, 2),
}

print("Hyperparameter search space:")
for param, range_obj in hyperparameter_ranges.items():
    print(f"  {param}: {range_obj}")
```

#### Step 3: Create Base Estimator

```python
from sagemaker.estimator import Estimator

container = sagemaker.image_uris.retrieve('xgboost', session.boto_region_name, version='1.7-1')

xgb_estimator = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/mla-c01/hpo-output',
    sagemaker_session=session
)

# Set static hyperparameters
xgb_estimator.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    eval_metric='error'
)
```

#### Step 4: Configure Hyperparameter Tuning Job

```python
tuner = HyperparameterTuner(
    estimator=xgb_estimator,
    objective_metric_name='validation:error',
    hyperparameter_ranges=hyperparameter_ranges,
    metric_definitions=[
        {'Name': 'validation:error', 'Regex': 'validation-error:([0-9\\.]+)'}
    ],
    max_jobs=20,  # Total number of training jobs
    max_parallel_jobs=3,  # Number of jobs to run in parallel
    objective_type='Minimize',
    strategy='Bayesian',  # Options: Bayesian, Random, Grid
    base_tuning_job_name='mla-c01-hpo'
)

print("Tuner configured:")
print(f"  Max jobs: {tuner.max_jobs}")
print(f"  Parallel jobs: {tuner.max_parallel_jobs}")
print(f"  Strategy: {tuner.strategy}")
```

#### Step 5: Start Tuning Job

```python
from sagemaker.inputs import TrainingInput

s3_input_train = TrainingInput(train_s3_path, content_type='csv')
s3_input_test = TrainingInput(test_s3_path, content_type='csv')

# Start tuning (set wait=False to avoid blocking)
tuner.fit(
    {
        'train': s3_input_train,
        'validation': s3_input_test
    },
    wait=False
)

print(f"Tuning job started: {tuner.latest_tuning_job.name}")
print("This will take approximately 60-90 minutes")
```

#### Step 6: Monitor Tuning Job

```python
# Check status
status = session.sagemaker_client.describe_hyper_parameter_tuning_job(
    HyperParameterTuningJobName=tuner.latest_tuning_job.name
)

print(f"\nTuning Job Status: {status['HyperParameterTuningJobStatus']}")
print(f"Training Jobs Completed: {status['TrainingJobStatusCounters']['Completed']}")
print(f"Training Jobs In Progress: {status['TrainingJobStatusCounters'].get('InProgress', 0)}")

# After tuning completes (wait or check later)
# Get best training job
# tuner.wait()  # Uncomment to wait for completion
```

#### Step 7: Analyze Results (After Completion)

```python
# This section runs after tuning completes

# Get tuning analytics
tuning_analytics = sagemaker.HyperparameterTuningJobAnalytics(
    tuner.latest_tuning_job.name
)

# Convert to DataFrame
tuning_df = tuning_analytics.dataframe()

# Display best runs
print("\nTop 5 Training Jobs:")
print(tuning_df.sort_values('FinalObjectiveValue').head())

# Get best hyperparameters
best_training_job = tuner.best_training_job()
print(f"\nBest Training Job: {best_training_job}")

# Visualize results
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 5))

# Plot 1: Objective value over time
plt.subplot(1, 2, 1)
plt.scatter(range(len(tuning_df)), tuning_df['FinalObjectiveValue'])
plt.xlabel('Training Job Number')
plt.ylabel('Validation Error')
plt.title('Tuning Progress')

# Plot 2: Max depth vs error
plt.subplot(1, 2, 2)
plt.scatter(tuning_df['max_depth'], tuning_df['FinalObjectiveValue'])
plt.xlabel('Max Depth')
plt.ylabel('Validation Error')
plt.title('Max Depth Impact')

plt.tight_layout()
plt.show()
```

### Lab Questions

1. What is the difference between Bayesian and Random search strategies?
2. How do max_jobs and max_parallel_jobs affect cost and time?
3. When would you use Grid search vs Bayesian optimization?

---

## Lab 5: Model Deployment and Endpoints

### Objective
Deploy a trained model to a SageMaker endpoint for real-time inference.

### Duration
30 minutes

### Steps

#### Step 1: Deploy Model from Training Job

Create notebook: `lab5-deployment.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
import boto3
import json

session = sagemaker.Session()
role = get_execution_role()

# Use the trained model from Lab 2
# Replace with your actual training job name
training_job_name = 'mla-c01-xgb-churn-2026-01-24-10-30-00-000'

# Attach to the training job
xgb_attached = sagemaker.estimator.Estimator.attach(training_job_name)

print(f"Model artifacts: {xgb_attached.model_data}")
```

#### Step 2: Deploy to Real-time Endpoint

```python
# Deploy model
predictor = xgb_attached.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.large',
    endpoint_name='mla-c01-churn-predictor'
)

print(f"Endpoint created: {predictor.endpoint_name}")
print("Endpoint is now live and ready for predictions!")
```

#### Step 3: Make Predictions

```python
from sagemaker.serializers import CSVSerializer
from sagemaker.deserializers import JSONDeserializer

# Configure serializers
predictor.serializer = CSVSerializer()
predictor.deserializer = JSONDeserializer()

# Create test samples (without label column)
test_samples = [
    [60, 85.5, 5000, 2, 1, 1, 1],  # Long tenure, 2-year contract - likely not churning
    [3, 115.0, 350, 0, 2, 0, 0],   # Short tenure, month-to-month - likely churning
    [24, 75.0, 1800, 1, 1, 1, 1],  # Medium tenure, 1-year contract
]

# Make predictions
for i, sample in enumerate(test_samples, 1):
    prediction = predictor.predict(sample)
    print(f"\nSample {i}:")
    print(f"  Features: {sample}")
    print(f"  Churn Probability: {prediction:.4f}")
    print(f"  Prediction: {'CHURN' if prediction > 0.5 else 'NO CHURN'}")
```

#### Step 4: Batch Predictions

```python
# Prepare multiple predictions
import pandas as pd
import numpy as np

# Generate batch of test data
batch_data = pd.DataFrame({
    'tenure': np.random.randint(1, 72, 20),
    'monthly_charges': np.random.uniform(20, 120, 20),
    'total_charges': np.random.uniform(100, 8000, 20),
    'contract_type': np.random.choice([0, 1, 2], 20),
    'payment_method': np.random.choice([0, 1, 2, 3], 20),
    'internet_service': np.random.choice([0, 1, 2], 20),
    'tech_support': np.random.choice([0, 1], 20),
})

# Convert to list of lists
batch_samples = batch_data.values.tolist()

# Predict
predictions = []
for sample in batch_samples:
    pred = predictor.predict(sample)
    predictions.append(pred)

# Add predictions to DataFrame
batch_data['churn_probability'] = predictions
batch_data['predicted_churn'] = [1 if p > 0.5 else 0 for p in predictions]

print("\nBatch Predictions:")
print(batch_data.head(10))
print(f"\nPredicted churn rate: {batch_data['predicted_churn'].mean():.2%}")
```

#### Step 5: Monitor Endpoint

```python
# Get endpoint metrics using CloudWatch
import datetime
from datetime import timedelta

cloudwatch = boto3.client('cloudwatch')

# Get invocation metrics
metrics = cloudwatch.get_metric_statistics(
    Namespace='AWS/SageMaker',
    MetricName='Invocations',
    Dimensions=[
        {'Name': 'EndpointName', 'Value': predictor.endpoint_name},
        {'Name': 'VariantName', 'Value': 'AllTraffic'}
    ],
    StartTime=datetime.datetime.now() - timedelta(hours=1),
    EndTime=datetime.datetime.now(),
    Period=300,
    Statistics=['Sum']
)

print("\nEndpoint Invocation Metrics:")
for datapoint in metrics['Datapoints']:
    print(f"  {datapoint['Timestamp']}: {datapoint['Sum']} invocations")
```

### Lab Questions

1. What is the difference between real-time and batch endpoints?
2. How would you implement A/B testing with endpoints?
3. What instance type considerations matter for deployment?

---

## Lab 6: SageMaker Pipelines for MLOps

### Objective
Build an end-to-end ML pipeline with SageMaker Pipelines.

### Duration
60 minutes

### Steps

#### Step 1: Define Pipeline Parameters

Create notebook: `lab6-pipelines.ipynb`

```python
import sagemaker
from sagemaker.workflow.parameters import (
    ParameterInteger,
    ParameterString,
    ParameterFloat
)
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import (
    ProcessingStep,
    TrainingStep,
    CreateModelStep
)
from sagemaker.workflow.conditions import ConditionGreaterThanOrEqualTo
from sagemaker.workflow.condition_step import ConditionStep
from sagemaker.workflow.functions import JsonGet
from sagemaker.workflow.properties import PropertyFile

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
region = session.boto_region_name

# Define pipeline parameters
processing_instance_count = ParameterInteger(
    name="ProcessingInstanceCount",
    default_value=1
)

processing_instance_type = ParameterString(
    name="ProcessingInstanceType",
    default_value="ml.m5.xlarge"
)

training_instance_type = ParameterString(
    name="TrainingInstanceType",
    default_value="ml.m5.xlarge"
)

model_approval_threshold = ParameterFloat(
    name="ModelApprovalThreshold",
    default_value=0.8
)

input_data = ParameterString(
    name="InputData",
    default_value=f"s3://{bucket}/mla-c01/pipeline-input/"
)

print("Pipeline parameters defined")
```

#### Step 2: Create Processing Step

```python
from sagemaker.processing import ProcessingInput, ProcessingOutput
from sagemaker.sklearn.processing import SKLearnProcessor

# Create preprocessing script
preprocessing_script = """
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
import argparse
import os

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-test-split-ratio', type=float, default=0.2)
    args, _ = parser.parse_known_args()

    # Read input data
    input_data_path = '/opt/ml/processing/input/data.csv'
    df = pd.read_csv(input_data_path)

    # Simple preprocessing
    # Remove duplicates
    df = df.drop_duplicates()

    # Fill missing values
    df = df.fillna(df.median())

    # Split data
    train, test = train_test_split(df, test_size=args.train_test_split_ratio, random_state=42)

    # Save to output
    train.to_csv('/opt/ml/processing/train/train.csv', index=False, header=False)
    test.to_csv('/opt/ml/processing/test/test.csv', index=False, header=False)

    print(f"Training samples: {len(train)}")
    print(f"Test samples: {len(test)}")
"""

# Save script locally
with open('preprocessing.py', 'w') as f:
    f.write(preprocessing_script)

# Upload to S3
preprocessing_code_uri = session.upload_data(
    'preprocessing.py',
    bucket=bucket,
    key_prefix='mla-c01/pipeline-code'
)

# Create processor
sklearn_processor = SKLearnProcessor(
    framework_version='1.0-1',
    role=role,
    instance_type=processing_instance_type,
    instance_count=processing_instance_count,
    base_job_name='mla-pipeline-preprocessing'
)

# Define processing step
step_process = ProcessingStep(
    name="PreprocessData",
    processor=sklearn_processor,
    inputs=[
        ProcessingInput(
            source=input_data,
            destination="/opt/ml/processing/input"
        )
    ],
    outputs=[
        ProcessingOutput(
            output_name="train",
            source="/opt/ml/processing/train"
        ),
        ProcessingOutput(
            output_name="test",
            source="/opt/ml/processing/test"
        )
    ],
    code=preprocessing_code_uri
)

print("Processing step created")
```

#### Step 3: Create Training Step

```python
from sagemaker.estimator import Estimator
from sagemaker.inputs import TrainingInput

# Get XGBoost container
container = sagemaker.image_uris.retrieve('xgboost', region, version='1.7-1')

# Create estimator
xgb_estimator = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type=training_instance_type,
    output_path=f's3://{bucket}/mla-c01/pipeline-output',
    base_job_name='mla-pipeline-training'
)

xgb_estimator.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    max_depth=5,
    eta=0.2,
    gamma=4,
    min_child_weight=6,
    subsample=0.8
)

# Define training step
step_train = TrainingStep(
    name="TrainModel",
    estimator=xgb_estimator,
    inputs={
        "train": TrainingInput(
            s3_data=step_process.properties.ProcessingOutputConfig.Outputs["train"].S3Output.S3Uri,
            content_type="text/csv"
        ),
        "validation": TrainingInput(
            s3_data=step_process.properties.ProcessingOutputConfig.Outputs["test"].S3Output.S3Uri,
            content_type="text/csv"
        )
    }
)

print("Training step created")
```

#### Step 4: Create Model Evaluation Step

```python
# Create evaluation script
evaluation_script = """
import json
import pathlib
import pickle
import tarfile
import numpy as np
import pandas as pd
import xgboost
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

if __name__ == '__main__':
    # Load model
    model_path = '/opt/ml/processing/model/model.tar.gz'
    with tarfile.open(model_path) as tar:
        tar.extractall(path='.')

    model = pickle.load(open('xgboost-model', 'rb'))

    # Load test data
    test_path = '/opt/ml/processing/test/test.csv'
    test_data = pd.read_csv(test_path, header=None)

    X_test = test_data.iloc[:, 1:].values
    y_test = test_data.iloc[:, 0].values

    # Make predictions
    predictions = model.predict(xgboost.DMatrix(X_test))
    predictions_binary = (predictions > 0.5).astype(int)

    # Calculate metrics
    accuracy = accuracy_score(y_test, predictions_binary)
    precision = precision_score(y_test, predictions_binary)
    recall = recall_score(y_test, predictions_binary)
    f1 = f1_score(y_test, predictions_binary)

    # Save evaluation report
    report_dict = {
        'metrics': {
            'accuracy': {'value': float(accuracy)},
            'precision': {'value': float(precision)},
            'recall': {'value': float(recall)},
            'f1': {'value': float(f1)}
        }
    }

    output_dir = '/opt/ml/processing/evaluation'
    pathlib.Path(output_dir).mkdir(parents=True, exist_ok=True)

    with open(f'{output_dir}/evaluation.json', 'w') as f:
        json.dump(report_dict, f)

    print(f"Accuracy: {accuracy:.4f}")
    print(f"Precision: {precision:.4f}")
    print(f"Recall: {recall:.4f}")
    print(f"F1 Score: {f1:.4f}")
"""

# Save and upload script
with open('evaluation.py', 'w') as f:
    f.write(evaluation_script)

evaluation_code_uri = session.upload_data(
    'evaluation.py',
    bucket=bucket,
    key_prefix='mla-c01/pipeline-code'
)

# Create evaluation processor
eval_processor = SKLearnProcessor(
    framework_version='1.0-1',
    role=role,
    instance_type='ml.m5.xlarge',
    instance_count=1,
    base_job_name='mla-pipeline-eval'
)

# Create evaluation report property
evaluation_report = PropertyFile(
    name="EvaluationReport",
    output_name="evaluation",
    path="evaluation.json"
)

# Define evaluation step
step_eval = ProcessingStep(
    name="EvaluateModel",
    processor=eval_processor,
    inputs=[
        ProcessingInput(
            source=step_train.properties.ModelArtifacts.S3ModelArtifacts,
            destination="/opt/ml/processing/model"
        ),
        ProcessingInput(
            source=step_process.properties.ProcessingOutputConfig.Outputs["test"].S3Output.S3Uri,
            destination="/opt/ml/processing/test"
        )
    ],
    outputs=[
        ProcessingOutput(
            output_name="evaluation",
            source="/opt/ml/processing/evaluation"
        )
    ],
    code=evaluation_code_uri,
    property_files=[evaluation_report]
)

print("Evaluation step created")
```

#### Step 5: Create Conditional Model Registration

```python
from sagemaker.model import Model
from sagemaker.workflow.model_step import ModelStep
from sagemaker.model_metrics import MetricsSource, ModelMetrics

# Create model
model = Model(
    image_uri=container,
    model_data=step_train.properties.ModelArtifacts.S3ModelArtifacts,
    sagemaker_session=session,
    role=role
)

# Define model registration step
step_register = ModelStep(
    name="RegisterModel",
    step_args=model.register(
        content_types=["text/csv"],
        response_types=["application/json"],
        inference_instances=["ml.m5.large"],
        transform_instances=["ml.m5.xlarge"],
        model_package_group_name="mla-c01-churn-models",
        approval_status="PendingManualApproval",
        model_metrics=ModelMetrics(
            model_statistics=MetricsSource(
                s3_uri=step_eval.properties.ProcessingOutputConfig.Outputs["evaluation"].S3Output.S3Uri,
                content_type="application/json"
            )
        )
    )
)

# Create condition: Only register if accuracy > threshold
cond_gte = ConditionGreaterThanOrEqualTo(
    left=JsonGet(
        step_name=step_eval.name,
        property_file=evaluation_report,
        json_path="metrics.accuracy.value"
    ),
    right=model_approval_threshold
)

step_cond = ConditionStep(
    name="CheckAccuracyCondition",
    conditions=[cond_gte],
    if_steps=[step_register],
    else_steps=[]
)

print("Conditional registration step created")
```

#### Step 6: Create and Execute Pipeline

```python
# Create pipeline
pipeline = Pipeline(
    name="mla-c01-churn-pipeline",
    parameters=[
        processing_instance_count,
        processing_instance_type,
        training_instance_type,
        model_approval_threshold,
        input_data
    ],
    steps=[step_process, step_train, step_eval, step_cond],
    sagemaker_session=session
)

# Create/update pipeline
pipeline.upsert(role_arn=role)

print(f"Pipeline created: {pipeline.name}")

# Start pipeline execution
execution = pipeline.start()
print(f"Pipeline execution started: {execution.arn}")

# Wait for completion (optional)
# execution.wait()
```

#### Step 7: Monitor Pipeline Execution

```python
# Get execution status
execution.describe()

# List all pipeline executions
executions = pipeline.list_executions()
print("\nRecent Pipeline Executions:")
for exec_summary in executions['PipelineExecutionSummaries'][:5]:
    print(f"  {exec_summary['PipelineExecutionArn']}")
    print(f"    Status: {exec_summary['PipelineExecutionStatus']}")
    print(f"    Start: {exec_summary['StartTime']}")
```

### Lab Questions

1. What are the benefits of using SageMaker Pipelines?
2. How does conditional execution improve efficiency?
3. When would you use pipeline parameters?

---

## Lab 7: Model Monitoring with SageMaker Model Monitor

### Objective
Set up continuous monitoring for deployed models.

### Duration
45 minutes

### Steps

#### Step 1: Create Baseline from Training Data

Create notebook: `lab7-model-monitor.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.model_monitor import DataCaptureConfig
from sagemaker.model_monitor import DefaultModelMonitor
from sagemaker.model_monitor.dataset_format import DatasetFormat

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()

# Path to training data (from Lab 2)
baseline_data_uri = f's3://{bucket}/mla-c01/churn-prediction/train/train.csv'
baseline_results_uri = f's3://{bucket}/mla-c01/model-monitor/baseline'

print(f"Baseline data: {baseline_data_uri}")
print(f"Results will be saved to: {baseline_results_uri}")
```

#### Step 2: Suggest Baseline Statistics

```python
from sagemaker.model_monitor import DefaultModelMonitor

my_monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    volume_size_in_gb=20,
    max_runtime_in_seconds=3600,
)

# Generate baseline
my_monitor.suggest_baseline(
    baseline_dataset=baseline_data_uri,
    dataset_format=DatasetFormat.csv(header=False),
    output_s3_uri=baseline_results_uri,
    wait=True
)

print("Baseline statistics generated successfully!")
```

#### Step 3: View Baseline Statistics

```python
import pandas as pd
from sagemaker.s3 import S3Downloader

# Download baseline statistics
baseline_job = my_monitor.latest_baselining_job
schema_df = pd.json_normalize(
    baseline_job.baseline_statistics().body_dict["features"]
)

print("\nBaseline Statistics:")
print(schema_df.head(10))

# Download constraints
constraints_df = pd.json_normalize(
    baseline_job.suggested_constraints().body_dict["features"]
)

print("\nSuggested Constraints:")
print(constraints_df.head(10))
```

#### Step 4: Enable Data Capture on Endpoint

```python
from sagemaker.model_monitor import DataCaptureConfig

# Configure data capture
data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,  # Capture 100% of requests (use lower % in production)
    destination_s3_uri=f's3://{bucket}/mla-c01/model-monitor/data-capture'
)

# Update existing endpoint or deploy new one with data capture
endpoint_name = 'mla-c01-churn-predictor'

# Note: You need to update the endpoint configuration
# This requires redeploying the model

predictor = xgb_attached.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.large',
    endpoint_name=endpoint_name,
    data_capture_config=data_capture_config,
    update_endpoint=True  # Update existing endpoint
)

print(f"Data capture enabled on endpoint: {endpoint_name}")
```

#### Step 5: Generate Traffic for Monitoring

```python
import time
import numpy as np

# Generate predictions to create captured data
print("Generating traffic...")

for i in range(50):
    # Create random sample
    sample = [
        np.random.randint(1, 72),      # tenure
        np.random.uniform(20, 120),     # monthly_charges
        np.random.uniform(100, 8000),   # total_charges
        np.random.choice([0, 1, 2]),    # contract_type
        np.random.choice([0, 1, 2, 3]), # payment_method
        np.random.choice([0, 1, 2]),    # internet_service
        np.random.choice([0, 1])        # tech_support
    ]

    prediction = predictor.predict(sample)

    if (i + 1) % 10 == 0:
        print(f"  Generated {i + 1} predictions")

    time.sleep(1)  # Small delay to simulate real traffic

print("Traffic generation complete!")
```

#### Step 6: Create Monitoring Schedule

```python
from sagemaker.model_monitor import CronExpressionGenerator

# Create monitoring schedule
mon_schedule_name = 'mla-c01-churn-monitor-schedule'

my_monitor.create_monitoring_schedule(
    monitor_schedule_name=mon_schedule_name,
    endpoint_input=endpoint_name,
    output_s3_uri=f's3://{bucket}/mla-c01/model-monitor/reports',
    statistics=my_monitor.baseline_statistics(),
    constraints=my_monitor.suggested_constraints(),
    schedule_cron_expression=CronExpressionGenerator.hourly(),  # Run every hour
    enable_cloudwatch_metrics=True
)

print(f"Monitoring schedule created: {mon_schedule_name}")
print("Schedule will run hourly to check for data drift")
```

#### Step 7: View Monitoring Results

```python
# List executions
executions = my_monitor.list_executions()

print("\nMonitoring Executions:")
for execution in executions[:5]:
    print(f"  Execution: {execution.scheduled_time}")
    print(f"    Status: {execution.describe()['ProcessingJobStatus']}")

    # If completed, show violations
    if execution.describe()['ProcessingJobStatus'] == 'Completed':
        violations = execution.constraint_violations()
        if violations:
            print(f"    Violations detected: {len(violations.body_dict['violations'])}")
        else:
            print("    No violations detected")
```

### Lab Questions

1. What types of drift can Model Monitor detect?
2. How would you handle detected violations?
3. What is an appropriate data capture percentage for production?

---

## Cleanup Instructions

After completing labs, clean up resources to avoid unnecessary charges:

### Delete Endpoints

```python
# Delete prediction endpoint
predictor.delete_endpoint()
print("Endpoint deleted")

# Delete monitoring schedule
my_monitor.delete_monitoring_schedule()
print("Monitoring schedule deleted")
```

### Delete S3 Data

```bash
# Use AWS CLI
aws s3 rm s3://your-bucket/mla-c01/ --recursive
```

### Delete SageMaker Resources

1. **SageMaker Studio**:
   - Stop all running apps
   - Delete user profiles (optional)
   - Delete domain (optional - only if not needed)

2. **Feature Store**:
   - Delete feature groups via console

3. **Model Registry**:
   - Delete model packages if not needed

### Check for Running Resources

```python
# List all endpoints
sm_client = boto3.client('sagemaker')
endpoints = sm_client.list_endpoints()['Endpoints']

print("Active endpoints:")
for ep in endpoints:
    print(f"  {ep['EndpointName']} - {ep['EndpointStatus']}")

# List training jobs (check for any still running)
training_jobs = sm_client.list_training_jobs(
    StatusEquals='InProgress'
)['TrainingJobSummaries']

print(f"\nRunning training jobs: {len(training_jobs)}")
```

---

## Additional Resources

### AWS Workshops
- [SageMaker Immersion Day](https://catalog.us-east-1.prod.workshops.aws/workshops/63069e26-921c-4ce1-9cc7-dd882ff62575)
- [MLOps Workshop](https://catalog.workshops.aws/mlops-workshop/en-US)
- [SageMaker Pipelines Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/1c03e2a4-9167-4a8c-a965-4f3b6f2c49cb)

### Documentation
- [SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/)
- [SageMaker Python SDK](https://sagemaker.readthedocs.io/)
- [SageMaker Examples GitHub](https://github.com/aws/amazon-sagemaker-examples)

### Practice Datasets
- [UCI ML Repository](https://archive.ics.uci.edu/ml/index.php)
- [Kaggle Datasets](https://www.kaggle.com/datasets)
- [AWS Open Data Registry](https://registry.opendata.aws/)

---

**Back to Series**: [AWS Machine Learning Engineer Associate (MLA-C01) - Complete Study Guide](/posts/aws-mla-c01-series/)

---

*Have questions about the labs? Leave a comment below!*
