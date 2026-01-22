---
title: "AWS AIF-C01 - Hands-On Labs: Practice with AWS AI Services"
author: thanhnv1808
date: 2026-01-22 14:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, hands-on, labs, bedrock, sagemaker, comprehend]
description: Hands-on labs to practice AWS AI services for the AIF-C01 exam. Learn by doing with Amazon Bedrock, SageMaker, Comprehend, and more.
pin: false
comments: true
---

## Introduction

This post provides hands-on labs to practice with AWS AI services covered in the AIF-C01 exam. Learning by doing is the best way to understand these services.

> **Cost Warning**: Some labs may incur AWS charges. Use AWS Free Tier when available and clean up resources after labs.
{: .prompt-warning }

### Prerequisites

- AWS Account (Free Tier eligible)
- Basic understanding of AWS Console
- Completed the theory lessons in this series

### AWS Free Tier for AI Services

| Service | Free Tier |
|---------|-----------|
| Amazon Bedrock | Pay per use (no free tier) |
| Amazon Comprehend | 50K units/month for 12 months |
| Amazon Transcribe | 60 minutes/month for 12 months |
| Amazon Translate | 2M characters/month for 12 months |
| Amazon Polly | 5M characters/month for 12 months |
| Amazon Rekognition | 5K images/month for 12 months |
| Amazon SageMaker | 250 hours t3.medium for 2 months |

---

## Lab 1: Getting Started with Amazon Bedrock

### Objective
Learn to access and use Foundation Models through Amazon Bedrock.

### Duration
30 minutes

### Steps

#### Step 1: Enable Amazon Bedrock Model Access

1. Open **AWS Console** → Search for **Bedrock**
2. In the left menu, click **Model access**
3. Click **Manage model access**
4. Select the models you want to enable:
   - **Amazon Titan** (Text and Embeddings)
   - **Claude** (Anthropic)
   - **Llama** (Meta)
5. Click **Request model access**
6. Wait for approval (usually instant for most models)

> **Note**: Some models require additional approval and may take longer.
{: .prompt-info }

#### Step 2: Test a Model in the Playground

1. In Bedrock console, click **Playgrounds** → **Chat**
2. Select a model (e.g., **Claude 3 Sonnet**)
3. Try these prompts:

**Prompt 1: Simple Question**
```
What is machine learning? Explain in 3 sentences.
```

**Prompt 2: Code Generation**
```
Write a Python function that checks if a number is prime.
```

**Prompt 3: Summarization**
```
Summarize the following text in bullet points:

Amazon Web Services (AWS) is a comprehensive cloud computing platform
provided by Amazon. It offers over 200 fully featured services from
data centers globally. AWS provides infrastructure technologies like
compute, storage, and databases, as well as emerging technologies
such as machine learning, artificial intelligence, data lakes,
analytics, and Internet of Things.
```

#### Step 3: Experiment with Inference Parameters

1. Click **Configurations** (gear icon)
2. Adjust **Temperature**:
   - Set to **0.0** → Run the same prompt 3 times (notice consistent output)
   - Set to **1.0** → Run the same prompt 3 times (notice varied output)
3. Adjust **Max tokens** to control output length
4. Try **Top P** parameter

#### Step 4: Compare Models

1. Run the same prompt on different models:
   - Claude 3 Sonnet
   - Amazon Titan Text
   - Llama 3
2. Compare:
   - Response quality
   - Response time
   - Output style

### Lab Questions

1. Which model provided the most detailed response?
2. How did temperature affect the output?
3. What differences did you notice between models?

---

## Lab 2: Building a RAG Application with Bedrock Knowledge Bases

### Objective
Create a knowledge base and use RAG to answer questions from your documents.

### Duration
45 minutes

### Prerequisites
- S3 bucket with documents (PDF, TXT, or MD files)

### Steps

#### Step 1: Prepare Your Data

1. Go to **S3** console
2. Create a new bucket: `aif-c01-lab-kb-{your-name}`
3. Create a folder: `documents/`
4. Upload sample documents (use AWS whitepapers or your own docs)

**Sample content to upload** (save as `aws-ai-services.txt`):
```
AWS AI Services Overview

Amazon Bedrock is a fully managed service that makes foundation models
available through an API. It supports models from AI21 Labs, Anthropic,
Cohere, Meta, Stability AI, and Amazon.

Amazon SageMaker is a fully managed machine learning service that enables
data scientists and developers to build, train, and deploy ML models quickly.

Amazon Comprehend is a natural language processing (NLP) service that uses
machine learning to find insights and relationships in text.

Amazon Rekognition is a service that makes it easy to add image and video
analysis to your applications using deep learning technology.
```

#### Step 2: Create a Knowledge Base

1. Go to **Bedrock** console → **Knowledge bases**
2. Click **Create knowledge base**
3. Configure:
   - **Name**: `aif-c01-lab-kb`
   - **Description**: `Lab knowledge base for AWS AI services`
   - **IAM role**: Create new role
4. Click **Next**

#### Step 3: Configure Data Source

1. **Data source name**: `s3-documents`
2. **S3 URI**: Browse to your bucket/documents folder
3. **Chunking strategy**: Default (Fixed size)
4. Click **Next**

#### Step 4: Select Embeddings Model

1. **Embeddings model**: Amazon Titan Embeddings G1 - Text
2. **Vector database**: Quick create a new vector store
3. Click **Next** → **Create knowledge base**
4. Wait for creation and sync to complete

#### Step 5: Test the Knowledge Base

1. Select your knowledge base
2. Click **Test knowledge base**
3. Select a model (Claude 3 Sonnet)
4. Ask questions:

```
What is Amazon Bedrock?
```

```
Compare Amazon SageMaker and Amazon Bedrock.
```

```
Which AWS service should I use for image analysis?
```

5. Notice the **Source chunks** showing where information came from

### Lab Questions

1. How does RAG help reduce hallucinations?
2. What happens if you ask about something not in your documents?
3. How are the source citations helpful?

---

## Lab 3: Amazon Bedrock Guardrails

### Objective
Create guardrails to filter content and protect your AI application.

### Duration
30 minutes

### Steps

#### Step 1: Create a Guardrail

1. Go to **Bedrock** console → **Guardrails**
2. Click **Create guardrail**
3. **Name**: `aif-c01-content-filter`
4. **Description**: `Lab guardrail for content filtering`

#### Step 2: Configure Content Filters

1. **Content filters**:
   - Enable **Hate** filter → Set to **High**
   - Enable **Insults** filter → Set to **High**
   - Enable **Sexual** filter → Set to **High**
   - Enable **Violence** filter → Set to **High**
2. Apply to both **User prompts** and **Model responses**

#### Step 3: Configure Denied Topics

1. Click **Add denied topic**
2. **Name**: `competitor-info`
3. **Definition**: `Questions about competitor products or services`
4. **Sample phrases**:
   - "Tell me about Google Cloud"
   - "Compare AWS to Azure"
   - "What are the alternatives to AWS"

#### Step 4: Configure Word Filters

1. **Managed word filters**: Enable profanity filter
2. **Custom words**: Add any words you want to block

#### Step 5: Configure PII Filters

1. Enable **PII** detection
2. Select PII types to detect:
   - Email addresses
   - Phone numbers
   - Credit card numbers
3. Set action to **Block** or **Anonymize**

#### Step 6: Test the Guardrail

1. Go to **Playgrounds** → **Chat**
2. Select your guardrail in configurations
3. Test with prompts:

**Should be blocked:**
```
Tell me about Google Cloud Platform services.
```

**Should trigger PII filter:**
```
My email is test@example.com and my phone is 123-456-7890.
```

**Normal prompt (should work):**
```
What are the benefits of using Amazon Bedrock?
```

### Lab Questions

1. What message does the user see when content is blocked?
2. How do denied topics differ from content filters?
3. Why is PII filtering important for AI applications?

---

## Lab 4: Amazon Comprehend for NLP

### Objective
Use Amazon Comprehend to analyze text for sentiment, entities, and key phrases.

### Duration
30 minutes

### Steps

#### Step 1: Access Amazon Comprehend

1. Open **AWS Console** → Search for **Comprehend**
2. Click **Launch Amazon Comprehend**

#### Step 2: Sentiment Analysis

1. Click **Analyze** in the left menu
2. Select **Built-in** → **Sentiment**
3. Enter text to analyze:

**Positive sentiment:**
```
I absolutely love this product! It exceeded all my expectations
and the customer service was fantastic. Highly recommended!
```

**Negative sentiment:**
```
This was a terrible experience. The product arrived damaged
and customer support was unhelpful. I want a refund immediately.
```

**Mixed sentiment:**
```
The product quality is excellent, but the shipping took way too long.
I'm happy with the item but frustrated with the delivery process.
```

4. Click **Analyze**
5. Review the sentiment scores (Positive, Negative, Neutral, Mixed)

#### Step 3: Entity Recognition

1. Select **Entities** analysis
2. Enter text:

```
Amazon Web Services was founded in 2006 by Andy Jassy in Seattle, Washington.
The company provides cloud computing services to millions of customers worldwide.
In 2023, AWS generated over $90 billion in revenue.
```

3. Click **Analyze**
4. Review detected entities:
   - ORGANIZATION: Amazon Web Services
   - PERSON: Andy Jassy
   - LOCATION: Seattle, Washington
   - DATE: 2006, 2023
   - QUANTITY: $90 billion

#### Step 4: Key Phrases Extraction

1. Select **Key phrases** analysis
2. Use the same text from Step 3
3. Click **Analyze**
4. Review extracted key phrases

#### Step 5: Language Detection

1. Select **Language** analysis
2. Try texts in different languages:

**English:**
```
Hello, how are you today?
```

**Vietnamese:**
```
Xin chào, hôm nay bạn thế nào?
```

**Japanese:**
```
こんにちは、今日はお元気ですか？
```

### Lab Questions

1. What confidence score did you get for sentiment analysis?
2. What types of entities can Comprehend detect?
3. How accurate was the language detection?

---

## Lab 5: Amazon Transcribe and Amazon Polly

### Objective
Convert speech to text and text to speech using AWS AI services.

### Duration
30 minutes

### Steps

#### Part A: Amazon Polly (Text-to-Speech)

1. Open **AWS Console** → Search for **Polly**
2. Click **Try Polly**
3. Select:
   - **Engine**: Neural
   - **Language**: English (US)
   - **Voice**: Joanna or Matthew
4. Enter text:

```
Welcome to the AWS AI Practitioner certification preparation course.
Today we will learn about Amazon Bedrock and other AI services.
```

5. Click **Listen** to hear the audio
6. Try different voices and languages
7. Click **Download** to save the MP3 file

**Experiment with SSML:**
```xml
<speak>
    Welcome to <emphasis level="strong">AWS</emphasis>.
    <break time="1s"/>
    Let's learn about <prosody rate="slow">machine learning</prosody>.
</speak>
```

#### Part B: Amazon Transcribe (Speech-to-Text)

1. Open **AWS Console** → Search for **Transcribe**
2. Click **Real-time transcription**
3. Select:
   - **Language**: English (US)
   - **Input type**: Microphone
4. Click **Start streaming**
5. Speak into your microphone
6. Watch the real-time transcription
7. Click **Stop streaming**

**Alternative: Transcribe from file**
1. Go to **Transcription jobs**
2. Click **Create job**
3. Upload an audio file from S3
4. Wait for transcription to complete
5. Review the output

### Lab Questions

1. Which Polly voice sounded most natural?
2. How accurate was the transcription?
3. What are some use cases for these services?

---

## Lab 6: Amazon Rekognition for Image Analysis

### Objective
Use Amazon Rekognition to analyze images for objects, faces, and text.

### Duration
30 minutes

### Steps

#### Step 1: Access Rekognition Console

1. Open **AWS Console** → Search for **Rekognition**
2. Click **Try demo** or use the console

#### Step 2: Object and Scene Detection

1. Click **Object and scene detection**
2. Upload an image or use a sample
3. Review detected labels with confidence scores
4. Try different images:
   - Outdoor scene
   - Office environment
   - Food photo

#### Step 3: Facial Analysis

1. Click **Facial analysis**
2. Upload a photo with faces
3. Review detected attributes:
   - Age range
   - Gender
   - Emotions
   - Facial features (glasses, beard, etc.)
   - Face quality (brightness, sharpness)

> **Privacy Note**: Be mindful of privacy when using facial analysis.
{: .prompt-warning }

#### Step 4: Text Detection (OCR)

1. Click **Text detection**
2. Upload an image with text:
   - Sign
   - Document
   - Screenshot
3. Review detected text and confidence scores

#### Step 5: Celebrity Recognition

1. Click **Celebrity recognition**
2. Upload an image with a well-known person
3. Review the recognition results

#### Step 6: Using AWS CLI

```bash
# Install AWS CLI and configure credentials first

# Detect labels in an image
aws rekognition detect-labels \
    --image '{"S3Object":{"Bucket":"your-bucket","Name":"your-image.jpg"}}' \
    --region us-east-1

# Detect faces
aws rekognition detect-faces \
    --image '{"S3Object":{"Bucket":"your-bucket","Name":"face-photo.jpg"}}' \
    --attributes ALL \
    --region us-east-1

# Detect text
aws rekognition detect-text \
    --image '{"S3Object":{"Bucket":"your-bucket","Name":"text-image.jpg"}}' \
    --region us-east-1
```

### Lab Questions

1. What confidence scores did you get for object detection?
2. How accurate was the facial analysis?
3. What types of text can Rekognition detect?

---

## Lab 7: Amazon SageMaker Clarify for Bias Detection

### Objective
Understand how to detect bias in ML models using SageMaker Clarify.

### Duration
45 minutes

### Steps

#### Step 1: Access SageMaker Studio

1. Open **AWS Console** → Search for **SageMaker**
2. Click **Studio** → **Open Studio**
3. Wait for Studio to load

#### Step 2: Create a Sample Notebook

1. Click **File** → **New** → **Notebook**
2. Select **Python 3 (Data Science)** kernel

#### Step 3: Load Sample Data

```python
# Import libraries
import pandas as pd
import numpy as np
from sagemaker import clarify

# Create sample dataset (loan approval)
np.random.seed(42)
n_samples = 1000

data = pd.DataFrame({
    'age': np.random.randint(18, 70, n_samples),
    'income': np.random.randint(20000, 150000, n_samples),
    'gender': np.random.choice(['male', 'female'], n_samples),
    'education': np.random.choice(['high_school', 'bachelor', 'master'], n_samples),
    'loan_approved': np.random.choice([0, 1], n_samples)
})

# Introduce bias: males more likely to be approved
data.loc[(data['gender'] == 'male'), 'loan_approved'] = np.random.choice(
    [0, 1],
    sum(data['gender'] == 'male'),
    p=[0.3, 0.7]
)
data.loc[(data['gender'] == 'female'), 'loan_approved'] = np.random.choice(
    [0, 1],
    sum(data['gender'] == 'female'),
    p=[0.6, 0.4]
)

print(data.head())
print("\nApproval rate by gender:")
print(data.groupby('gender')['loan_approved'].mean())
```

#### Step 4: Analyze Bias

```python
# Check class imbalance
print("Class Imbalance (CI):")
print(f"Male approval rate: {data[data['gender']=='male']['loan_approved'].mean():.2%}")
print(f"Female approval rate: {data[data['gender']=='female']['loan_approved'].mean():.2%}")

# Calculate Difference in Positive Proportions (DPP)
male_positive = data[data['gender']=='male']['loan_approved'].mean()
female_positive = data[data['gender']=='female']['loan_approved'].mean()
dpp = male_positive - female_positive
print(f"\nDifference in Positive Proportions: {dpp:.2%}")

# DPP > 0.1 or < -0.1 often indicates significant bias
if abs(dpp) > 0.1:
    print("⚠️ Significant bias detected!")
else:
    print("✓ No significant bias detected")
```

#### Step 5: Understand Clarify Metrics

| Metric | Description | Ideal Value |
|--------|-------------|-------------|
| **Class Imbalance (CI)** | Difference in class sizes | Close to 0 |
| **DPL** | Difference in Positive Label | Close to 0 |
| **KL Divergence** | Distribution difference | Close to 0 |
| **DPPL** | Difference in Predicted Positive Labels | Close to 0 |

### Lab Questions

1. What bias did you detect in the sample data?
2. Why is it important to check for bias before deployment?
3. What actions would you take if bias is detected?

---

## Lab 8: Prompt Engineering Practice

### Objective
Practice different prompt engineering techniques with Amazon Bedrock.

### Duration
30 minutes

### Steps

#### Step 1: Open Bedrock Playground

1. Go to **Bedrock** → **Playgrounds** → **Chat**
2. Select **Claude 3 Sonnet**

#### Step 2: Zero-Shot Prompting

```
Classify the sentiment of this review as positive, negative, or neutral:

"The product arrived on time and works as expected."
```

#### Step 3: Few-Shot Prompting

```
Classify the sentiment of customer reviews.

Review: "Amazing product! Best purchase ever!"
Sentiment: Positive

Review: "Terrible quality. Broke after one day."
Sentiment: Negative

Review: "It's okay, nothing special."
Sentiment: Neutral

Review: "Fast shipping but the color was different than shown."
Sentiment:
```

#### Step 4: Chain-of-Thought Prompting

```
Solve this problem step by step:

A store has 150 apples. They sell 40% in the morning and 30 more in the afternoon.
How many apples are left?

Let's think step by step:
```

#### Step 5: Structured Output

```
Extract information from this text and return as JSON:

Text: "John Smith is a 35-year-old software engineer at Amazon in Seattle.
His email is john.smith@email.com and his phone is 555-0123."

Return JSON with fields: name, age, job, company, location, email, phone
```

#### Step 6: Role-Based Prompting

```
You are an expert AWS Solutions Architect with 10 years of experience.

A customer asks: "Should I use Lambda or EC2 for my web application
that gets 1000 requests per minute?"

Provide your professional recommendation with pros and cons.
```

#### Step 7: Negative Prompting

```
Write a product description for a laptop.

Requirements:
- Be concise (under 100 words)
- Focus on benefits, not features
- Use professional tone

Do NOT:
- Use superlatives like "best" or "amazing"
- Mention prices
- Use technical jargon
```

### Lab Questions

1. Which prompting technique gave the best results?
2. How did few-shot improve over zero-shot?
3. Why is Chain-of-Thought helpful for reasoning tasks?

---

## Lab 9: PartyRock - No-Code GenAI

### Objective
Build a GenAI application without code using PartyRock.

### Duration
20 minutes

### Steps

#### Step 1: Access PartyRock

1. Go to [partyrock.aws](https://partyrock.aws)
2. Sign in with your Amazon account (not AWS account)

#### Step 2: Create an App

1. Click **Build your own app**
2. Describe your app:
   ```
   Create an app that generates study flashcards for AWS certifications.
   User enters a topic, and the app generates 5 Q&A flashcards.
   ```
3. Click **Generate app**

#### Step 3: Customize the App

1. Review the generated widgets
2. Modify the prompt in the text generation widget
3. Add/remove widgets as needed
4. Test the app with different inputs

#### Step 4: Share Your App

1. Click **Make public** to share
2. Copy the shareable link
3. Test in a new browser window

### Lab Ideas

- **Study Buddy**: Generate practice questions
- **Code Explainer**: Explain code snippets
- **Email Writer**: Draft professional emails
- **Recipe Generator**: Create recipes from ingredients

### Lab Questions

1. How easy was it to create an app without code?
2. What limitations did you notice?
3. When would you use PartyRock vs. Bedrock APIs?

---

## Lab 10: Amazon Q Developer

### Objective
Use Amazon Q Developer for code assistance.

### Duration
20 minutes

### Steps

#### Step 1: Install Amazon Q in IDE

**For VS Code:**
1. Open VS Code
2. Go to Extensions
3. Search for "Amazon Q"
4. Install the extension
5. Sign in with AWS Builder ID

#### Step 2: Code Generation

1. Create a new Python file
2. Type a comment and press Tab:

```python
# Function to calculate fibonacci sequence up to n terms
```

3. Accept or modify the suggestion

#### Step 3: Code Explanation

1. Select a block of code
2. Right-click → **Amazon Q** → **Explain**
3. Read the explanation

#### Step 4: Code Transformation

1. Select code
2. Ask Q to refactor or optimize:
   - "Make this function more efficient"
   - "Add error handling"
   - "Convert to async function"

#### Step 5: Security Scanning

1. Write code with a potential vulnerability:

```python
import sqlite3

def get_user(user_id):
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    # SQL injection vulnerability
    cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
    return cursor.fetchone()
```

2. Amazon Q should flag the SQL injection risk
3. Ask Q to fix the security issue

### Lab Questions

1. How accurate were the code suggestions?
2. Did Q identify security vulnerabilities?
3. How does Q compare to other AI coding assistants?

---

## Cleanup Instructions

After completing labs, clean up resources to avoid charges:

### Amazon Bedrock
- Knowledge bases: Delete knowledge bases and associated vector stores
- Guardrails: Delete guardrails if not needed

### Amazon S3
- Delete buckets created for labs

### Amazon SageMaker
- Stop and delete notebook instances
- Delete endpoints if created

### General
- Check **AWS Cost Explorer** for any unexpected charges
- Set up **Billing Alerts** to monitor costs

```bash
# AWS CLI commands to help with cleanup
aws s3 rb s3://aif-c01-lab-kb-{your-name} --force
```

---

## Additional Resources

### AWS Workshops
- [Amazon Bedrock Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/a4bdb007-5600-4368-81c5-ff5b4154f518/en-US)
- [Amazon SageMaker Workshop](https://catalog.workshops.aws/sagemaker-intro/en-US)
- [Generative AI on AWS Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/80ae1ed2-f415-4d3d-9eb0-e9118c147bd4/en-US)

### AWS Skill Builder (Free Courses)
- Amazon Bedrock Getting Started
- Generative AI Foundations on AWS
- AWS AI Practitioner Essentials

### Documentation
- [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/)
- [Amazon SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/)
- [Amazon Comprehend Developer Guide](https://docs.aws.amazon.com/comprehend/)

---

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)

---

*Have questions about the labs? Leave a comment below!*
