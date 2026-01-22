---
title: "AWS AIF-C01 - Domain 2: Fundamentals of Generative AI"
author: thanhnv1808
date: 2026-01-22 10:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, generative-ai, llm, foundation-models, bedrock]
description: Domain 2 covers 24% of the AWS AI Practitioner exam. Learn Generative AI concepts, Foundation Models, and AWS GenAI services.
pin: false
comments: true
---

## Domain Overview

**Domain 2: Fundamentals of Generative AI** represents **24%** of the exam (approximately 12 questions).

This domain covers three main task statements:
1. Explain the basic concepts of Generative AI
2. Understand the capabilities and limitations of GenAI
3. Describe AWS infrastructure and technologies for building GenAI applications

---

## Task 2.1: Basic Concepts of Generative AI

### What is Generative AI?

**Generative AI** is a type of artificial intelligence that can create new content such as:
- Text (articles, code, summaries)
- Images (art, photos, designs)
- Audio (music, speech)
- Video (animations, deepfakes)
- Code (programs, scripts)

```
Traditional AI: Analyzes and classifies existing content
Generative AI: Creates new, original content
```

### Key GenAI Terminology

| Term | Definition |
|------|------------|
| **Foundation Model (FM)** | Large pre-trained model that can be adapted for various tasks |
| **Large Language Model (LLM)** | FM trained on massive text data for language tasks |
| **Token** | Basic unit of text (word, subword, or character) |
| **Embedding** | Numerical representation of text/data |
| **Vector** | Mathematical array representing embeddings |
| **Chunking** | Breaking text into smaller pieces for processing |
| **Prompt** | Input text given to a model to generate a response |
| **Inference** | Process of generating outputs from a trained model |

### Tokens and Tokenization

```
Input: "Hello, how are you?"

Tokenization examples:
Word-level:    ["Hello", ",", "how", "are", "you", "?"]
Subword:       ["Hello", ",", "how", "are", "you", "?"]
Character:     ["H", "e", "l", "l", "o", ",", ...]
```

> **Exam Tip**: Token count affects cost and processing time. More tokens = higher cost.
{: .prompt-tip }

### Embeddings and Vectors

**Embeddings** convert text into numerical vectors that capture semantic meaning.

```
"King" - "Man" + "Woman" ≈ "Queen"

Vector representation allows mathematical operations on language.
```

| Concept | Description | Use Case |
|---------|-------------|----------|
| **Word Embeddings** | Vector for individual words | Word similarity |
| **Sentence Embeddings** | Vector for entire sentences | Semantic search |
| **Document Embeddings** | Vector for documents | Document comparison |

### Types of Foundation Models

#### 1. Large Language Models (LLMs)
- **Purpose**: Text generation, understanding, and analysis
- **Examples**: Claude, GPT-4, Llama
- **Use Cases**: Chatbots, content creation, code generation

#### 2. Multimodal Models
- **Purpose**: Process multiple data types (text, images, audio)
- **Examples**: Claude 3, GPT-4V
- **Use Cases**: Image captioning, visual Q&A

#### 3. Diffusion Models
- **Purpose**: Generate images from text descriptions
- **Examples**: Stable Diffusion, DALL-E
- **Use Cases**: Art generation, design

#### 4. Transformer Models
- **Architecture**: Attention mechanism for parallel processing
- **Key Innovation**: Self-attention to understand context
- **Foundation for**: Most modern LLMs

### Foundation Model Lifecycle

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Data         │ -> │ Model        │ -> │ Pre-training │
│ Selection    │    │ Selection    │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
                                               │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Feedback     │ <- │ Deployment   │ <- │ Fine-tuning  │
│              │    │              │    │ & Evaluation │
└──────────────┘    └──────────────┘    └──────────────┘
```

| Stage | Description |
|-------|-------------|
| **Data Selection** | Choose training data sources |
| **Model Selection** | Choose base architecture |
| **Pre-training** | Train on large datasets |
| **Fine-tuning** | Adapt for specific tasks |
| **Evaluation** | Test model performance |
| **Deployment** | Make model available |
| **Feedback** | Collect user feedback for improvement |

### GenAI Use Cases

| Use Case | Description | Example |
|----------|-------------|---------|
| **Text Generation** | Create written content | Articles, emails, reports |
| **Code Generation** | Write and debug code | GitHub Copilot, Amazon Q Developer |
| **Image Generation** | Create images from text | Marketing graphics, art |
| **Summarization** | Condense long documents | Meeting notes, articles |
| **Translation** | Convert between languages | Real-time translation |
| **AI Assistants** | Conversational interfaces | Customer support, Q&A |
| **Search Enhancement** | Semantic search | Knowledge bases |
| **Recommendations** | Personalized suggestions | Product recommendations |

---

## Task 2.2: Capabilities and Limitations of GenAI

### Advantages of GenAI

| Advantage | Description |
|-----------|-------------|
| **Adaptability** | Can be applied to many different tasks |
| **Responsiveness** | Quick generation of content |
| **Simplicity** | Natural language interface |
| **Scalability** | Handle many requests simultaneously |
| **Creativity** | Generate novel content |
| **Efficiency** | Automate time-consuming tasks |

### Limitations and Challenges

| Limitation | Description | Impact |
|------------|-------------|--------|
| **Hallucinations** | Generating false or made-up information | Incorrect outputs |
| **Interpretability** | Hard to understand why model gave certain output | Trust issues |
| **Inaccuracy** | May produce factually wrong content | Quality concerns |
| **Nondeterminism** | Same input can produce different outputs | Inconsistency |
| **Bias** | May reflect biases in training data | Unfair outcomes |
| **Knowledge Cutoff** | Limited to training data date | Outdated information |
| **Context Limits** | Limited input/output length | Long document challenges |

> **Exam Tip**: Hallucinations are a key limitation - models can confidently generate incorrect information.
{: .prompt-warning }

### Model Selection Factors

| Factor | Description | Consideration |
|--------|-------------|---------------|
| **Model Type** | LLM, multimodal, diffusion | Match to use case |
| **Performance** | Speed, accuracy, quality | Balance with cost |
| **Capabilities** | Features supported | Required functionality |
| **Constraints** | Size, latency, cost | Infrastructure limits |
| **Compliance** | Regulatory requirements | Data handling, privacy |
| **Cost** | Per-token or per-request pricing | Budget constraints |

### Business Value Metrics

| Metric | Description |
|--------|-------------|
| **Cross-domain Performance** | How well model works across different tasks |
| **Efficiency** | Time and cost savings |
| **Conversion Rate** | Impact on user actions |
| **ARPU** | Average Revenue Per User improvement |
| **Accuracy** | Correctness of outputs |
| **Customer Lifetime Value** | Long-term customer value impact |

---

## Task 2.3: AWS GenAI Infrastructure and Services

### Core AWS GenAI Services

#### Amazon Bedrock
The primary AWS service for accessing Foundation Models.

| Feature | Description |
|---------|-------------|
| **Model Access** | Access to multiple FMs (Claude, Llama, Titan, etc.) |
| **Serverless** | No infrastructure management |
| **Customization** | Fine-tune models with your data |
| **Knowledge Bases** | RAG implementation |
| **Guardrails** | Safety and compliance controls |
| **Agents** | Orchestrate multi-step tasks |

> **Exam Tip**: Amazon Bedrock is the main AWS service for accessing and using Foundation Models.
{: .prompt-tip }

#### Amazon Bedrock Features

| Feature | Purpose |
|---------|---------|
| **Bedrock Knowledge Bases** | Implement RAG with your data |
| **Bedrock Guardrails** | Content filtering and safety |
| **Bedrock Agents** | Autonomous task execution |
| **Bedrock Model Evaluation** | Compare and evaluate models |
| **Bedrock Studio** | Visual model experimentation |

#### Amazon SageMaker JumpStart
- **Purpose**: Access pre-trained models and deploy with one click
- **Features**: Model hub, easy deployment, fine-tuning
- **Use Case**: Custom ML model development

#### Amazon Q
AWS AI assistant for business and development:

| Variant | Purpose |
|---------|---------|
| **Amazon Q Business** | Enterprise knowledge assistant |
| **Amazon Q Developer** | Code assistance and generation |
| **Amazon Q in QuickSight** | BI and analytics assistance |
| **Amazon Q in Connect** | Customer service assistance |

#### PartyRock
- **Purpose**: No-code GenAI application builder
- **Use Case**: Prototyping and learning
- **Access**: Free, browser-based

### AWS GenAI Advantages

| Advantage | Description |
|-----------|-------------|
| **Accessibility** | Easy access to multiple FMs |
| **Lower Barrier** | No need to train your own models |
| **Efficiency** | Managed infrastructure |
| **Cost-effective** | Pay-per-use pricing |
| **Speed to Market** | Rapid application development |
| **Security** | Enterprise-grade security |
| **Compliance** | Meet regulatory requirements |

### AWS Infrastructure Benefits for GenAI

| Benefit | Description |
|---------|-------------|
| **Security** | Data encryption, VPC integration |
| **Compliance** | HIPAA, SOC, ISO certifications |
| **Shared Responsibility** | Clear security boundaries |
| **Safety** | Built-in content filtering |
| **Scalability** | Handle varying workloads |
| **Global Reach** | Multiple regions available |

### Cost Considerations

| Pricing Model | Description | Use Case |
|---------------|-------------|----------|
| **Token-based** | Pay per input/output token | Variable usage |
| **Provisioned Throughput** | Reserved capacity | Consistent usage |
| **Custom Models** | Training and hosting costs | Specialized needs |

#### Cost Tradeoffs

| Factor | Tradeoff |
|--------|----------|
| **Responsiveness** | Faster = More expensive |
| **Availability** | Higher = More expensive |
| **Redundancy** | More = More expensive |
| **Performance** | Better = More expensive |
| **Model Size** | Larger = More expensive |

---

## Comparing AWS GenAI Services

| Service | Best For | Complexity |
|---------|----------|------------|
| **PartyRock** | Learning and prototyping | Lowest |
| **Amazon Q** | Business/developer assistance | Low |
| **Bedrock** | Production GenAI apps | Medium |
| **SageMaker JumpStart** | Custom ML/FM deployment | Medium-High |
| **SageMaker** | Full ML development | Highest |

---

## Practice Questions

### Question 1
What is the primary purpose of tokenization in Large Language Models?

A. To encrypt the input text
B. To break text into smaller units for processing
C. To translate text between languages
D. To compress the model size

<details>
<summary>Show Answer</summary>
<b>B. To break text into smaller units for processing</b>

Tokenization converts text into tokens (words, subwords, or characters) that the model can process.
</details>

### Question 2
Which AWS service provides serverless access to multiple Foundation Models like Claude and Llama?

A. Amazon SageMaker
B. Amazon Bedrock
C. Amazon Comprehend
D. Amazon Lex

<details>
<summary>Show Answer</summary>
<b>B. Amazon Bedrock</b>

Amazon Bedrock is the managed service for accessing Foundation Models without managing infrastructure.
</details>

### Question 3
A GenAI model confidently provides an incorrect answer that sounds plausible. What is this phenomenon called?

A. Bias
B. Overfitting
C. Hallucination
D. Underfitting

<details>
<summary>Show Answer</summary>
<b>C. Hallucination</b>

Hallucinations occur when GenAI models generate false or made-up information that appears convincing.
</details>

### Question 4
Which type of Foundation Model would you use to generate images from text descriptions?

A. Large Language Model (LLM)
B. Transformer Model
C. Diffusion Model
D. Recurrent Neural Network

<details>
<summary>Show Answer</summary>
<b>C. Diffusion Model</b>

Diffusion models (like Stable Diffusion, DALL-E) are specifically designed for image generation from text.
</details>

### Question 5
What is the main advantage of using embeddings in GenAI?

A. They reduce model size
B. They convert text to numerical representations for semantic operations
C. They encrypt sensitive data
D. They speed up training

<details>
<summary>Show Answer</summary>
<b>B. They convert text to numerical representations for semantic operations</b>

Embeddings represent text as vectors, enabling mathematical operations that capture semantic meaning.
</details>

### Question 6
Which Amazon Bedrock feature helps implement Retrieval Augmented Generation (RAG)?

A. Bedrock Guardrails
B. Bedrock Agents
C. Bedrock Knowledge Bases
D. Bedrock Model Evaluation

<details>
<summary>Show Answer</summary>
<b>C. Bedrock Knowledge Bases</b>

Bedrock Knowledge Bases allows you to connect your data sources for RAG implementation.
</details>

---

## Key Takeaways

1. **Foundation Models**: Large pre-trained models that can be adapted for many tasks
2. **LLMs**: Specialized FMs for language tasks (text generation, understanding)
3. **Tokens**: Basic units of text; more tokens = higher cost
4. **Embeddings**: Numerical vectors that capture semantic meaning
5. **Hallucinations**: Models can generate confident but incorrect information
6. **Amazon Bedrock**: Primary AWS service for accessing FMs
7. **Pricing**: Token-based for variable usage, provisioned throughput for consistent usage

### AWS Service Selection Guide

```
Need to build GenAI apps?          → Amazon Bedrock
Need business/code assistant?      → Amazon Q
Learning/prototyping?              → PartyRock
Custom model training?             → SageMaker JumpStart
Full ML development?               → SageMaker
```

---

**Previous Lesson**: [Domain 1: Fundamentals of AI and ML](/posts/aws-aif-c01-domain-1-fundamentals-ai-ml/)

**Next Lesson**: [Domain 3: Applications of Foundation Models](/posts/aws-aif-c01-domain-3-foundation-models/)

---

*Questions about Generative AI fundamentals? Leave a comment below!*
