---
title: "AWS AIF-C01 - Full Practice Exam (65 Questions)"
author: thanhnv1808
date: 2026-01-22 16:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, practice-exam, mock-test, certification]
description: Full-length practice exam for AWS AI Practitioner (AIF-C01) with 65 questions covering all domains. Test your exam readiness!
pin: false
comments: true
---

## Practice Exam Instructions

This practice exam simulates the real AWS Certified AI Practitioner (AIF-C01) exam.

| Exam Detail | Information |
|-------------|-------------|
| **Total Questions** | 65 |
| **Time Limit** | 90 minutes |
| **Passing Score** | 700/1000 |
| **Question Types** | Multiple choice, Multiple response |

### Domain Distribution

| Domain | Questions |
|--------|-----------|
| Domain 1: Fundamentals of AI and ML | 13 (20%) |
| Domain 2: Fundamentals of GenAI | 16 (24%) |
| Domain 3: Applications of Foundation Models | 18 (28%) |
| Domain 4: Guidelines for Responsible AI | 9 (14%) |
| Domain 5: Security, Compliance, Governance | 9 (14%) |

> **Tip**: Time yourself! Aim for about 1.5 minutes per question.
{: .prompt-tip }

---

## Domain 1: Fundamentals of AI and ML (Questions 1-13)

### Question 1
What is the correct hierarchy of AI technologies from broadest to most specific?

A. Machine Learning → Deep Learning → Artificial Intelligence → Generative AI
B. Artificial Intelligence → Machine Learning → Deep Learning → Generative AI
C. Deep Learning → Machine Learning → Artificial Intelligence → Generative AI
D. Generative AI → Deep Learning → Machine Learning → Artificial Intelligence

<details>
<summary>Show Answer</summary>
<b>B. Artificial Intelligence → Machine Learning → Deep Learning → Generative AI</b>

AI is the broadest field, ML is a subset of AI, Deep Learning is a subset of ML, and GenAI uses Deep Learning techniques.
</details>

---

### Question 2
A company wants to predict house prices based on historical sales data with known prices. Which type of machine learning is most appropriate?

A. Unsupervised learning
B. Supervised learning
C. Reinforcement learning
D. Transfer learning

<details>
<summary>Show Answer</summary>
<b>B. Supervised learning</b>

Predicting prices using labeled historical data (houses with known prices) is a supervised learning regression problem.
</details>

---

### Question 3
Which AWS service would you use to build a conversational chatbot for customer service?

A. Amazon Comprehend
B. Amazon Lex
C. Amazon Polly
D. Amazon Transcribe

<details>
<summary>Show Answer</summary>
<b>B. Amazon Lex</b>

Amazon Lex is specifically designed for building conversational interfaces (chatbots) with natural language understanding.
</details>

---

### Question 4
A retail company wants to group customers with similar purchasing behaviors without predefined categories. Which ML technique should they use?

A. Classification
B. Regression
C. Clustering
D. Reinforcement learning

<details>
<summary>Show Answer</summary>
<b>C. Clustering</b>

Clustering is an unsupervised learning technique that groups similar data points without predefined labels.
</details>

---

### Question 5
Which type of data would be considered UNSTRUCTURED?

A. Database tables
B. CSV files
C. Video recordings
D. Excel spreadsheets

<details>
<summary>Show Answer</summary>
<b>C. Video recordings</b>

Video recordings are unstructured data. Database tables, CSV files, and spreadsheets are structured data with defined schemas.
</details>

---

### Question 6
What is the main difference between batch inferencing and real-time inferencing?

A. Batch is more accurate than real-time
B. Real-time processes data immediately while batch processes data in bulk at scheduled times
C. Batch uses more computing resources than real-time
D. Real-time is only for image processing

<details>
<summary>Show Answer</summary>
<b>B. Real-time processes data immediately while batch processes data in bulk at scheduled times</b>

Real-time inference provides immediate predictions; batch inference processes large amounts of data at scheduled intervals.
</details>

---

### Question 7
Which metric is most appropriate when the cost of missing positive cases (false negatives) is very high, such as in disease detection?

A. Precision
B. Accuracy
C. Recall
D. F1 Score

<details>
<summary>Show Answer</summary>
<b>C. Recall</b>

Recall (sensitivity) measures the ability to find all positive cases. When false negatives are costly, maximize recall.
</details>

---

### Question 8
A model performs excellently on training data but poorly on new, unseen data. What is this problem called?

A. Underfitting
B. Overfitting
C. Bias
D. Class imbalance

<details>
<summary>Show Answer</summary>
<b>B. Overfitting</b>

Overfitting (high variance) occurs when a model learns the training data too well and fails to generalize to new data.
</details>

---

### Question 9
Which AWS service would you use to extract text from scanned documents and forms?

A. Amazon Comprehend
B. Amazon Textract
C. Amazon Rekognition
D. Amazon Translate

<details>
<summary>Show Answer</summary>
<b>B. Amazon Textract</b>

Amazon Textract is designed to extract text, forms, and tables from scanned documents.
</details>

---

### Question 10
What is the purpose of feature engineering in the ML pipeline?

A. To deploy models to production
B. To create new input variables from existing data to improve model performance
C. To evaluate model accuracy
D. To collect training data

<details>
<summary>Show Answer</summary>
<b>B. To create new input variables from existing data to improve model performance</b>

Feature engineering transforms raw data into features that better represent the underlying problem to improve model performance.
</details>

---

### Question 11
Which AWS service provides a complete platform for building, training, and deploying custom ML models?

A. Amazon Bedrock
B. Amazon SageMaker
C. Amazon Comprehend
D. Amazon Lex

<details>
<summary>Show Answer</summary>
<b>B. Amazon SageMaker</b>

SageMaker is a complete ML platform for the entire ML lifecycle from data preparation to deployment.
</details>

---

### Question 12
A company wants to automatically translate customer reviews from multiple languages to English. Which service should they use?

A. Amazon Comprehend
B. Amazon Transcribe
C. Amazon Translate
D. Amazon Polly

<details>
<summary>Show Answer</summary>
<b>C. Amazon Translate</b>

Amazon Translate is a neural machine translation service for converting text between languages.
</details>

---

### Question 13
In which scenario would ML NOT be appropriate? (Select the best answer)

A. Predicting customer churn based on behavior patterns
B. Determining if a transaction follows specific regulatory rules
C. Recommending products based on purchase history
D. Detecting fraudulent transactions

<details>
<summary>Show Answer</summary>
<b>B. Determining if a transaction follows specific regulatory rules</b>

Rule-based compliance checking requires deterministic outcomes, not predictions. Traditional programming with explicit rules is more appropriate.
</details>

---

## Domain 2: Fundamentals of Generative AI (Questions 14-29)

### Question 14
What is a Foundation Model (FM)?

A. A small model trained for a specific task
B. A large pre-trained model that can be adapted for various tasks
C. A model used only for classification
D. A model that requires no training data

<details>
<summary>Show Answer</summary>
<b>B. A large pre-trained model that can be adapted for various tasks</b>

Foundation Models are large models trained on vast datasets that can be fine-tuned or prompted for many different tasks.
</details>

---

### Question 15
Which of the following is a key characteristic of Large Language Models (LLMs)?

A. They can only process images
B. They are trained on small, curated datasets
C. They are transformer-based models trained on massive text data
D. They cannot generate new content

<details>
<summary>Show Answer</summary>
<b>C. They are transformer-based models trained on massive text data</b>

LLMs are typically transformer-based architectures trained on enormous amounts of text data.
</details>

---

### Question 16
What is the primary purpose of tokenization in LLMs?

A. To encrypt the input text for security
B. To break text into smaller units that the model can process
C. To translate text to another language
D. To compress the model size

<details>
<summary>Show Answer</summary>
<b>B. To break text into smaller units that the model can process</b>

Tokenization converts text into tokens (words, subwords, or characters) that the model can process mathematically.
</details>

---

### Question 17
What are embeddings in the context of GenAI?

A. Images embedded in documents
B. Numerical vector representations that capture semantic meaning
C. Encrypted versions of prompts
D. Physical storage locations for models

<details>
<summary>Show Answer</summary>
<b>B. Numerical vector representations that capture semantic meaning</b>

Embeddings convert text/data into numerical vectors where similar meanings are close together in vector space.
</details>

---

### Question 18
A GenAI model confidently provides factually incorrect information. What is this phenomenon called?

A. Bias
B. Hallucination
C. Overfitting
D. Underfitting

<details>
<summary>Show Answer</summary>
<b>B. Hallucination</b>

Hallucinations occur when GenAI models generate plausible-sounding but factually incorrect or fabricated information.
</details>

---

### Question 19
Which AWS service is the primary managed service for accessing Foundation Models like Claude and Titan?

A. Amazon SageMaker
B. Amazon Bedrock
C. Amazon Comprehend
D. Amazon Q

<details>
<summary>Show Answer</summary>
<b>B. Amazon Bedrock</b>

Amazon Bedrock is the managed service that provides API access to multiple Foundation Models.
</details>

---

### Question 20
What type of model would you use to generate images from text descriptions?

A. Large Language Model
B. Transformer Model
C. Diffusion Model
D. Recurrent Neural Network

<details>
<summary>Show Answer</summary>
<b>C. Diffusion Model</b>

Diffusion models (like Stable Diffusion, DALL-E) are specifically designed for image generation from text prompts.
</details>

---

### Question 21
Which is NOT a typical use case for Generative AI?

A. Text summarization
B. Code generation
C. Sorting a list of numbers
D. Image creation

<details>
<summary>Show Answer</summary>
<b>C. Sorting a list of numbers</b>

Sorting is a deterministic algorithm task. GenAI is for creating new content, not deterministic computations.
</details>

---

### Question 22
What is Amazon Q Developer primarily used for?

A. Business analytics
B. Code assistance and generation
C. Image recognition
D. Language translation

<details>
<summary>Show Answer</summary>
<b>B. Code assistance and generation</b>

Amazon Q Developer is an AI coding assistant for code generation, completion, explanation, and security scanning.
</details>

---

### Question 23
What is PartyRock?

A. A high-performance computing service
B. A no-code GenAI application builder
C. A database service
D. A video streaming service

<details>
<summary>Show Answer</summary>
<b>B. A no-code GenAI application builder</b>

PartyRock is AWS's free, no-code platform for building GenAI applications for learning and prototyping.
</details>

---

### Question 24
Which pricing model would be most cost-effective for a company with consistent, predictable GenAI workloads?

A. On-demand pricing
B. Provisioned throughput
C. Spot instances
D. Free tier

<details>
<summary>Show Answer</summary>
<b>B. Provisioned throughput</b>

Provisioned throughput offers reserved capacity at lower per-token costs for consistent workloads.
</details>

---

### Question 25
What is a key advantage of using Amazon Bedrock?

A. It requires you to train models from scratch
B. It provides serverless access to multiple Foundation Models
C. It only supports Amazon's own models
D. It requires managing GPU infrastructure

<details>
<summary>Show Answer</summary>
<b>B. It provides serverless access to multiple Foundation Models</b>

Bedrock offers serverless access to multiple FMs from different providers without infrastructure management.
</details>

---

### Question 26
Which of the following is a disadvantage of Generative AI? (Select TWO)

A. Can generate creative content
B. May produce hallucinations
C. Provides fast responses
D. Outputs can be nondeterministic
E. Can understand natural language

<details>
<summary>Show Answer</summary>
<b>B. May produce hallucinations</b> and <b>D. Outputs can be nondeterministic</b>

Hallucinations (false information) and nondeterminism (varying outputs for same input) are key GenAI limitations.
</details>

---

### Question 27
What does the term "multimodal" mean in the context of AI models?

A. Multiple users can access the model
B. The model can process multiple types of data (text, images, etc.)
C. The model runs on multiple servers
D. The model has multiple versions

<details>
<summary>Show Answer</summary>
<b>B. The model can process multiple types of data (text, images, etc.)</b>

Multimodal models can understand and generate multiple data types like text, images, audio, and video.
</details>

---

### Question 28
What is Amazon Bedrock Knowledge Bases used for?

A. Storing user credentials
B. Implementing Retrieval Augmented Generation (RAG)
C. Training models from scratch
D. Managing billing information

<details>
<summary>Show Answer</summary>
<b>B. Implementing Retrieval Augmented Generation (RAG)</b>

Knowledge Bases enable RAG by connecting your data sources for retrieval-augmented responses.
</details>

---

### Question 29
Which factor does NOT affect the cost of using Foundation Models?

A. Number of input tokens
B. Number of output tokens
C. Model selected
D. Time of day

<details>
<summary>Show Answer</summary>
<b>D. Time of day</b>

FM costs depend on tokens and model choice, not time of day. This differs from some other AWS services.
</details>

---

## Domain 3: Applications of Foundation Models (Questions 30-47)

### Question 30
What effect does lowering the temperature parameter have on model outputs?

A. Outputs become more creative and varied
B. Outputs become more focused and deterministic
C. Outputs become longer
D. Outputs become faster

<details>
<summary>Show Answer</summary>
<b>B. Outputs become more focused and deterministic</b>

Lower temperature (closer to 0) makes outputs more consistent and deterministic; higher temperature increases randomness.
</details>

---

### Question 31
What is the primary benefit of RAG (Retrieval Augmented Generation)?

A. It makes models train faster
B. It allows models to access current, domain-specific information without retraining
C. It reduces the model size
D. It eliminates the need for prompts

<details>
<summary>Show Answer</summary>
<b>B. It allows models to access current, domain-specific information without retraining</b>

RAG retrieves relevant information from external sources, providing current context without model retraining.
</details>

---

### Question 32
Which prompt engineering technique provides multiple examples to guide the model?

A. Zero-shot prompting
B. Chain-of-thought prompting
C. Few-shot prompting
D. Negative prompting

<details>
<summary>Show Answer</summary>
<b>C. Few-shot prompting</b>

Few-shot prompting provides multiple examples (typically 3-5) to help the model understand the desired pattern.
</details>

---

### Question 33
A company wants to add their product documentation to improve chatbot responses. They don't want to retrain the model. What should they use?

A. Fine-tuning
B. RAG with Amazon Bedrock Knowledge Bases
C. Pre-training
D. Reinforcement learning

<details>
<summary>Show Answer</summary>
<b>B. RAG with Amazon Bedrock Knowledge Bases</b>

RAG allows adding knowledge without retraining by retrieving relevant documents during inference.
</details>

---

### Question 34
What is Chain-of-Thought prompting used for?

A. Connecting multiple models together
B. Improving reasoning by showing step-by-step thinking
C. Generating multiple responses
D. Reducing token usage

<details>
<summary>Show Answer</summary>
<b>B. Improving reasoning by showing step-by-step thinking</b>

Chain-of-thought prompting guides the model through logical reasoning steps for complex problems.
</details>

---

### Question 35
Which metric is commonly used to evaluate summarization quality?

A. BLEU
B. ROUGE
C. F1 Score
D. AUC

<details>
<summary>Show Answer</summary>
<b>B. ROUGE</b>

ROUGE (Recall-Oriented Understudy for Gisting Evaluation) is designed for evaluating summaries.
</details>

---

### Question 36
What is prompt injection?

A. Adding more examples to prompts
B. A security attack where malicious input alters model behavior
C. A technique to improve model accuracy
D. A method to reduce costs

<details>
<summary>Show Answer</summary>
<b>B. A security attack where malicious input alters model behavior</b>

Prompt injection is when malicious prompts manipulate the model to behave unintentionally.
</details>

---

### Question 37
What is the purpose of Amazon Bedrock Agents?

A. To monitor model performance
B. To execute multi-step tasks autonomously
C. To train new models
D. To store training data

<details>
<summary>Show Answer</summary>
<b>B. To execute multi-step tasks autonomously</b>

Bedrock Agents can reason, plan, and execute multi-step tasks by calling APIs and using tools.
</details>

---

### Question 38
Which AWS database service can be used as a vector store for RAG?

A. Amazon DynamoDB
B. Amazon OpenSearch Service
C. Amazon RDS for MySQL
D. Amazon ElastiCache

<details>
<summary>Show Answer</summary>
<b>B. Amazon OpenSearch Service</b>

OpenSearch Service supports vector search, making it suitable for RAG implementations.
</details>

---

### Question 39
What is fine-tuning in the context of Foundation Models?

A. Training a model from scratch
B. Adapting a pre-trained model for specific tasks using additional data
C. Reducing model size
D. Increasing temperature

<details>
<summary>Show Answer</summary>
<b>B. Adapting a pre-trained model for specific tasks using additional data</b>

Fine-tuning adjusts a pre-trained model's weights using domain-specific data for better task performance.
</details>

---

### Question 40
What is RLHF (Reinforcement Learning from Human Feedback)?

A. A technique to reduce model size
B. A method to align model outputs with human preferences
C. A way to increase model speed
D. A data encryption method

<details>
<summary>Show Answer</summary>
<b>B. A method to align model outputs with human preferences</b>

RLHF uses human feedback to train a reward model that guides the AI toward preferred behaviors.
</details>

---

### Question 41
When would you choose fine-tuning over RAG?

A. When you need access to current information
B. When you need the model to adopt a specific style or domain expertise
C. When you have limited training data
D. When cost is the primary concern

<details>
<summary>Show Answer</summary>
<b>B. When you need the model to adopt a specific style or domain expertise</b>

Fine-tuning is best when you need consistent style, tone, or deep domain expertise embedded in the model.
</details>

---

### Question 42
What is the BLEU metric used to evaluate?

A. Image generation quality
B. Translation quality
C. Classification accuracy
D. Model speed

<details>
<summary>Show Answer</summary>
<b>B. Translation quality</b>

BLEU (Bilingual Evaluation Understudy) measures the quality of machine-translated text.
</details>

---

### Question 43
Which prompt engineering technique is most effective for simple, well-defined tasks where the model has relevant pre-trained knowledge?

A. Few-shot prompting
B. Zero-shot prompting
C. Chain-of-thought prompting
D. Fine-tuning

<details>
<summary>Show Answer</summary>
<b>B. Zero-shot prompting</b>

Zero-shot works well for simple tasks where the model's pre-trained knowledge is sufficient without examples.
</details>

---

### Question 44
What is the purpose of guardrails in Amazon Bedrock?

A. To increase model speed
B. To filter content and implement safety controls
C. To reduce costs
D. To train models

<details>
<summary>Show Answer</summary>
<b>B. To filter content and implement safety controls</b>

Guardrails filter harmful content, block topics, detect PII, and implement safety policies.
</details>

---

### Question 45
A company needs to evaluate multiple Foundation Models for their use case. Which Amazon Bedrock feature should they use?

A. Knowledge Bases
B. Agents
C. Model Evaluation
D. Guardrails

<details>
<summary>Show Answer</summary>
<b>C. Model Evaluation</b>

Bedrock Model Evaluation allows comparing models using automatic metrics and human evaluation.
</details>

---

### Question 46
What is in-context learning?

A. Training a model with new data
B. Providing examples in the prompt to guide model behavior without training
C. Running the model in a specific region
D. Encrypting model inputs

<details>
<summary>Show Answer</summary>
<b>B. Providing examples in the prompt to guide model behavior without training</b>

In-context learning uses examples in the prompt itself (few-shot) to guide the model without changing weights.
</details>

---

### Question 47
Which is the correct order of FM customization methods from lowest to highest cost?

A. Fine-tuning → RAG → In-context learning → Pre-training
B. In-context learning → RAG → Fine-tuning → Pre-training
C. Pre-training → Fine-tuning → RAG → In-context learning
D. RAG → In-context learning → Pre-training → Fine-tuning

<details>
<summary>Show Answer</summary>
<b>B. In-context learning → RAG → Fine-tuning → Pre-training</b>

Cost increases: In-context (just prompts) → RAG (add retrieval) → Fine-tuning (train on data) → Pre-training (train from scratch).
</details>

---

## Domain 4: Guidelines for Responsible AI (Questions 48-56)

### Question 48
Which of the following are features of responsible AI? (Select TWO)

A. Maximum model size
B. Fairness
C. Highest possible accuracy only
D. Transparency
E. Fastest inference time

<details>
<summary>Show Answer</summary>
<b>B. Fairness</b> and <b>D. Transparency</b>

Responsible AI features include fairness, transparency, inclusivity, robustness, safety, veracity, and privacy.
</details>

---

### Question 49
What is bias in machine learning?

A. A feature that improves accuracy
B. Systematic errors that lead to unfair outcomes for certain groups
C. The optimal model parameter
D. A type of neural network

<details>
<summary>Show Answer</summary>
<b>B. Systematic errors that lead to unfair outcomes for certain groups</b>

Bias causes models to perform differently or unfairly across different groups or demographics.
</details>

---

### Question 50
Which AWS service is primarily used for detecting bias in ML models?

A. Amazon SageMaker Clarify
B. Amazon SageMaker Model Monitor
C. Amazon Bedrock Guardrails
D. Amazon Comprehend

<details>
<summary>Show Answer</summary>
<b>A. Amazon SageMaker Clarify</b>

SageMaker Clarify is designed specifically for bias detection and model explainability.
</details>

---

### Question 51
What is the relationship between model complexity and explainability?

A. More complex models are always more explainable
B. There is typically a tradeoff - more complex models are often less explainable
C. Complexity has no effect on explainability
D. Simple models cannot be explained

<details>
<summary>Show Answer</summary>
<b>B. There is typically a tradeoff - more complex models are often less explainable</b>

Complex models (deep learning) often perform better but are harder to interpret than simpler models.
</details>

---

### Question 52
Which AWS service adds human review to ML prediction workflows?

A. Amazon SageMaker Clarify
B. Amazon Augmented AI (A2I)
C. Amazon SageMaker Model Monitor
D. Amazon Comprehend

<details>
<summary>Show Answer</summary>
<b>B. Amazon Augmented AI (A2I)</b>

A2I enables human oversight and review of ML predictions for quality assurance.
</details>

---

### Question 53
What does high variance in a model indicate?

A. The model is too simple
B. The model is overfitting to training data
C. The model has low accuracy
D. The model is fair

<details>
<summary>Show Answer</summary>
<b>B. The model is overfitting to training data</b>

High variance means the model is too complex and fits training data too closely, failing to generalize.
</details>

---

### Question 54
What is the purpose of Amazon SageMaker Model Cards?

A. To deploy models
B. To document model details, intended use, and limitations
C. To train models faster
D. To monitor model performance

<details>
<summary>Show Answer</summary>
<b>B. To document model details, intended use, and limitations</b>

Model Cards provide standardized documentation for model transparency and responsible AI practices.
</details>

---

### Question 55
Which is a legal risk associated with Generative AI?

A. Faster processing
B. Intellectual property infringement from generated content
C. Lower costs
D. Improved accuracy

<details>
<summary>Show Answer</summary>
<b>B. Intellectual property infringement from generated content</b>

GenAI may generate content similar to copyrighted material, creating IP infringement risks.
</details>

---

### Question 56
What characteristic should training datasets have to reduce bias?

A. As large as possible only
B. Diverse and representative of all user groups
C. Only from one source
D. Exclusively recent data

<details>
<summary>Show Answer</summary>
<b>B. Diverse and representative of all user groups</b>

Datasets should be inclusive, diverse, balanced, and representative to minimize bias.
</details>

---

## Domain 5: Security, Compliance, and Governance (Questions 57-65)

### Question 57
Under the AWS Shared Responsibility Model, who is responsible for the security of data used in AI models?

A. AWS only
B. Customer only
C. Both equally
D. Third-party vendors

<details>
<summary>Show Answer</summary>
<b>B. Customer only</b>

Customers are responsible for their data (security in the cloud); AWS secures the infrastructure (security of the cloud).
</details>

---

### Question 58
Which AWS service logs all API calls made in your AWS account?

A. AWS Config
B. AWS CloudTrail
C. Amazon Inspector
D. AWS Trusted Advisor

<details>
<summary>Show Answer</summary>
<b>B. AWS CloudTrail</b>

CloudTrail records API calls for audit trails, showing who did what and when.
</details>

---

### Question 59
What is the purpose of AWS PrivateLink in AI workloads?

A. To increase model accuracy
B. To enable private connectivity without using the public internet
C. To reduce costs
D. To train models faster

<details>
<summary>Show Answer</summary>
<b>B. To enable private connectivity without using the public internet</b>

PrivateLink creates VPC endpoints for private, secure access to AWS services.
</details>

---

### Question 60
Which service discovers and protects sensitive data in Amazon S3?

A. AWS Config
B. Amazon Macie
C. AWS CloudTrail
D. Amazon Inspector

<details>
<summary>Show Answer</summary>
<b>B. Amazon Macie</b>

Macie uses ML to discover, classify, and protect sensitive data (like PII) in S3.
</details>

---

### Question 61
What is data lineage?

A. The encryption method used for data
B. Tracking data from its source through all transformations to usage
C. The physical location of data
D. The age of the data

<details>
<summary>Show Answer</summary>
<b>B. Tracking data from its source through all transformations to usage</b>

Data lineage documents the origin, movement, and transformation of data throughout its lifecycle.
</details>

---

### Question 62
Which AWS service helps prepare for compliance audits?

A. AWS CloudTrail
B. AWS Config
C. AWS Audit Manager
D. AWS Artifact

<details>
<summary>Show Answer</summary>
<b>C. AWS Audit Manager</b>

Audit Manager collects evidence and maps to compliance frameworks for audit preparation.
</details>

---

### Question 63
What type of encryption protects data while it is being transferred?

A. Encryption at rest
B. Encryption in transit
C. Client-side encryption
D. Server-side encryption

<details>
<summary>Show Answer</summary>
<b>B. Encryption in transit</b>

Encryption in transit (TLS/SSL) protects data during transfer; encryption at rest protects stored data.
</details>

---

### Question 64
Which is a best practice for IAM in AI workloads?

A. Use root account for all operations
B. Share credentials between team members
C. Apply least privilege principle
D. Disable MFA for convenience

<details>
<summary>Show Answer</summary>
<b>C. Apply least privilege principle</b>

Grant only the minimum permissions needed. Use roles, enable MFA, and never share credentials.
</details>

---

### Question 65
Which service provides access to AWS compliance reports and certifications?

A. AWS Config
B. AWS CloudTrail
C. AWS Audit Manager
D. AWS Artifact

<details>
<summary>Show Answer</summary>
<b>D. AWS Artifact</b>

AWS Artifact provides on-demand access to AWS compliance reports (SOC, ISO, PCI-DSS, etc.).
</details>

---

## Scoring Guide

Calculate your score:
- Each correct answer = 1 point
- Total possible = 65 points
- Passing threshold ≈ 70% (46 correct)

| Score | Status | Recommendation |
|-------|--------|----------------|
| 58-65 (89-100%) | Excellent | Ready for the exam |
| 46-57 (70-88%) | Good | Review weak areas |
| 33-45 (50-69%) | Needs Work | Study more, retake practice exam |
| 0-32 (<50%) | More Study Needed | Review all domains thoroughly |

### Score by Domain

| Domain | Questions | Your Score | Percentage |
|--------|-----------|------------|------------|
| Domain 1 | 1-13 (13 Q) | ___ /13 | ___% |
| Domain 2 | 14-29 (16 Q) | ___ /16 | ___% |
| Domain 3 | 30-47 (18 Q) | ___ /18 | ___% |
| Domain 4 | 48-56 (9 Q) | ___ /9 | ___% |
| Domain 5 | 57-65 (9 Q) | ___ /9 | ___% |
| **Total** | **65 Q** | **___ /65** | **___%** |

---

## Answer Key

| Q | A | Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|---|---|
| 1 | B | 14 | B | 27 | B | 40 | B | 53 | B |
| 2 | B | 15 | C | 28 | B | 41 | B | 54 | B |
| 3 | B | 16 | B | 29 | D | 42 | B | 55 | B |
| 4 | C | 17 | B | 30 | B | 43 | B | 56 | B |
| 5 | C | 18 | B | 31 | B | 44 | B | 57 | B |
| 6 | B | 19 | B | 32 | C | 45 | C | 58 | B |
| 7 | C | 20 | C | 33 | B | 46 | B | 59 | B |
| 8 | B | 21 | C | 34 | B | 47 | B | 60 | B |
| 9 | B | 22 | B | 35 | B | 48 | B,D | 61 | B |
| 10 | B | 23 | B | 36 | B | 49 | B | 62 | C |
| 11 | B | 24 | B | 37 | B | 50 | A | 63 | B |
| 12 | C | 25 | B | 38 | B | 51 | B | 64 | C |
| 13 | B | 26 | B,D | 39 | B | 52 | B | 65 | D |

---

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)

---

*How did you score? Share in the comments below!*
