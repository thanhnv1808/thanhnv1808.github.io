---
title: "AWS AIF-C01 - Domain 3: Applications of Foundation Models"
author: thanhnv1808
date: 2026-01-22 11:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, foundation-models, rag, prompt-engineering, bedrock]
description: Domain 3 covers 28% of the AWS AI Practitioner exam. Learn about RAG, prompt engineering, fine-tuning, and model evaluation.
pin: false
comments: true
---

## Domain Overview

**Domain 3: Applications of Foundation Models** represents **28%** of the exam (approximately 14 questions) - the largest domain.

This domain covers four main task statements:
1. Describe design considerations for FM applications
2. Choose effective prompt engineering techniques
3. Describe the training and fine-tuning process for FMs
4. Describe methods to evaluate FM performance

---

## Task 3.1: Design Considerations for FM Applications

### Model Selection Criteria

| Criteria | Description | Considerations |
|----------|-------------|----------------|
| **Cost** | Pricing per token or request | Budget constraints |
| **Modality** | Text, image, audio, multimodal | Required input/output types |
| **Latency** | Response time | Real-time vs. batch needs |
| **Multi-lingual** | Language support | Target audience |
| **Model Size** | Parameters count | Performance vs. cost |
| **Complexity** | Model capabilities | Task requirements |
| **Customization** | Fine-tuning options | Specific domain needs |
| **Input/Output Length** | Context window size | Document length needs |

### Inference Parameters

#### Temperature
Controls randomness in model outputs.

| Temperature | Effect | Use Case |
|-------------|--------|----------|
| **0.0** | Deterministic, consistent | Factual Q&A, code |
| **0.1-0.3** | Low creativity | Technical writing |
| **0.5-0.7** | Balanced | General tasks |
| **0.8-1.0** | High creativity | Creative writing, brainstorming |

```
Temperature = 0.0 → Same input always produces same output
Temperature = 1.0 → More varied, creative outputs
```

> **Exam Tip**: Lower temperature = more focused/consistent; Higher temperature = more creative/varied.
{: .prompt-tip }

#### Other Inference Parameters

| Parameter | Description |
|-----------|-------------|
| **Max Tokens** | Maximum output length |
| **Top P** | Nucleus sampling threshold |
| **Top K** | Consider top K most likely tokens |
| **Stop Sequences** | Tokens that stop generation |

### Retrieval Augmented Generation (RAG)

**RAG** combines Foundation Models with external knowledge retrieval.

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│   User      │ --> │ Knowledge Base   │ --> │   Model     │
│   Query     │     │ (Vector Search)  │     │  + Context  │
└─────────────┘     └──────────────────┘     └─────────────┘
                            │                       │
                    ┌───────v───────┐       ┌───────v───────┐
                    │   Relevant    │       │   Enhanced    │
                    │   Documents   │       │   Response    │
                    └───────────────┘       └───────────────┘
```

#### RAG Benefits

| Benefit | Description |
|---------|-------------|
| **Current Information** | Access up-to-date knowledge |
| **Domain-Specific** | Use proprietary data |
| **Reduced Hallucinations** | Grounded in real documents |
| **No Retraining** | Update knowledge without model changes |
| **Source Attribution** | Cite where information came from |
| **Cost-Effective** | Cheaper than fine-tuning |

#### RAG Implementation with Amazon Bedrock

**Amazon Bedrock Knowledge Bases** provides managed RAG:

1. **Data Sources**: S3, web crawlers, databases
2. **Embeddings**: Convert documents to vectors
3. **Vector Store**: Store and search embeddings
4. **Retrieval**: Find relevant documents
5. **Augmentation**: Add context to prompts
6. **Generation**: Generate grounded responses

### Vector Databases for RAG

| AWS Service | Description | Use Case |
|-------------|-------------|----------|
| **Amazon OpenSearch Service** | Full-text and vector search | Large-scale search |
| **Amazon Aurora** | Vector support with pgvector | Relational + vector |
| **Amazon Neptune** | Graph database with vectors | Knowledge graphs |
| **Amazon RDS for PostgreSQL** | PostgreSQL with pgvector | General purpose |

### FM Customization Options

| Method | Training Data | Cost | Complexity | Use Case |
|--------|--------------|------|------------|----------|
| **In-context Learning** | None (prompt examples) | Low | Low | Quick adaptation |
| **RAG** | Your documents | Medium | Medium | Current knowledge |
| **Fine-tuning** | Labeled examples | High | High | Domain expertise |
| **Continued Pre-training** | Large corpus | Very High | Very High | New knowledge |

> **Exam Tip**: Know the cost/complexity tradeoffs: In-context < RAG < Fine-tuning < Pre-training.
{: .prompt-warning }

### Agents and Multi-Step Tasks

#### Amazon Bedrock Agents
Autonomous systems that can:
- Break down complex tasks
- Call external APIs and tools
- Maintain conversation context
- Execute multi-step workflows

```
User Request → Agent → Plan Steps → Execute Actions → Return Result
                 ↓
         Tools/APIs/Functions
```

#### Key Agent Concepts

| Concept | Description |
|---------|-------------|
| **Agent** | Orchestrates tasks and tools |
| **Action Groups** | Collections of actions the agent can take |
| **Knowledge Base** | External information sources |
| **Instructions** | Define agent behavior |
| **Model Context Protocol (MCP)** | Standard for tool integration |

---

## Task 3.2: Prompt Engineering Techniques

### Prompt Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Instruction** | What to do | "Summarize the following text" |
| **Context** | Background information | Company information, domain knowledge |
| **Input** | Data to process | The text to summarize |
| **Output Format** | Desired response format | "Respond in bullet points" |
| **Negative Prompts** | What NOT to do | "Do not include opinions" |

### Prompt Engineering Techniques

#### 1. Zero-Shot Prompting
No examples provided - rely on model's pre-trained knowledge.

```
Prompt: "Translate to French: Hello, how are you?"
Output: "Bonjour, comment allez-vous?"
```
- **Best for**: Simple, well-defined tasks
- **Limitation**: May not work for complex tasks

#### 2. Single-Shot (One-Shot) Prompting
One example provided to guide the model.

```
Prompt:
"Translate to French:
English: Good morning → French: Bonjour
English: Hello, how are you? → French:"

Output: "Bonjour, comment allez-vous?"
```

#### 3. Few-Shot Prompting
Multiple examples to establish pattern.

```
Prompt:
"Classify the sentiment:
Text: I love this product! → Positive
Text: This is terrible. → Negative
Text: It works fine. → Neutral
Text: Best purchase ever! → "

Output: "Positive"
```
- **Best for**: Pattern-based tasks
- **Tip**: Use 3-5 diverse examples

#### 4. Chain-of-Thought (CoT) Prompting
Guide model through reasoning steps.

```
Prompt:
"Q: If there are 3 apples and you take away 2, how many do you have?
Let's think step by step:
1. You started with 0 apples
2. You took 2 apples
3. Now you have 2 apples

A: 2 apples (the ones you took)"
```
- **Best for**: Math, logic, complex reasoning
- **Key phrase**: "Let's think step by step"

> **Exam Tip**: Chain-of-thought improves reasoning by showing step-by-step thinking.
{: .prompt-tip }

### Prompt Best Practices

| Practice | Description |
|----------|-------------|
| **Be Specific** | Clear, unambiguous instructions |
| **Be Concise** | Remove unnecessary words |
| **Use Examples** | Show expected format |
| **Set Constraints** | Define boundaries |
| **Iterate** | Test and refine prompts |
| **Use Delimiters** | Separate sections clearly |

### Prompt Templates

```
You are a [ROLE].
Your task is to [TASK].

Context:
[RELEVANT BACKGROUND]

Instructions:
1. [STEP 1]
2. [STEP 2]
3. [STEP 3]

Input:
[USER INPUT]

Output format:
[DESIRED FORMAT]

Constraints:
- [CONSTRAINT 1]
- [CONSTRAINT 2]
```

### Prompt Security Risks

| Risk | Description | Mitigation |
|------|-------------|------------|
| **Prompt Injection** | Malicious input alters behavior | Input validation, guardrails |
| **Prompt Leaking** | System prompt exposed | Hide sensitive instructions |
| **Jailbreaking** | Bypass safety measures | Multiple safety layers |
| **Data Extraction** | Extract training data | Output filtering |

> **Exam Tip**: Amazon Bedrock Guardrails helps mitigate prompt security risks.
{: .prompt-warning }

---

## Task 3.3: Training and Fine-Tuning Process

### FM Training Methods

| Method | Description | Data Needed | Cost |
|--------|-------------|-------------|------|
| **Pre-training** | Train from scratch | Massive datasets | Very High |
| **Continued Pre-training** | Extend existing model | Large domain corpus | High |
| **Fine-tuning** | Adapt for specific tasks | Labeled examples | Medium |
| **Instruction Tuning** | Train to follow instructions | Instruction-response pairs | Medium |
| **Distillation** | Create smaller model from larger | Teacher model outputs | Low-Medium |

### Fine-Tuning Deep Dive

**Fine-tuning** adapts a pre-trained model for specific tasks or domains.

```
Pre-trained Model → Fine-tuning Data → Domain-Specific Model
     (General)         (Your Data)         (Specialized)
```

#### When to Fine-Tune

| Scenario | Recommendation |
|----------|----------------|
| General Q&A | Use base model |
| Need current info | Use RAG |
| Specific domain language | Fine-tune |
| Consistent output format | Fine-tune |
| Proprietary terminology | Fine-tune |

#### Fine-Tuning Requirements

| Requirement | Description |
|-------------|-------------|
| **Quality Data** | Clean, representative examples |
| **Sufficient Quantity** | Enough examples (100s-1000s) |
| **Proper Labels** | Correct input-output pairs |
| **Diverse Examples** | Cover all variations |
| **Validation Set** | Test performance |

### Data Preparation for Fine-Tuning

| Step | Description |
|------|-------------|
| **Data Curation** | Select relevant, quality data |
| **Data Cleaning** | Remove noise, errors |
| **Labeling** | Create input-output pairs |
| **Balancing** | Ensure representative distribution |
| **Governance** | Ensure compliance, privacy |

### RLHF (Reinforcement Learning from Human Feedback)

Process to align models with human preferences:

```
1. Collect human feedback on model outputs
2. Train reward model based on preferences
3. Use RL to optimize model for higher rewards
4. Iterate to improve alignment
```

| Step | Description |
|------|-------------|
| **Feedback Collection** | Humans rate model outputs |
| **Reward Modeling** | Learn what humans prefer |
| **Policy Optimization** | Adjust model behavior |
| **Iteration** | Continuous improvement |

### Transfer Learning

Apply knowledge from one task to another:

```
Source Task (General) → Transfer → Target Task (Specific)
   Pre-trained Model    Learning    Fine-tuned Model
```

---

## Task 3.4: Methods to Evaluate FM Performance

### Evaluation Approaches

| Approach | Description | When to Use |
|----------|-------------|-------------|
| **Human Evaluation** | Human judges rate outputs | Quality assessment |
| **Automatic Metrics** | Computed scores | Large-scale testing |
| **Benchmark Datasets** | Standard test sets | Model comparison |
| **A/B Testing** | Compare versions | Production testing |

### Automatic Evaluation Metrics

#### Text Generation Metrics

| Metric | Full Name | Measures |
|--------|-----------|----------|
| **ROUGE** | Recall-Oriented Understudy for Gisting Evaluation | Summary quality |
| **BLEU** | Bilingual Evaluation Understudy | Translation quality |
| **BERTScore** | BERT-based similarity | Semantic similarity |
| **Perplexity** | Model uncertainty | Language modeling quality |

#### ROUGE Variants

| Variant | Description |
|---------|-------------|
| **ROUGE-1** | Unigram overlap |
| **ROUGE-2** | Bigram overlap |
| **ROUGE-L** | Longest common subsequence |

> **Exam Tip**: ROUGE is commonly used for summarization; BLEU is commonly used for translation.
{: .prompt-tip }

### Amazon Bedrock Model Evaluation

Features:
- Compare multiple models
- Automatic and human evaluation
- Custom evaluation criteria
- Built-in benchmark datasets

### Business-Focused Evaluation

| Metric | Description |
|--------|-------------|
| **Task Completion Rate** | % of tasks successfully completed |
| **User Satisfaction** | User feedback scores |
| **Time Savings** | Productivity improvement |
| **Accuracy** | Correctness of outputs |
| **Engagement** | User interaction levels |

### RAG Evaluation

| Metric | Description |
|--------|-------------|
| **Retrieval Precision** | Relevance of retrieved docs |
| **Retrieval Recall** | Coverage of relevant docs |
| **Answer Accuracy** | Correctness of final answer |
| **Groundedness** | Is answer supported by sources? |
| **Faithfulness** | Does answer match sources? |

### Agent Evaluation

| Metric | Description |
|--------|-------------|
| **Task Success Rate** | % of tasks completed correctly |
| **Step Efficiency** | Number of steps taken |
| **Tool Selection Accuracy** | Correct tool choices |
| **Error Recovery** | Handling of failures |

---

## Practice Questions

### Question 1
What is the primary benefit of using Retrieval Augmented Generation (RAG)?

A. It reduces model training time
B. It allows the model to access current, domain-specific information
C. It decreases the model size
D. It eliminates the need for prompts

<details>
<summary>Show Answer</summary>
<b>B. It allows the model to access current, domain-specific information</b>

RAG retrieves relevant information from external sources, providing the model with current and domain-specific context.
</details>

### Question 2
What effect does lowering the temperature parameter have on model outputs?

A. Outputs become more creative and varied
B. Outputs become more deterministic and consistent
C. Outputs become longer
D. Outputs become faster

<details>
<summary>Show Answer</summary>
<b>B. Outputs become more deterministic and consistent</b>

Lower temperature values make the model more focused and deterministic, producing consistent outputs.
</details>

### Question 3
Which prompt engineering technique involves providing 3-5 examples to guide the model?

A. Zero-shot prompting
B. Chain-of-thought prompting
C. Few-shot prompting
D. Negative prompting

<details>
<summary>Show Answer</summary>
<b>C. Few-shot prompting</b>

Few-shot prompting provides multiple examples (typically 3-5) to help the model understand the desired pattern.
</details>

### Question 4
Which metric is most commonly used to evaluate summarization quality?

A. BLEU
B. ROUGE
C. Perplexity
D. F1 Score

<details>
<summary>Show Answer</summary>
<b>B. ROUGE</b>

ROUGE (Recall-Oriented Understudy for Gisting Evaluation) is specifically designed for evaluating summaries.
</details>

### Question 5
A company needs to add current product information to their chatbot without retraining the model. Which approach should they use?

A. Fine-tuning
B. Pre-training
C. RAG with Amazon Bedrock Knowledge Bases
D. RLHF

<details>
<summary>Show Answer</summary>
<b>C. RAG with Amazon Bedrock Knowledge Bases</b>

RAG allows adding current information without model retraining by retrieving from external knowledge bases.
</details>

### Question 6
What is the purpose of Chain-of-Thought prompting?

A. To reduce token usage
B. To improve reasoning by showing step-by-step thinking
C. To generate multiple responses
D. To filter harmful content

<details>
<summary>Show Answer</summary>
<b>B. To improve reasoning by showing step-by-step thinking</b>

Chain-of-thought prompting guides the model through logical steps, improving performance on complex reasoning tasks.
</details>

---

## Key Takeaways

1. **Model Selection**: Consider cost, latency, modality, and customization needs
2. **Temperature**: Lower = consistent, Higher = creative
3. **RAG**: Best for current/domain knowledge without retraining
4. **Prompt Engineering**: Zero-shot → One-shot → Few-shot → Chain-of-thought
5. **Fine-tuning**: Use when you need domain-specific behavior
6. **Evaluation**: ROUGE for summaries, BLEU for translation
7. **Agents**: Use for multi-step, autonomous tasks

### Customization Decision Tree

```
Need current information?         → RAG
Need specific output style?       → Few-shot prompting
Need domain expertise?            → Fine-tuning
Need complex reasoning?           → Chain-of-thought
Need multi-step actions?          → Agents
```

---

**Previous Lesson**: [Domain 2: Fundamentals of Generative AI](/posts/aws-aif-c01-domain-2-fundamentals-genai/)

**Next Lesson**: [Domain 4: Guidelines for Responsible AI](/posts/aws-aif-c01-domain-4-responsible-ai/)

---

*Questions about Foundation Models and their applications? Leave a comment below!*
