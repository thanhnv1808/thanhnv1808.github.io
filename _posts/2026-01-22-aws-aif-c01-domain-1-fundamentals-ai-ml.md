---
title: "AWS AIF-C01 - Domain 1: Fundamentals of AI and ML"
author: thanhnv1808
date: 2026-01-22 09:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, machine-learning, artificial-intelligence, deep-learning]
description: Domain 1 covers 20% of the AWS AI Practitioner exam. Learn AI/ML fundamentals, use cases, and the ML development lifecycle.
pin: false
comments: true
---

## Domain Overview

**Domain 1: Fundamentals of AI and ML** represents **20%** of the exam (approximately 10 questions).

This domain covers three main task statements:
1. Explain basic AI concepts and terminologies
2. Identify practical use cases for AI
3. Describe the ML development lifecycle

---

## Task 1.1: Basic AI Concepts and Terminologies

### Understanding the AI Hierarchy

```
Artificial Intelligence (AI)
    └── Machine Learning (ML)
            └── Deep Learning
                    └── Generative AI (GenAI)
```

### Key Definitions

| Term | Definition |
|------|------------|
| **Artificial Intelligence (AI)** | Technology that enables computers to simulate human intelligence and problem-solving |
| **Machine Learning (ML)** | Subset of AI that learns patterns from data without explicit programming |
| **Deep Learning** | Subset of ML using neural networks with multiple layers |
| **Generative AI** | AI that creates new content (text, images, audio, video) |
| **Neural Network** | Computing system inspired by biological neural networks |
| **Algorithm** | Set of rules or instructions for solving problems |
| **Model** | Mathematical representation trained on data to make predictions |

> **Exam Tip**: Know the hierarchy - AI contains ML, ML contains Deep Learning, Deep Learning enables GenAI.
{: .prompt-tip }

### AI vs ML vs Deep Learning vs GenAI

| Aspect | AI | ML | Deep Learning | GenAI |
|--------|----|----|---------------|-------|
| **Scope** | Broadest | Subset of AI | Subset of ML | Uses Deep Learning |
| **Learning** | Rule-based or learned | Learns from data | Multiple neural layers | Creates new content |
| **Data Needs** | Varies | Moderate | Large amounts | Very large amounts |
| **Example** | Chess program | Spam filter | Image recognition | ChatGPT, DALL-E |

### Types of Data in AI/ML

#### By Label Status
- **Labeled Data** - Data with known outcomes (used in supervised learning)
- **Unlabeled Data** - Data without known outcomes (used in unsupervised learning)

#### By Structure
| Type | Description | Example |
|------|-------------|---------|
| **Structured** | Organized in rows/columns | Database tables, spreadsheets |
| **Unstructured** | No predefined format | Images, videos, text documents |
| **Semi-structured** | Partially organized | JSON, XML, emails |

#### By Format
| Type | Description | AWS Services |
|------|-------------|--------------|
| **Tabular** | Rows and columns | SageMaker, Athena |
| **Time-series** | Data points over time | Forecast, CloudWatch |
| **Image** | Visual data | Rekognition, SageMaker |
| **Text** | Natural language | Comprehend, Textract |
| **Audio** | Sound data | Transcribe, Polly |

### Learning Paradigms

#### 1. Supervised Learning
- **Definition**: Model learns from labeled data
- **Use Cases**: Classification, regression, prediction
- **Examples**: Spam detection, price prediction, fraud detection

```
Input: Email with label "spam" or "not spam"
Output: Model that can classify new emails
```

#### 2. Unsupervised Learning
- **Definition**: Model finds patterns in unlabeled data
- **Use Cases**: Clustering, anomaly detection, dimensionality reduction
- **Examples**: Customer segmentation, recommendation systems

```
Input: Customer purchase data (no labels)
Output: Customer groups/segments
```

#### 3. Reinforcement Learning
- **Definition**: Model learns through trial and error with rewards/penalties
- **Use Cases**: Game playing, robotics, autonomous vehicles
- **Examples**: AlphaGo, self-driving cars, recommendation optimization

```
Input: Environment state
Action: Agent takes action
Output: Reward or penalty
```

> **Exam Tip**: Remember - Supervised = labeled data, Unsupervised = unlabeled data, Reinforcement = rewards/penalties.
{: .prompt-tip }

### Inferencing Types

| Type | Description | Use Case | Latency |
|------|-------------|----------|---------|
| **Real-time** | Immediate predictions | Chatbots, fraud detection | Low (ms) |
| **Batch** | Process large datasets | Daily reports, bulk predictions | High (minutes/hours) |

### Key ML Terms

| Term | Definition |
|------|------------|
| **Training** | Process of teaching a model using data |
| **Inferencing** | Using a trained model to make predictions |
| **Bias** | Systematic error in predictions; unfair outcomes |
| **Fairness** | Ensuring model treats all groups equitably |
| **Overfitting** | Model performs well on training data but poorly on new data |
| **Underfitting** | Model performs poorly on both training and new data |
| **LLM** | Large Language Model - AI trained on massive text data |

---

## Task 1.2: Practical Use Cases for AI

### When AI/ML Provides Value

| Value | Description | Example |
|-------|-------------|---------|
| **Decision Support** | Assist humans in making better decisions | Medical diagnosis assistance |
| **Automation** | Automate repetitive tasks | Document processing |
| **Scalability** | Handle large-scale operations | Customer service chatbots |
| **Pattern Recognition** | Find patterns humans might miss | Fraud detection |
| **Personalization** | Customize experiences | Product recommendations |

### When NOT to Use AI/ML

| Scenario | Reason |
|----------|--------|
| **Simple rule-based problems** | Traditional programming is sufficient |
| **Limited data available** | ML needs data to learn |
| **Deterministic outcomes required** | ML provides predictions, not certainties |
| **Cost exceeds benefit** | Implementation cost vs. value gained |
| **Explainability is critical** | Some ML models are "black boxes" |
| **Real-time regulatory compliance** | May need deterministic rules |

> **Exam Tip**: AI/ML is NOT appropriate when you need specific, deterministic outcomes rather than predictions.
{: .prompt-warning }

### ML Techniques by Use Case

#### Regression
- **Purpose**: Predict continuous numerical values
- **Examples**: House prices, temperature, sales forecast
- **Algorithms**: Linear regression, polynomial regression

#### Classification
- **Purpose**: Categorize data into predefined classes
- **Examples**: Spam/not spam, fraud/legitimate, sentiment analysis
- **Algorithms**: Logistic regression, decision trees, SVM

#### Clustering
- **Purpose**: Group similar data points together
- **Examples**: Customer segmentation, document grouping
- **Algorithms**: K-means, hierarchical clustering

### Real-World AI Applications

| Application | Description | AWS Service |
|-------------|-------------|-------------|
| **Computer Vision** | Analyze images and videos | Amazon Rekognition |
| **NLP** | Understand and generate text | Amazon Comprehend |
| **Speech Recognition** | Convert speech to text | Amazon Transcribe |
| **Text-to-Speech** | Convert text to speech | Amazon Polly |
| **Recommendation Systems** | Suggest relevant items | Amazon Personalize |
| **Fraud Detection** | Identify fraudulent activities | Amazon Fraud Detector |
| **Forecasting** | Predict future values | Amazon Forecast |
| **Chatbots** | Conversational interfaces | Amazon Lex |

### AWS AI/ML Services Overview

| Service | Purpose | Use Case |
|---------|---------|----------|
| **Amazon SageMaker** | Complete ML platform | Build, train, deploy ML models |
| **Amazon Transcribe** | Speech-to-text | Transcribe audio/video |
| **Amazon Translate** | Language translation | Translate text between languages |
| **Amazon Comprehend** | NLP analysis | Sentiment analysis, entity extraction |
| **Amazon Lex** | Conversational AI | Build chatbots |
| **Amazon Polly** | Text-to-speech | Generate natural speech |
| **Amazon Rekognition** | Image/video analysis | Face detection, object recognition |
| **Amazon Textract** | Document analysis | Extract text from documents |
| **Amazon Forecast** | Time-series forecasting | Demand forecasting |
| **Amazon Personalize** | Recommendations | Personalized product recommendations |
| **Amazon Fraud Detector** | Fraud detection | Identify fraudulent transactions |
| **Amazon Kendra** | Intelligent search | Enterprise search |

---

## Task 1.3: ML Development Lifecycle

### The ML Pipeline

```
┌──────────────┐    ┌─────────────────┐    ┌────────────────┐
│ Data         │ -> │ Data            │ -> │ Feature        │
│ Collection   │    │ Pre-processing  │    │ Engineering    │
└──────────────┘    └─────────────────┘    └────────────────┘
                                                   │
                                                   v
┌──────────────┐    ┌─────────────────┐    ┌────────────────┐
│ Deployment   │ <- │ Model           │ <- │ Model          │
│ & Monitoring │    │ Evaluation      │    │ Training       │
└──────────────┘    └─────────────────┘    └────────────────┘
```

### Pipeline Stages

#### 1. Data Collection
- Gather relevant data from various sources
- **AWS Tools**: S3, Kinesis, Data Exchange

#### 2. Exploratory Data Analysis (EDA)
- Understand data characteristics
- Identify patterns, outliers, missing values
- **AWS Tools**: SageMaker Data Wrangler, QuickSight

#### 3. Data Pre-processing
- Clean and transform data
- Handle missing values, outliers
- Normalize/standardize features
- **AWS Tools**: SageMaker Data Wrangler, Glue

#### 4. Feature Engineering
- Create new features from existing data
- Select most relevant features
- **AWS Tools**: SageMaker Feature Store

#### 5. Model Training
- Train model on prepared data
- Select appropriate algorithm
- **AWS Tools**: SageMaker Training

#### 6. Hyperparameter Tuning
- Optimize model parameters
- **AWS Tools**: SageMaker Automatic Model Tuning

#### 7. Model Evaluation
- Test model on held-out data
- Calculate performance metrics
- **AWS Tools**: SageMaker Model Evaluation

#### 8. Deployment
- Deploy model to production
- **AWS Tools**: SageMaker Endpoints, Lambda

#### 9. Monitoring
- Track model performance over time
- Detect model drift
- **AWS Tools**: SageMaker Model Monitor

### Sources of ML Models

| Source | Description | Pros | Cons |
|--------|-------------|------|------|
| **Open Source Pre-trained** | Models trained by others | Fast deployment, no training cost | May not fit specific needs |
| **Custom Trained** | Models trained on your data | Tailored to your needs | Requires data and resources |
| **AWS Managed Services** | AWS pre-built AI services | Easy to use, no ML expertise needed | Less customization |

### Model Deployment Options

| Option | Description | Use Case |
|--------|-------------|----------|
| **Managed API** | Use AWS managed service APIs | Quick deployment, no infrastructure |
| **Self-hosted API** | Deploy on your own infrastructure | Full control, custom requirements |
| **SageMaker Endpoints** | Managed inference endpoints | Production ML workloads |
| **Lambda** | Serverless inference | Low-latency, event-driven |

### MLOps Fundamentals

**MLOps** = Machine Learning + DevOps

| Concept | Description |
|---------|-------------|
| **Experimentation** | Track and compare experiments |
| **Reproducibility** | Ensure experiments can be repeated |
| **Automation** | Automate pipeline stages |
| **Monitoring** | Track model performance |
| **Model Registry** | Version and manage models |
| **CI/CD for ML** | Continuous integration/deployment |

### Model Performance Metrics

#### Classification Metrics

| Metric | Description | When to Use |
|--------|-------------|-------------|
| **Accuracy** | Correct predictions / Total predictions | Balanced classes |
| **Precision** | True positives / Predicted positives | Cost of false positives is high |
| **Recall** | True positives / Actual positives | Cost of false negatives is high |
| **F1 Score** | Harmonic mean of precision and recall | Balance precision and recall |
| **AUC-ROC** | Area under ROC curve | Compare model performance |

#### Regression Metrics

| Metric | Description |
|--------|-------------|
| **MAE** | Mean Absolute Error |
| **MSE** | Mean Squared Error |
| **RMSE** | Root Mean Squared Error |
| **R²** | Coefficient of determination |

### Business Metrics

| Metric | Description |
|--------|-------------|
| **ROI** | Return on investment |
| **Cost per User** | Operational cost divided by users |
| **Development Cost** | Total cost to build the solution |
| **Customer Feedback** | User satisfaction scores |
| **Time to Value** | Time from development to business impact |

> **Exam Tip**: Know the difference between model metrics (accuracy, F1) and business metrics (ROI, cost).
{: .prompt-tip }

---

## Practice Questions

### Question 1
A company wants to predict customer churn based on historical data with known outcomes. Which type of learning should they use?

A. Unsupervised learning
B. Supervised learning
C. Reinforcement learning
D. Transfer learning

<details>
<summary>Show Answer</summary>
<b>B. Supervised learning</b>

The data has known outcomes (churned/not churned), making this a supervised learning problem.
</details>

### Question 2
Which AWS service would you use to extract text from scanned documents?

A. Amazon Comprehend
B. Amazon Textract
C. Amazon Transcribe
D. Amazon Translate

<details>
<summary>Show Answer</summary>
<b>B. Amazon Textract</b>

Amazon Textract is specifically designed to extract text and data from scanned documents.
</details>

### Question 3
A retail company wants to group customers with similar purchasing behavior without predefined categories. Which ML technique is most appropriate?

A. Classification
B. Regression
C. Clustering
D. Time-series forecasting

<details>
<summary>Show Answer</summary>
<b>C. Clustering</b>

Clustering groups similar data points without predefined labels, perfect for customer segmentation.
</details>

### Question 4
Which metric is most important when the cost of false negatives is very high (e.g., cancer detection)?

A. Precision
B. Accuracy
C. Recall
D. F1 Score

<details>
<summary>Show Answer</summary>
<b>C. Recall</b>

Recall measures the ability to find all positive cases. When missing a positive case (false negative) is costly, recall is the priority.
</details>

### Question 5
What is the main difference between batch and real-time inferencing?

A. Batch uses more data than real-time
B. Real-time processes data immediately while batch processes data in bulk
C. Batch is more accurate than real-time
D. Real-time requires less computational resources

<details>
<summary>Show Answer</summary>
<b>B. Real-time processes data immediately while batch processes data in bulk</b>

Real-time inferencing provides immediate predictions, while batch inferencing processes large amounts of data at scheduled intervals.
</details>

---

## Key Takeaways

1. **AI Hierarchy**: AI > ML > Deep Learning > GenAI
2. **Learning Types**: Supervised (labeled), Unsupervised (unlabeled), Reinforcement (rewards)
3. **ML Techniques**: Regression (continuous), Classification (categories), Clustering (groups)
4. **ML Pipeline**: Data → Preprocess → Feature Engineering → Train → Evaluate → Deploy → Monitor
5. **AWS Services**: Know which service to use for each use case (Comprehend for NLP, Rekognition for images, etc.)
6. **Metrics**: Model metrics (accuracy, F1) vs. Business metrics (ROI, cost)

---

**Next Lesson**: [Domain 2: Fundamentals of Generative AI](/posts/aws-aif-c01-domain-2-fundamentals-genai/)

---

*Questions about AI/ML fundamentals? Leave a comment below!*
