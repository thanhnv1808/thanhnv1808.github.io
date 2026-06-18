---
title: "AWS MLA-C01 - Advanced Labs: Production MLOps and Distributed Training"
author: thanhnv1808
date: 2026-01-24 14:00:00 +0700
categories: [AWS, ML Engineer Associate]
tags: [aws, machine-learning, mla-c01, sagemaker, advanced, mlops]
description: Advanced hands-on labs for AWS ML Engineer Associate. Deep dive into distributed training, multi-model endpoints, A/B testing, and production MLOps patterns.
pin: false
comments: true
---

## Introduction

These advanced labs build on the foundational labs and provide production-ready ML engineering patterns and techniques.

> **Prerequisites**: Complete the basic hands-on labs first. These labs involve significant AWS charges - monitor costs carefully.
{: .prompt-warning }

---

## Lab 8: Distributed Training with Data Parallelism

### Objective
Implement distributed training across multiple GPUs and instances to accelerate model training.

### Duration
90 minutes

### Use Case
Train a deep learning model using PyTorch with SageMaker's distributed training capabilities.

### Steps

#### Step 1: Prepare Training Script with Distributed Support

Create notebook: `lab8-distributed-training.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.pytorch import PyTorch

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
```

Create training script: `train_distributed.py`

```python
# train_distributed.py
import argparse
import os
import json
import torch
import torch.nn as nn
import torch.optim as optim
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data import Dataset, DataLoader
from torch.utils.data.distributed import DistributedSampler
import numpy as np

class SimpleModel(nn.Module):
    """Simple neural network for demonstration"""
    def __init__(self, input_dim, hidden_dim, output_dim):
        super(SimpleModel, self).__init__()
        self.network = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(hidden_dim, output_dim),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.network(x)

class TabularDataset(Dataset):
    """Custom dataset for tabular data"""
    def __init__(self, data_dir, train=True):
        filename = 'train.csv' if train else 'test.csv'
        data = np.loadtxt(os.path.join(data_dir, filename), delimiter=',')

        self.X = torch.FloatTensor(data[:, 1:])  # Features (skip label)
        self.y = torch.FloatTensor(data[:, 0]).unsqueeze(1)  # Label

    def __len__(self):
        return len(self.X)

    def __getitem__(self, idx):
        return self.X[idx], self.y[idx]

def train(args):
    # Initialize distributed training
    is_distributed = len(args.hosts) > 1 and args.dist_backend is not None

    if is_distributed:
        # Initialize the distributed environment
        world_size = len(args.hosts)
        os.environ['WORLD_SIZE'] = str(world_size)
        host_rank = args.hosts.index(args.current_host)
        os.environ['RANK'] = str(host_rank)
        dist.init_process_group(backend=args.dist_backend, rank=host_rank, world_size=world_size)
        print(f'Initialized the distributed environment: \'{args.dist_backend}\' backend on '
              f'{world_size} nodes. Current host rank is {host_rank}.')

    # Set device
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    print(f"Using device: {device}")

    # Load data
    train_dataset = TabularDataset(args.data_dir, train=True)

    # Use DistributedSampler for distributed training
    if is_distributed:
        train_sampler = DistributedSampler(
            train_dataset,
            num_replicas=world_size,
            rank=host_rank
        )
    else:
        train_sampler = None

    train_loader = DataLoader(
        train_dataset,
        batch_size=args.batch_size,
        shuffle=(train_sampler is None),
        sampler=train_sampler,
        num_workers=args.num_workers,
        pin_memory=True
    )

    # Create model
    input_dim = train_dataset.X.shape[1]
    model = SimpleModel(input_dim, args.hidden_dim, 1).to(device)

    # Wrap model with DDP for distributed training
    if is_distributed:
        model = DDP(model)

    # Loss and optimizer
    criterion = nn.BCELoss()
    optimizer = optim.Adam(model.parameters(), lr=args.lr)

    # Training loop
    model.train()
    for epoch in range(args.epochs):
        if is_distributed:
            train_sampler.set_epoch(epoch)

        epoch_loss = 0.0
        correct = 0
        total = 0

        for batch_idx, (data, target) in enumerate(train_loader):
            data, target = data.to(device), target.to(device)

            optimizer.zero_grad()
            output = model(data)
            loss = criterion(output, target)
            loss.backward()
            optimizer.step()

            epoch_loss += loss.item()
            predictions = (output > 0.5).float()
            correct += (predictions == target).sum().item()
            total += target.size(0)

            if batch_idx % 10 == 0:
                print(f'Epoch {epoch+1}/{args.epochs}, Batch {batch_idx}, Loss: {loss.item():.4f}')

        accuracy = 100. * correct / total
        avg_loss = epoch_loss / len(train_loader)

        print(f'Epoch {epoch+1}/{args.epochs} completed:')
        print(f'  Average Loss: {avg_loss:.4f}')
        print(f'  Accuracy: {accuracy:.2f}%')

    # Save model (only on rank 0 in distributed setting)
    if not is_distributed or host_rank == 0:
        model_to_save = model.module if is_distributed else model
        torch.save(model_to_save.state_dict(), os.path.join(args.model_dir, 'model.pth'))
        print(f'Model saved to {args.model_dir}')

if __name__ == '__main__':
    parser = argparse.ArgumentParser()

    # Hyperparameters
    parser.add_argument('--epochs', type=int, default=10)
    parser.add_argument('--batch-size', type=int, default=64)
    parser.add_argument('--lr', type=float, default=0.001)
    parser.add_argument('--hidden-dim', type=int, default=128)
    parser.add_argument('--num-workers', type=int, default=4)

    # SageMaker parameters
    parser.add_argument('--model-dir', type=str, default=os.environ.get('SM_MODEL_DIR'))
    parser.add_argument('--data-dir', type=str, default=os.environ.get('SM_CHANNEL_TRAINING'))
    parser.add_argument('--num-gpus', type=int, default=os.environ.get('SM_NUM_GPUS', 0))

    # Distributed training parameters
    parser.add_argument('--hosts', type=list, default=json.loads(os.environ.get('SM_HOSTS', '[]')))
    parser.add_argument('--current-host', type=str, default=os.environ.get('SM_CURRENT_HOST'))
    parser.add_argument('--dist-backend', type=str, default='nccl')

    args = parser.parse_args()
    train(args)
```

#### Step 2: Prepare and Upload Training Data

```python
# Generate larger dataset for distributed training
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split

np.random.seed(42)
n_samples = 100000  # Larger dataset

data = pd.DataFrame({
    'feature_1': np.random.randn(n_samples),
    'feature_2': np.random.randn(n_samples),
    'feature_3': np.random.randn(n_samples),
    'feature_4': np.random.randn(n_samples),
    'feature_5': np.random.randn(n_samples),
})

# Create non-linear target
data['target'] = (
    (data['feature_1'] * data['feature_2'] > 0) |
    (data['feature_3'] + data['feature_4'] > 1)
).astype(int)

# Split data
train_data, test_data = train_test_split(data, test_size=0.2, random_state=42)

# Reorder: label first
train_data = train_data[['target'] + [col for col in train_data.columns if col != 'target']]
test_data = test_data[['target'] + [col for col in test_data.columns if col != 'target']]

# Save locally
train_data.to_csv('train.csv', index=False, header=False)
test_data.to_csv('test.csv', index=False, header=False)

print(f"Training samples: {len(train_data)}")
print(f"Test samples: {len(test_data)}")

# Upload to S3
prefix = 'mla-c01/distributed-training'
train_s3 = session.upload_data('train.csv', bucket=bucket, key_prefix=f'{prefix}/train')
test_s3 = session.upload_data('test.csv', bucket=bucket, key_prefix=f'{prefix}/test')

print(f"\nTraining data: {train_s3}")
print(f"Test data: {test_s3}")
```

#### Step 3: Configure Distributed Training Job

```python
from sagemaker.pytorch import PyTorch

# Save training script
with open('train_distributed.py', 'w') as f:
    f.write(train_distributed_script)  # Use the script content from Step 1

# Create PyTorch estimator with distributed training
pytorch_estimator = PyTorch(
    entry_point='train_distributed.py',
    role=role,
    instance_count=2,  # Use 2 instances for distribution
    instance_type='ml.p3.2xlarge',  # GPU instance
    framework_version='2.0',
    py_version='py310',
    hyperparameters={
        'epochs': 20,
        'batch-size': 128,
        'lr': 0.001,
        'hidden-dim': 256
    },
    distribution={
        'pytorchddp': {
            'enabled': True
        }
    },
    output_path=f's3://{bucket}/{prefix}/output',
    base_job_name='mla-distributed-training'
)

print("Estimator configured for distributed training:")
print(f"  Instance count: {pytorch_estimator.instance_count}")
print(f"  Instance type: {pytorch_estimator.instance_type}")
print(f"  Distribution: PyTorch DDP enabled")
```

#### Step 4: Start Distributed Training

```python
# Start training
pytorch_estimator.fit({'training': train_s3}, wait=False)

print(f"Distributed training job started: {pytorch_estimator.latest_training_job.name}")
print("Training on 2 instances with data parallelism")
```

#### Step 5: Monitor Training Progress

```python
import boto3
import time

sm_client = boto3.client('sagemaker')

job_name = pytorch_estimator.latest_training_job.name

while True:
    response = sm_client.describe_training_job(TrainingJobName=job_name)
    status = response['TrainingJobStatus']

    print(f"Status: {status}")

    if status in ['Completed', 'Failed', 'Stopped']:
        print(f"\nFinal status: {status}")
        if status == 'Completed':
            print(f"Training time: {response['TrainingTimeInSeconds']} seconds")
            print(f"Billable time: {response['BillableTimeInSeconds']} seconds")
        break

    time.sleep(30)
```

#### Step 6: Compare Single vs Distributed Training

```python
# Train same model on single instance for comparison
single_estimator = PyTorch(
    entry_point='train_distributed.py',
    role=role,
    instance_count=1,  # Single instance
    instance_type='ml.p3.2xlarge',
    framework_version='2.0',
    py_version='py310',
    hyperparameters={
        'epochs': 20,
        'batch-size': 128,
        'lr': 0.001,
        'hidden-dim': 256
    },
    output_path=f's3://{bucket}/{prefix}/output-single',
    base_job_name='mla-single-training'
)

# Start single-instance training
single_estimator.fit({'training': train_s3}, wait=False)

print("Started single-instance training for comparison")
```

### Lab Questions

1. What speedup did you observe with distributed training?
2. When is distributed training cost-effective?
3. What are the trade-offs between data parallelism and model parallelism?

---

## Lab 9: Multi-Model Endpoints

### Objective
Deploy multiple models to a single endpoint to reduce costs and improve resource utilization.

### Duration
60 minutes

### Use Case
Deploy different customer segmentation models (by region) to one endpoint.

### Steps

#### Step 1: Train Multiple Models

Create notebook: `lab9-multi-model-endpoint.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
import boto3
import pandas as pd
import numpy as np

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
prefix = 'mla-c01/multi-model'

# Train models for different regions
regions = ['us-east', 'us-west', 'europe', 'asia']

for region in regions:
    print(f"\nTraining model for {region}...")

    # Create region-specific data
    np.random.seed(hash(region) % 2**32)
    n_samples = 1000

    data = pd.DataFrame({
        'feature_1': np.random.randn(n_samples) + (regions.index(region) * 0.5),
        'feature_2': np.random.randn(n_samples),
        'feature_3': np.random.randn(n_samples),
        'label': np.random.randint(0, 2, n_samples)
    })

    # Reorder: label first
    data = data[['label', 'feature_1', 'feature_2', 'feature_3']]

    # Save and upload
    filename = f'train_{region}.csv'
    data.to_csv(filename, index=False, header=False)

    s3_path = session.upload_data(filename, bucket=bucket, key_prefix=f'{prefix}/data')

    # Train XGBoost model
    from sagemaker.estimator import Estimator

    container = sagemaker.image_uris.retrieve('xgboost', session.boto_region_name, version='1.7-1')

    estimator = Estimator(
        image_uri=container,
        role=role,
        instance_count=1,
        instance_type='ml.m5.xlarge',
        output_path=f's3://{bucket}/{prefix}/models/{region}',
        sagemaker_session=session,
        base_job_name=f'mla-mme-{region}'
    )

    estimator.set_hyperparameters(
        objective='binary:logistic',
        num_round=50,
        max_depth=5
    )

    estimator.fit({'train': s3_path}, wait=True)
    print(f"  Model trained and saved to: {estimator.model_data}")
```

#### Step 2: Prepare Models for Multi-Model Endpoint

```python
# Copy model artifacts to multi-model location
s3_client = boto3.client('s3')

multi_model_prefix = f'{prefix}/multi-model-artifacts'

for region in regions:
    # Get the latest training job for this region
    sm_client = boto3.client('sagemaker')

    training_jobs = sm_client.list_training_jobs(
        NameContains=f'mla-mme-{region}',
        SortBy='CreationTime',
        SortOrder='Descending',
        MaxResults=1
    )

    if training_jobs['TrainingJobSummaries']:
        job_name = training_jobs['TrainingJobSummaries'][0]['TrainingJobName']
        job_details = sm_client.describe_training_job(TrainingJobName=job_name)
        model_s3_uri = job_details['ModelArtifacts']['S3ModelArtifacts']

        # Copy model to multi-model location
        source_key = model_s3_uri.replace(f's3://{bucket}/', '')
        target_key = f'{multi_model_prefix}/{region}/model.tar.gz'

        copy_source = {'Bucket': bucket, 'Key': source_key}
        s3_client.copy_object(CopySource=copy_source, Bucket=bucket, Key=target_key)

        print(f"Copied {region} model to: s3://{bucket}/{target_key}")

multi_model_s3_uri = f's3://{bucket}/{multi_model_prefix}/'
print(f"\nAll models available at: {multi_model_s3_uri}")
```

#### Step 3: Create Multi-Model Endpoint

```python
from sagemaker.multidatamodel import MultiDataModel

# Get XGBoost container
container = sagemaker.image_uris.retrieve('xgboost', session.boto_region_name, version='1.7-1')

# Create MultiDataModel
multi_model_name = 'mla-multi-model-endpoint'

mdm = MultiDataModel(
    name=multi_model_name,
    model_data_prefix=multi_model_s3_uri,
    image_uri=container,
    role=role,
    sagemaker_session=session
)

print(f"Multi-model created: {multi_model_name}")
```

#### Step 4: Deploy Multi-Model Endpoint

```python
# Deploy endpoint
predictor = mdm.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    endpoint_name='mla-multi-model-endpoint'
)

print(f"Multi-model endpoint deployed: {predictor.endpoint_name}")
print("This single endpoint can serve all regional models")
```

#### Step 5: Invoke Different Models

```python
from sagemaker.serializers import CSVSerializer
from sagemaker.deserializers import JSONDeserializer

predictor.serializer = CSVSerializer()
predictor.deserializer = JSONDeserializer()

# Test each regional model
test_samples = {
    'us-east': [1.5, 0.3, -0.2],
    'us-west': [2.0, -0.5, 0.8],
    'europe': [2.5, 0.1, -0.6],
    'asia': [3.0, -0.2, 0.4]
}

for region, features in test_samples.items():
    # Specify which model to use via target_model parameter
    model_path = f'{region}/model.tar.gz'

    prediction = predictor.predict(
        data=features,
        target_model=model_path
    )

    print(f"\nRegion: {region}")
    print(f"  Features: {features}")
    print(f"  Prediction: {prediction:.4f}")
    print(f"  Class: {'Positive' if prediction > 0.5 else 'Negative'}")
```

#### Step 6: Add New Model Dynamically

```python
# Train a new model for a new region
print("\nAdding new model for 'south-america' region...")

np.random.seed(12345)
n_samples = 1000

new_data = pd.DataFrame({
    'feature_1': np.random.randn(n_samples) + 4.0,
    'feature_2': np.random.randn(n_samples),
    'feature_3': np.random.randn(n_samples),
    'label': np.random.randint(0, 2, n_samples)
})

new_data = new_data[['label', 'feature_1', 'feature_2', 'feature_3']]
new_data.to_csv('train_south_america.csv', index=False, header=False)

# Train model
s3_path = session.upload_data('train_south_america.csv', bucket=bucket, key_prefix=f'{prefix}/data')

estimator = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/{prefix}/models/south-america',
    sagemaker_session=session
)

estimator.set_hyperparameters(
    objective='binary:logistic',
    num_round=50,
    max_depth=5
)

estimator.fit({'train': s3_path}, wait=True)

# Add to multi-model endpoint (no redeployment needed!)
mdm.add_model(
    model_data_source=estimator.model_data,
    model_data_path='south-america/model.tar.gz'
)

print("New model added to endpoint without redeployment!")

# Test new model
new_prediction = predictor.predict(
    data=[4.5, 0.2, -0.3],
    target_model='south-america/model.tar.gz'
)

print(f"South America model prediction: {new_prediction:.4f}")
```

#### Step 7: Monitor Model Usage

```python
import boto3
from datetime import datetime, timedelta

cloudwatch = boto3.client('cloudwatch')

# Get metrics for model invocations
end_time = datetime.now()
start_time = end_time - timedelta(hours=1)

for region in regions + ['south-america']:
    metrics = cloudwatch.get_metric_statistics(
        Namespace='AWS/SageMaker',
        MetricName='ModelInvocations',
        Dimensions=[
            {'Name': 'EndpointName', 'Value': predictor.endpoint_name},
            {'Name': 'ModelName', 'Value': f'{region}/model.tar.gz'}
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=3600,
        Statistics=['Sum']
    )

    total_invocations = sum([d['Sum'] for d in metrics['Datapoints']])
    print(f"{region}: {total_invocations} invocations")
```

### Lab Questions

1. What cost savings can multi-model endpoints provide?
2. When should you use multi-model endpoints vs separate endpoints?
3. What are the performance considerations?

---

## Lab 10: A/B Testing with Production Variants

### Objective
Implement A/B testing to safely deploy and test new model versions.

### Duration
45 minutes

### Steps

#### Step 1: Prepare Two Model Versions

Create notebook: `lab10-ab-testing.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.model import Model
from sagemaker.predictor import Predictor

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
prefix = 'mla-c01/ab-testing'

# Train Model A (conservative parameters)
from sagemaker.estimator import Estimator

container = sagemaker.image_uris.retrieve('xgboost', session.boto_region_name, version='1.7-1')

# Prepare data (reuse from previous labs)
train_s3 = f's3://{bucket}/mla-c01/churn-prediction/train/train.csv'

# Model A: Conservative
estimator_a = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/{prefix}/model-a',
    base_job_name='mla-ab-model-a'
)

estimator_a.set_hyperparameters(
    objective='binary:logistic',
    num_round=50,
    max_depth=3,  # Shallow trees
    eta=0.1       # Slow learning rate
)

estimator_a.fit({'train': train_s3}, wait=True)
print(f"Model A trained: {estimator_a.model_data}")

# Model B: Aggressive
estimator_b = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path=f's3://{bucket}/{prefix}/model-b',
    base_job_name='mla-ab-model-b'
)

estimator_b.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    max_depth=7,  # Deeper trees
    eta=0.3       # Faster learning rate
)

estimator_b.fit({'train': train_s3}, wait=True)
print(f"Model B trained: {estimator_b.model_data}")
```

#### Step 2: Create Models

```python
from sagemaker.model import Model
from datetime import datetime

timestamp = datetime.now().strftime('%Y-%m-%d-%H-%M-%S')

# Create Model A
model_a = Model(
    image_uri=container,
    model_data=estimator_a.model_data,
    role=role,
    name=f'mla-ab-model-a-{timestamp}',
    sagemaker_session=session
)

# Create Model B
model_b = Model(
    image_uri=container,
    model_data=estimator_b.model_data,
    role=role,
    name=f'mla-ab-model-b-{timestamp}',
    sagemaker_session=session
)

print(f"Model A: {model_a.name}")
print(f"Model B: {model_b.name}")
```

#### Step 3: Deploy with Production Variants

```python
from sagemaker.model_monitor import DataCaptureConfig

# Configure data capture for monitoring
data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,
    destination_s3_uri=f's3://{bucket}/{prefix}/data-capture'
)

# Deploy with two variants
# Variant A: 70% traffic
# Variant B: 30% traffic (testing new model)

endpoint_name = 'mla-ab-testing-endpoint'

predictor = model_a.deploy(
    endpoint_name=endpoint_name,
    initial_instance_count=1,
    instance_type='ml.m5.large',
    variant_name='ModelA',
    initial_weight=70,  # 70% of traffic
    data_capture_config=data_capture_config
)

# Add variant B to same endpoint
from sagemaker.session import production_variant

variant_b = production_variant(
    model_name=model_b.name,
    instance_type='ml.m5.large',
    initial_instance_count=1,
    variant_name='ModelB',
    initial_weight=30  # 30% of traffic
)

# Update endpoint to add variant B
sm_client = boto3.client('sagemaker')

# Get current endpoint config
endpoint_description = sm_client.describe_endpoint(EndpointName=endpoint_name)
current_config = endpoint_description['EndpointConfigName']

# Create new config with both variants
new_config_name = f'mla-ab-config-{timestamp}'

sm_client.create_endpoint_config(
    EndpointConfigName=new_config_name,
    ProductionVariants=[
        {
            'VariantName': 'ModelA',
            'ModelName': model_a.name,
            'InitialInstanceCount': 1,
            'InstanceType': 'ml.m5.large',
            'InitialVariantWeight': 70
        },
        {
            'VariantName': 'ModelB',
            'ModelName': model_b.name,
            'InitialInstanceCount': 1,
            'InstanceType': 'ml.m5.large',
            'InitialVariantWeight': 30
        }
    ],
    DataCaptureConfig={
        'EnableCapture': True,
        'InitialSamplingPercentage': 100,
        'DestinationS3Uri': f's3://{bucket}/{prefix}/data-capture',
        'CaptureOptions': [
            {'CaptureMode': 'Input'},
            {'CaptureMode': 'Output'}
        ]
    }
)

# Update endpoint
sm_client.update_endpoint(
    EndpointName=endpoint_name,
    EndpointConfigName=new_config_name
)

print(f"Endpoint updated with A/B testing:")
print(f"  Model A (variant): 70% traffic")
print(f"  Model B (variant): 30% traffic")

# Wait for update
waiter = sm_client.get_waiter('endpoint_in_service')
waiter.wait(EndpointName=endpoint_name)
print("Endpoint update complete!")
```

#### Step 4: Generate Test Traffic

```python
from sagemaker.serializers import CSVSerializer
from sagemaker.deserializers import JSONDeserializer
import time
import numpy as np

predictor = Predictor(
    endpoint_name=endpoint_name,
    sagemaker_session=session,
    serializer=CSVSerializer(),
    deserializer=JSONDeserializer()
)

# Generate 100 test requests
print("Generating test traffic...")

variant_a_count = 0
variant_b_count = 0

for i in range(100):
    # Random test sample
    sample = [
        np.random.randint(1, 72),
        np.random.uniform(20, 120),
        np.random.uniform(100, 8000),
        np.random.choice([0, 1, 2]),
        np.random.choice([0, 1, 2, 3]),
        np.random.choice([0, 1, 2]),
        np.random.choice([0, 1])
    ]

    # Make prediction
    response = predictor.predict(sample)

    # Note: We can't directly see which variant handled the request
    # without checking CloudWatch logs

    if (i + 1) % 20 == 0:
        print(f"  Sent {i + 1} requests")

    time.sleep(0.1)

print("Traffic generation complete!")
```

#### Step 5: Monitor Variant Performance

```python
import boto3
from datetime import datetime, timedelta
import pandas as pd

cloudwatch = boto3.client('cloudwatch')

end_time = datetime.now()
start_time = end_time - timedelta(minutes=30)

# Get metrics for each variant
variants = ['ModelA', 'ModelB']
metrics_data = []

for variant in variants:
    # Invocations
    invocations = cloudwatch.get_metric_statistics(
        Namespace='AWS/SageMaker',
        MetricName='ModelInvocations',
        Dimensions=[
            {'Name': 'EndpointName', 'Value': endpoint_name},
            {'Name': 'VariantName', 'Value': variant}
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=300,
        Statistics=['Sum']
    )

    # Latency
    latency = cloudwatch.get_metric_statistics(
        Namespace='AWS/SageMaker',
        MetricName='ModelLatency',
        Dimensions=[
            {'Name': 'EndpointName', 'Value': endpoint_name},
            {'Name': 'VariantName', 'Value': variant}
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=300,
        Statistics=['Average', 'Maximum']
    )

    total_invocations = sum([d['Sum'] for d in invocations['Datapoints']])
    avg_latency = sum([d['Average'] for d in latency['Datapoints']]) / max(len(latency['Datapoints']), 1)

    metrics_data.append({
        'Variant': variant,
        'Invocations': total_invocations,
        'Avg Latency (ms)': avg_latency
    })

metrics_df = pd.DataFrame(metrics_data)
print("\nVariant Performance Metrics:")
print(metrics_df)

# Verify traffic split
if metrics_df['Invocations'].sum() > 0:
    for idx, row in metrics_df.iterrows():
        percentage = (row['Invocations'] / metrics_df['Invocations'].sum()) * 100
        print(f"{row['Variant']}: {percentage:.1f}% of traffic")
```

#### Step 6: Shift Traffic Based on Performance

```python
# If Model B performs better, shift more traffic to it

# Example: Shift to 50-50
print("\nShifting traffic to 50-50 split...")

sm_client.update_endpoint_weights_and_capacities(
    EndpointName=endpoint_name,
    DesiredWeightsAndCapacities=[
        {
            'VariantName': 'ModelA',
            'DesiredWeight': 50
        },
        {
            'VariantName': 'ModelB',
            'DesiredWeight': 50
        }
    ]
)

print("Traffic weights updated to 50-50")

# Or shift all traffic to Model B
print("\nShifting all traffic to Model B...")

sm_client.update_endpoint_weights_and_capacities(
    EndpointName=endpoint_name,
    DesiredWeightsAndCapacities=[
        {
            'VariantName': 'ModelA',
            'DesiredWeight': 0
        },
        {
            'VariantName': 'ModelB',
            'DesiredWeight': 100
        }
    ]
)

print("All traffic now routed to Model B")
```

#### Step 7: Automate Variant Analysis

```python
# Create function to analyze variant performance
def analyze_variants(endpoint_name, hours=1):
    """Analyze variant performance and recommend traffic split"""
    end_time = datetime.now()
    start_time = end_time - timedelta(hours=hours)

    results = []

    for variant in ['ModelA', 'ModelB']:
        # Get errors
        errors = cloudwatch.get_metric_statistics(
            Namespace='AWS/SageMaker',
            MetricName='ModelInvocationErrors',
            Dimensions=[
                {'Name': 'EndpointName', 'Value': endpoint_name},
                {'Name': 'VariantName', 'Value': variant}
            ],
            StartTime=start_time,
            EndTime=end_time,
            Period=3600,
            Statistics=['Sum']
        )

        # Get invocations
        invocations = cloudwatch.get_metric_statistics(
            Namespace='AWS/SageMaker',
            MetricName='ModelInvocations',
            Dimensions=[
                {'Name': 'EndpointName', 'Value': endpoint_name},
                {'Name': 'VariantName', 'Value': variant}
            ],
            StartTime=start_time,
            EndTime=end_time,
            Period=3600,
            Statistics=['Sum']
        )

        # Get latency
        latency = cloudwatch.get_metric_statistics(
            Namespace='AWS/SageMaker',
            MetricName='ModelLatency',
            Dimensions=[
                {'Name': 'EndpointName', 'Value': endpoint_name},
                {'Name': 'VariantName', 'Value': variant}
            ],
            StartTime=start_time,
            EndTime=end_time,
            Period=3600,
            Statistics=['Average']
        )

        total_errors = sum([d['Sum'] for d in errors['Datapoints']])
        total_invocations = sum([d['Sum'] for d in invocations['Datapoints']])
        avg_latency = sum([d['Average'] for d in latency['Datapoints']]) / max(len(latency['Datapoints']), 1)

        error_rate = (total_errors / max(total_invocations, 1)) * 100

        results.append({
            'Variant': variant,
            'Invocations': total_invocations,
            'Errors': total_errors,
            'Error Rate (%)': error_rate,
            'Avg Latency (ms)': avg_latency,
            'Score': 100 - error_rate - (avg_latency / 10)  # Simple scoring
        })

    df = pd.DataFrame(results)

    print("\nVariant Analysis:")
    print(df)

    # Recommendation
    best_variant = df.loc[df['Score'].idxmax(), 'Variant']
    print(f"\nRecommendation: Shift more traffic to {best_variant}")

    return df

# Run analysis
variant_analysis = analyze_variants(endpoint_name)
```

### Lab Questions

1. How do you decide when to shift traffic between variants?
2. What metrics are most important for A/B testing?
3. How long should an A/B test run before making decisions?

---

## Lab 11: Automated Retraining Pipeline

### Objective
Build a fully automated pipeline that retrains models on schedule or when data drift is detected.

### Duration
75 minutes

### Steps

#### Step 1: Create Retraining Pipeline

Create notebook: `lab11-automated-retraining.ipynb`

```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import (
    ProcessingStep,
    TrainingStep,
    CreateModelStep,
    TransformStep
)
from sagemaker.workflow.parameters import (
    ParameterInteger,
    ParameterString,
    ParameterFloat
)
from sagemaker.workflow.conditions import ConditionGreaterThan
from sagemaker.workflow.condition_step import ConditionStep
from sagemaker.workflow.functions import JsonGet
from sagemaker.workflow.properties import PropertyFile
from sagemaker.workflow.lambda_step import LambdaStep, Lambda
from sagemaker.lambda_helper import Lambda as LambdaHelper
from sagemaker.processing import ProcessingInput, ProcessingOutput
from sagemaker.sklearn.processing import SKLearnProcessor
import boto3

session = sagemaker.Session()
role = get_execution_role()
bucket = session.default_bucket()
region = session.boto_region_name

# Pipeline parameters
model_approval_threshold = ParameterFloat(
    name="ModelApprovalThreshold",
    default_value=0.85
)

training_instance_type = ParameterString(
    name="TrainingInstanceType",
    default_value="ml.m5.xlarge"
)

input_data_path = ParameterString(
    name="InputDataPath",
    default_value=f"s3://{bucket}/mla-c01/retraining/input/"
)
```

#### Step 2: Create Data Quality Check Step

```python
# Data quality check script
data_quality_script = """
import json
import pandas as pd
import numpy as np
from pathlib import Path

def check_data_quality(input_path, output_path):
    # Read data
    df = pd.read_csv(input_path)

    quality_report = {
        'num_rows': len(df),
        'num_cols': len(df.columns),
        'missing_percentage': (df.isnull().sum().sum() / (len(df) * len(df.columns))) * 100,
        'duplicate_rows': df.duplicated().sum(),
        'quality_score': 0.0
    }

    # Calculate quality score
    score = 100.0
    score -= quality_report['missing_percentage'] * 2  # Penalize missing data
    score -= (quality_report['duplicate_rows'] / len(df)) * 100 * 5  # Penalize duplicates
    quality_report['quality_score'] = max(0, score)

    # Save report
    Path(output_path).mkdir(parents=True, exist_ok=True)
    with open(f'{output_path}/quality_report.json', 'w') as f:
        json.dump(quality_report, f)

    print(f"Data Quality Score: {quality_report['quality_score']:.2f}")
    print(f"Rows: {quality_report['num_rows']}, Missing: {quality_report['missing_percentage']:.2f}%")

    return quality_report

if __name__ == '__main__':
    check_data_quality('/opt/ml/processing/input', '/opt/ml/processing/output')
"""

# Save script
with open('check_data_quality.py', 'w') as f:
    f.write(data_quality_script)

# Upload script
quality_script_uri = session.upload_data(
    'check_data_quality.py',
    bucket=bucket,
    key_prefix='mla-c01/retraining/scripts'
)

# Create processor
quality_processor = SKLearnProcessor(
    framework_version='1.0-1',
    role=role,
    instance_type='ml.m5.xlarge',
    instance_count=1
)

# Create quality report property
quality_report = PropertyFile(
    name="DataQualityReport",
    output_name="quality",
    path="quality_report.json"
)

# Define quality check step
step_quality_check = ProcessingStep(
    name="CheckDataQuality",
    processor=quality_processor,
    inputs=[
        ProcessingInput(
            source=input_data_path,
            destination="/opt/ml/processing/input"
        )
    ],
    outputs=[
        ProcessingOutput(
            output_name="quality",
            source="/opt/ml/processing/output"
        )
    ],
    code=quality_script_uri,
    property_files=[quality_report]
)
```

#### Step 3: Create Drift Detection Step

```python
# Drift detection script
drift_detection_script = """
import json
import pandas as pd
import numpy as np
from pathlib import Path
from scipy import stats

def detect_drift(current_data_path, baseline_stats_path, output_path):
    # Read current data
    current_df = pd.read_csv(current_data_path)

    # Load baseline statistics (mock for this example)
    # In production, load from previous model training
    baseline_stats = {
        'mean': current_df.mean().to_dict(),
        'std': current_df.std().to_dict()
    }

    # Calculate drift score using KL divergence or similar
    drift_scores = {}

    for col in current_df.select_dtypes(include=[np.number]).columns:
        # Simple drift detection: compare means
        current_mean = current_df[col].mean()
        current_std = current_df[col].std()

        # Mock baseline (in production, use actual baseline)
        baseline_mean = current_mean + np.random.uniform(-0.5, 0.5)
        baseline_std = current_std + np.random.uniform(-0.1, 0.1)

        # Calculate drift (normalized difference)
        if baseline_std > 0:
            drift = abs((current_mean - baseline_mean) / baseline_std)
        else:
            drift = 0

        drift_scores[col] = drift

    # Overall drift score
    overall_drift = np.mean(list(drift_scores.values()))

    drift_report = {
        'overall_drift_score': float(overall_drift),
        'feature_drift_scores': drift_scores,
        'drift_detected': overall_drift > 0.5,  # Threshold
        'recommendation': 'retrain' if overall_drift > 0.5 else 'no_action'
    }

    # Save report
    Path(output_path).mkdir(parents=True, exist_ok=True)
    with open(f'{output_path}/drift_report.json', 'w') as f:
        json.dump(drift_report, f)

    print(f"Drift Score: {overall_drift:.4f}")
    print(f"Drift Detected: {drift_report['drift_detected']}")
    print(f"Recommendation: {drift_report['recommendation']}")

    return drift_report

if __name__ == '__main__':
    detect_drift(
        '/opt/ml/processing/current/data.csv',
        '/opt/ml/processing/baseline/',
        '/opt/ml/processing/output'
    )
"""

# Save and upload
with open('detect_drift.py', 'w') as f:
    f.write(drift_detection_script)

drift_script_uri = session.upload_data(
    'detect_drift.py',
    bucket=bucket,
    key_prefix='mla-c01/retraining/scripts'
)

# Create drift processor
drift_processor = SKLearnProcessor(
    framework_version='1.0-1',
    role=role,
    instance_type='ml.m5.xlarge',
    instance_count=1
)

# Create drift report property
drift_report = PropertyFile(
    name="DriftReport",
    output_name="drift",
    path="drift_report.json"
)

# Define drift detection step
step_drift_check = ProcessingStep(
    name="DetectDataDrift",
    processor=drift_processor,
    inputs=[
        ProcessingInput(
            source=input_data_path,
            destination="/opt/ml/processing/current"
        )
    ],
    outputs=[
        ProcessingOutput(
            output_name="drift",
            source="/opt/ml/processing/output"
        )
    ],
    code=drift_script_uri,
    property_files=[drift_report]
)
```

#### Step 4: Create Training Step (Only if Drift Detected)

```python
from sagemaker.estimator import Estimator
from sagemaker.inputs import TrainingInput

container = sagemaker.image_uris.retrieve('xgboost', region, version='1.7-1')

# Create estimator
xgb_estimator = Estimator(
    image_uri=container,
    role=role,
    instance_count=1,
    instance_type=training_instance_type,
    output_path=f's3://{bucket}/mla-c01/retraining/models',
    base_job_name='mla-automated-retraining'
)

xgb_estimator.set_hyperparameters(
    objective='binary:logistic',
    num_round=100,
    max_depth=5,
    eta=0.2
)

# Training step
step_train = TrainingStep(
    name="TrainModel",
    estimator=xgb_estimator,
    inputs={
        "train": TrainingInput(
            s3_data=input_data_path,
            content_type="text/csv"
        )
    }
)
```

#### Step 5: Create Conditional Retraining Logic

```python
from sagemaker.workflow.conditions import ConditionGreaterThan
from sagemaker.workflow.functions import JsonGet

# Condition: Retrain if drift score > 0.3
cond_drift_detected = ConditionGreaterThan(
    left=JsonGet(
        step_name=step_drift_check.name,
        property_file=drift_report,
        json_path="overall_drift_score"
    ),
    right=0.3
)

# Conditional step
step_cond_retrain = ConditionStep(
    name="CheckIfRetrainingNeeded",
    conditions=[cond_drift_detected],
    if_steps=[step_train],
    else_steps=[]
)
```

#### Step 6: Create Notification Step

```python
# Create Lambda function for SNS notification
lambda_code = """
import json
import boto3

def lambda_handler(event, context):
    sns = boto3.client('sns')

    drift_score = event.get('drift_score', 0)
    retrained = event.get('retrained', False)

    message = f'''
    ML Pipeline Execution Complete

    Drift Score: {drift_score}
    Retraining Triggered: {retrained}

    Pipeline Execution ARN: {event.get('execution_arn', 'N/A')}
    '''

    # Note: Create SNS topic first and update ARN
    topic_arn = 'arn:aws:sns:us-east-1:123456789012:ml-pipeline-notifications'

    try:
        sns.publish(
            TopicArn=topic_arn,
            Subject='ML Pipeline Execution Complete',
            Message=message
        )
        return {'statusCode': 200, 'body': 'Notification sent'}
    except Exception as e:
        print(f'Error sending notification: {e}')
        return {'statusCode': 500, 'body': str(e)}
"""

# Note: In production, create Lambda function via AWS Console or CloudFormation
# This is a placeholder to show the concept
```

#### Step 7: Build and Execute Pipeline

```python
# Create pipeline
retraining_pipeline = Pipeline(
    name="mla-automated-retraining-pipeline",
    parameters=[
        model_approval_threshold,
        training_instance_type,
        input_data_path
    ],
    steps=[
        step_quality_check,
        step_drift_check,
        step_cond_retrain
    ],
    sagemaker_session=session
)

# Create/update pipeline
retraining_pipeline.upsert(role_arn=role)
print(f"Pipeline created: {retraining_pipeline.name}")

# Execute pipeline
execution = retraining_pipeline.start()
print(f"Pipeline execution started: {execution.arn}")
```

#### Step 8: Schedule Pipeline with EventBridge

```python
import boto3
import json

events_client = boto3.client('events')

# Create EventBridge rule to run pipeline daily
rule_name = 'mla-daily-retraining-check'

# Create rule
events_client.put_rule(
    Name=rule_name,
    ScheduleExpression='cron(0 2 * * ? *)',  # Run at 2 AM UTC daily
    State='ENABLED',
    Description='Trigger ML retraining pipeline daily'
)

# Add SageMaker Pipeline as target
pipeline_arn = f'arn:aws:sagemaker:{region}:{session.account_id()}:pipeline/{retraining_pipeline.name}'

events_client.put_targets(
    Rule=rule_name,
    Targets=[
        {
            'Id': '1',
            'Arn': pipeline_arn,
            'RoleArn': role,
            'SageMakerPipelineParameters': {
                'PipelineParameterList': [
                    {
                        'Name': 'InputDataPath',
                        'Value': f's3://{bucket}/mla-c01/retraining/daily-data/'
                    }
                ]
            }
        }
    ]
)

print(f"EventBridge rule created: {rule_name}")
print("Pipeline will run daily at 2 AM UTC")
```

### Lab Questions

1. What triggers should you use for automated retraining?
2. How do you prevent unnecessary retraining?
3. What monitoring is needed for automated pipelines?

---

## Lab 12: Real-Time Feature Engineering with Lambda

### Objective
Implement real-time feature engineering for inference using AWS Lambda.

### Duration
60 minutes

### Steps

#### Step 1: Create Feature Engineering Lambda

Create notebook: `lab12-realtime-features.ipynb`

```python
# Lambda function code for feature engineering
lambda_feature_engineering = """
import json
import boto3
import numpy as np
from datetime import datetime

def lambda_handler(event, context):
    '''
    Real-time feature engineering Lambda function
    Input: Raw customer data
    Output: Engineered features ready for model inference
    '''

    # Parse input
    body = json.loads(event.get('body', '{}'))

    # Raw features
    tenure_months = body.get('tenure_months', 0)
    monthly_charges = body.get('monthly_charges', 0)
    contract_type = body.get('contract_type', 'month')  # month, year, 2year
    support_calls_30d = body.get('support_calls_30d', 0)
    data_usage_gb = body.get('data_usage_gb', 0)

    # Feature engineering
    features = {}

    # 1. Tenure features
    features['tenure_months'] = tenure_months
    features['tenure_years'] = tenure_months / 12.0
    features['is_new_customer'] = 1 if tenure_months < 6 else 0
    features['is_long_term'] = 1 if tenure_months > 24 else 0

    # 2. Financial features
    features['monthly_charges'] = monthly_charges
    total_charges = tenure_months * monthly_charges
    features['total_charges'] = total_charges
    features['charges_per_month'] = monthly_charges
    features['avg_monthly_value'] = total_charges / max(tenure_months, 1)

    # 3. Contract features
    contract_mapping = {'month': 0, 'year': 1, '2year': 2}
    features['contract_type_encoded'] = contract_mapping.get(contract_type, 0)
    features['is_contract'] = 1 if contract_type != 'month' else 0

    # 4. Usage features
    features['support_calls_30d'] = support_calls_30d
    features['high_support_user'] = 1 if support_calls_30d > 3 else 0
    features['data_usage_gb'] = data_usage_gb
    features['heavy_user'] = 1 if data_usage_gb > 10 else 0

    # 5. Interaction features
    features['charges_per_gb'] = monthly_charges / max(data_usage_gb, 1)
    features['support_intensity'] = support_calls_30d / max(tenure_months, 1)

    # 6. Risk features
    risk_score = 0
    if features['is_new_customer']:
        risk_score += 20
    if contract_type == 'month':
        risk_score += 30
    if support_calls_30d > 5:
        risk_score += 25
    if monthly_charges > 100:
        risk_score += 15
    features['churn_risk_score'] = min(risk_score, 100)

    # 7. Time-based features
    now = datetime.now()
    features['day_of_week'] = now.weekday()
    features['hour_of_day'] = now.hour
    features['is_weekend'] = 1 if now.weekday() >= 5 else 0

    # Convert to model input format (values only, in order)
    feature_vector = [
        features['tenure_months'],
        features['monthly_charges'],
        features['total_charges'],
        features['contract_type_encoded'],
        features['support_calls_30d'],
        features['data_usage_gb'],
        features['is_new_customer'],
        features['high_support_user'],
        features['churn_risk_score']
    ]

    return {
        'statusCode': 200,
        'body': json.dumps({
            'raw_features': body,
            'engineered_features': features,
            'model_input': feature_vector
        })
    }
"""

print("Feature engineering Lambda function code created")
```

#### Step 2: Create Lambda Function

```python
import boto3
import zipfile
import io

lambda_client = boto3.client('lambda')
iam_client = boto3.client('iam')

# Create IAM role for Lambda
role_policy = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

role_name = 'mla-feature-engineering-lambda-role'

try:
    role_response = iam_client.create_role(
        RoleName=role_name,
        AssumeRolePolicyDocument=json.dumps(role_policy),
        Description='Role for feature engineering Lambda function'
    )
    role_arn = role_response['Role']['Arn']
    print(f"Created IAM role: {role_arn}")

    # Attach basic Lambda execution policy
    iam_client.attach_role_policy(
        RoleName=role_name,
        PolicyArn='arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'
    )

    # Wait for role to be ready
    import time
    time.sleep(10)

except iam_client.exceptions.EntityAlreadyExistsException:
    role_arn = iam_client.get_role(RoleName=role_name)['Role']['Arn']
    print(f"Using existing role: {role_arn}")

# Create deployment package
zip_buffer = io.BytesIO()
with zipfile.ZipFile(zip_buffer, 'w', zipfile.ZIP_DEFLATED) as zip_file:
    zip_file.writestr('lambda_function.py', lambda_feature_engineering)

zip_buffer.seek(0)

# Create Lambda function
function_name = 'mla-feature-engineering'

try:
    response = lambda_client.create_function(
        FunctionName=function_name,
        Runtime='python3.11',
        Role=role_arn,
        Handler='lambda_function.lambda_handler',
        Code={'ZipFile': zip_buffer.read()},
        Description='Real-time feature engineering for ML inference',
        Timeout=30,
        MemorySize=256
    )
    print(f"Lambda function created: {response['FunctionArn']}")
except lambda_client.exceptions.ResourceConflictException:
    print(f"Lambda function {function_name} already exists")
    response = lambda_client.get_function(FunctionName=function_name)
```

#### Step 3: Test Feature Engineering Lambda

```python
import json

# Test the Lambda function
test_event = {
    'body': json.dumps({
        'tenure_months': 24,
        'monthly_charges': 85.50,
        'contract_type': 'year',
        'support_calls_30d': 2,
        'data_usage_gb': 15.5
    })
}

response = lambda_client.invoke(
    FunctionName=function_name,
    InvocationType='RequestResponse',
    Payload=json.dumps(test_event)
)

result = json.loads(response['Payload'].read())
print("Lambda Response:")
print(json.dumps(json.loads(result['body']), indent=2))
```

#### Step 4: Create Serial Inference Pipeline

```python
from sagemaker.model import Model
from sagemaker.pipeline import PipelineModel
from sagemaker.sparkml import SparkMLModel

# Assuming we have a trained model from previous labs
model_data = f's3://{bucket}/mla-c01/churn-prediction/output/model.tar.gz'

container = sagemaker.image_uris.retrieve('xgboost', region, version='1.7-1')

# Create inference model
inference_model = Model(
    image_uri=container,
    model_data=model_data,
    role=role,
    sagemaker_session=session
)

# In production, you would create a PipelineModel that calls:
# 1. Lambda for feature engineering
# 2. SageMaker model for prediction

# For this lab, we'll use Lambda + SageMaker endpoint separately
```

#### Step 5: Create API Gateway Integration

```python
# Create API Gateway to expose Lambda + SageMaker as a single endpoint
apigateway_client = boto3.client('apigateway')

# Create REST API
api_response = apigateway_client.create_rest_api(
    name='mla-ml-inference-api',
    description='ML Inference API with feature engineering',
    endpointConfiguration={'types': ['REGIONAL']}
)

api_id = api_response['id']
print(f"API Gateway created: {api_id}")

# Get root resource
resources = apigateway_client.get_resources(restApiId=api_id)
root_id = resources['items'][0]['id']

# Create /predict resource
predict_resource = apigateway_client.create_resource(
    restApiId=api_id,
    parentId=root_id,
    pathPart='predict'
)

predict_resource_id = predict_resource['id']

# Create POST method
apigateway_client.put_method(
    restApiId=api_id,
    resourceId=predict_resource_id,
    httpMethod='POST',
    authorizationType='NONE'
)

# Integrate with Lambda
lambda_arn = response['FunctionArn']

apigateway_client.put_integration(
    restApiId=api_id,
    resourceId=predict_resource_id,
    httpMethod='POST',
    type='AWS_PROXY',
    integrationHttpMethod='POST',
    uri=f'arn:aws:apigateway:{region}:lambda:path/2015-03-31/functions/{lambda_arn}/invocations'
)

# Deploy API
deployment = apigateway_client.create_deployment(
    restApiId=api_id,
    stageName='prod'
)

# Get API endpoint
api_endpoint = f'https://{api_id}.execute-api.{region}.amazonaws.com/prod/predict'
print(f"\nAPI Endpoint: {api_endpoint}")

# Add Lambda permission for API Gateway
lambda_client.add_permission(
    FunctionName=function_name,
    StatementId='apigateway-invoke',
    Action='lambda:InvokeFunction',
    Principal='apigateway.amazonaws.com',
    SourceArn=f'arn:aws:execute-api:{region}:{session.account_id()}:{api_id}/*/*'
)
```

#### Step 6: Test End-to-End Inference

```python
import requests

# Test API
test_payload = {
    'tenure_months': 36,
    'monthly_charges': 95.00,
    'contract_type': '2year',
    'support_calls_30d': 1,
    'data_usage_gb': 20.0
}

response = requests.post(api_endpoint, json=test_payload)

if response.status_code == 200:
    result = response.json()
    print("\nInference Result:")
    print(json.dumps(result, indent=2))
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

### Lab Questions

1. What are the benefits of Lambda-based feature engineering?
2. When should feature engineering happen in Lambda vs SageMaker?
3. How do you handle feature engineering latency?

---

## Cleanup Instructions

After completing advanced labs, clean up all resources:

### Delete Endpoints

```python
# Delete all test endpoints
sm_client = boto3.client('sagemaker')

endpoints = [
    'mla-multi-model-endpoint',
    'mla-ab-testing-endpoint',
    'mla-c01-churn-predictor'
]

for endpoint in endpoints:
    try:
        sm_client.delete_endpoint(EndpointName=endpoint)
        print(f"Deleted endpoint: {endpoint}")
    except Exception as e:
        print(f"Could not delete {endpoint}: {e}")
```

### Delete Lambda Functions and API Gateway

```python
# Delete Lambda
lambda_client.delete_function(FunctionName='mla-feature-engineering')

# Delete API Gateway
apigateway_client.delete_rest_api(restApiId=api_id)

# Delete IAM role
iam_client.detach_role_policy(
    RoleName=role_name,
    PolicyArn='arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'
)
iam_client.delete_role(RoleName=role_name)
```

### Delete S3 Data

```bash
aws s3 rm s3://${BUCKET}/mla-c01/ --recursive
```

### Delete EventBridge Rules

```python
events_client = boto3.client('events')
events_client.remove_targets(Rule='mla-daily-retraining-check', Ids=['1'])
events_client.delete_rule(Name='mla-daily-retraining-check')
```

---

## Additional Resources

### Advanced AWS Workshops
- [SageMaker Advanced Workshop](https://catalog.workshops.aws/sagemaker-advanced/en-US)
- [MLOps Workshop](https://catalog.workshops.aws/mlops-workshop/en-US)
- [SageMaker Multi-Model Endpoints](https://aws.amazon.com/blogs/machine-learning/save-costs-by-automatically-scaling-machine-learning-models-with-amazon-sagemaker-endpoints/)

### Documentation
- [SageMaker Distributed Training](https://docs.aws.amazon.com/sagemaker/latest/dg/distributed-training.html)
- [Multi-Model Endpoints](https://docs.aws.amazon.com/sagemaker/latest/dg/multi-model-endpoints.html)
- [SageMaker Pipelines](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html)

### Blog Posts
- [A/B Testing with SageMaker](https://aws.amazon.com/blogs/machine-learning/a-b-testing-ml-models-in-production-using-amazon-sagemaker/)
- [Automated Model Retraining](https://aws.amazon.com/blogs/machine-learning/automate-model-retraining-with-amazon-sagemaker-pipelines/)

---

**Back to Series**: [AWS Machine Learning Engineer Associate (MLA-C01) - Complete Study Guide](/posts/aws-mla-c01-series/)

---

*Questions about advanced labs? Leave a comment below!*
