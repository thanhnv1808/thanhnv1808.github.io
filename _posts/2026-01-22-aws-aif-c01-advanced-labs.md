---
title: "AWS AIF-C01 - Advanced Hands-On Labs Part 2"
author: thanhnv1808
date: 2026-01-22 18:00:00 +0700
categories: [AWS, AI Practitioner]
tags: [aws, ai-practitioner, aif-c01, advanced-labs, sagemaker, bedrock-agents]
description: Advanced hands-on labs for AWS AI Practitioner. Deep dive into Bedrock Agents, SageMaker, end-to-end projects, and more.
pin: false
comments: true
---

## Introduction

These advanced labs build on the foundational labs and provide deeper hands-on experience with AWS AI services.

> **Prerequisites**: Complete the basic labs first. Some labs may incur AWS charges.
{: .prompt-warning }

---

## Lab 11: Build an AI Agent with Amazon Bedrock Agents

### Objective
Create an autonomous AI agent that can execute multi-step tasks.

### Duration
60 minutes

### Use Case
Build a customer service agent that can:
- Look up order status
- Process returns
- Answer product questions

### Steps

#### Step 1: Create Lambda Functions for Agent Actions

**Order Status Function:**

1. Go to **AWS Lambda** console
2. Create function: `aif-lab-order-status`
3. Runtime: Python 3.12
4. Add code:

```python
import json

def lambda_handler(event, context):
    # Parse the agent's request
    agent = event.get('agent', {})
    action_group = event.get('actionGroup', '')
    api_path = event.get('apiPath', '')
    parameters = event.get('parameters', [])

    # Extract order_id from parameters
    order_id = None
    for param in parameters:
        if param.get('name') == 'order_id':
            order_id = param.get('value')

    # Mock order database
    orders = {
        "ORD001": {"status": "Shipped", "delivery": "Jan 25, 2026", "items": ["Laptop"]},
        "ORD002": {"status": "Processing", "delivery": "Jan 28, 2026", "items": ["Mouse", "Keyboard"]},
        "ORD003": {"status": "Delivered", "delivery": "Jan 20, 2026", "items": ["Monitor"]}
    }

    if order_id and order_id in orders:
        order = orders[order_id]
        response_body = {
            "application/json": {
                "body": json.dumps({
                    "order_id": order_id,
                    "status": order["status"],
                    "estimated_delivery": order["delivery"],
                    "items": order["items"]
                })
            }
        }
    else:
        response_body = {
            "application/json": {
                "body": json.dumps({"error": f"Order {order_id} not found"})
            }
        }

    return {
        "messageVersion": "1.0",
        "response": {
            "actionGroup": action_group,
            "apiPath": api_path,
            "httpMethod": "GET",
            "httpStatusCode": 200,
            "responseBody": response_body
        }
    }
```

**Return Request Function:**

1. Create function: `aif-lab-return-request`
2. Add code:

```python
import json
import uuid

def lambda_handler(event, context):
    parameters = event.get('parameters', [])

    order_id = None
    reason = None
    for param in parameters:
        if param.get('name') == 'order_id':
            order_id = param.get('value')
        if param.get('name') == 'reason':
            reason = param.get('value')

    # Generate return ID
    return_id = f"RET{uuid.uuid4().hex[:6].upper()}"

    response_body = {
        "application/json": {
            "body": json.dumps({
                "return_id": return_id,
                "order_id": order_id,
                "status": "Return Initiated",
                "reason": reason,
                "instructions": "Please ship item to our return center within 14 days."
            })
        }
    }

    return {
        "messageVersion": "1.0",
        "response": {
            "actionGroup": event.get('actionGroup', ''),
            "apiPath": event.get('apiPath', ''),
            "httpMethod": "POST",
            "httpStatusCode": 200,
            "responseBody": response_body
        }
    }
```

#### Step 2: Create the Agent

1. Go to **Amazon Bedrock** → **Agents**
2. Click **Create Agent**
3. Configure:
   - **Name**: `customer-service-agent`
   - **Description**: `AI agent for customer service tasks`
   - **Model**: Claude 3 Sonnet

4. **Agent Instructions** (System Prompt):
```
You are a helpful customer service agent for an e-commerce company.

Your capabilities:
1. Look up order status using order IDs (format: ORD001, ORD002, etc.)
2. Process return requests for orders
3. Answer general product and policy questions

Guidelines:
- Always be polite and professional
- Ask for order ID when needed
- Confirm details before processing returns
- If you cannot help, offer to connect with a human agent
```

#### Step 3: Create Action Group

1. Click **Add Action Group**
2. **Name**: `order-management`
3. **Description**: `Actions for managing orders`
4. **Action Type**: Define with API schemas

5. **OpenAPI Schema**:
```yaml
openapi: 3.0.0
info:
  title: Order Management API
  version: 1.0.0
paths:
  /order-status:
    get:
      operationId: getOrderStatus
      summary: Get the status of an order
      parameters:
        - name: order_id
          in: query
          required: true
          schema:
            type: string
          description: The order ID (e.g., ORD001)
      responses:
        '200':
          description: Order status retrieved successfully
  /return-request:
    post:
      operationId: createReturnRequest
      summary: Create a return request for an order
      parameters:
        - name: order_id
          in: query
          required: true
          schema:
            type: string
          description: The order ID to return
        - name: reason
          in: query
          required: true
          schema:
            type: string
          description: Reason for return
      responses:
        '200':
          description: Return request created successfully
```

6. Connect Lambda functions to actions

#### Step 4: Test the Agent

1. Click **Test** in the agent console
2. Try these conversations:

**Conversation 1: Order Status**
```
User: What's the status of my order ORD001?
Agent: [Calls getOrderStatus] Your order ORD001 is currently Shipped
       and estimated to arrive on Jan 25, 2026. It contains: Laptop.
```

**Conversation 2: Return Request**
```
User: I want to return order ORD003
Agent: I can help you with that return. Could you please tell me
       the reason for the return?
User: The monitor has a dead pixel
Agent: [Calls createReturnRequest] I've initiated a return for your order.
       Your return ID is RET2A3B4C. Please ship the item to our return
       center within 14 days.
```

### Lab Questions
1. How does the agent decide which action to call?
2. What happens if the agent needs information it doesn't have?
3. How would you add more capabilities to this agent?

---

## Lab 12: End-to-End RAG Application

### Objective
Build a complete RAG application with custom UI using Bedrock APIs.

### Duration
90 minutes

### Steps

#### Step 1: Set Up Project Structure

```bash
mkdir bedrock-rag-app
cd bedrock-rag-app
```

Create `requirements.txt`:
```
boto3>=1.34.0
streamlit>=1.30.0
langchain>=0.1.0
langchain-aws>=0.1.0
faiss-cpu>=1.7.0
```

#### Step 2: Create the RAG Application

Create `app.py`:

```python
import streamlit as st
import boto3
import json
from typing import List

# Initialize Bedrock client
bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    region_name='us-east-1'
)

# Sample knowledge base (in production, use Bedrock Knowledge Bases)
KNOWLEDGE_BASE = {
    "return_policy": """
    Return Policy:
    - Items can be returned within 30 days of purchase
    - Items must be in original packaging
    - Refunds are processed within 5-7 business days
    - Electronics have a 15-day return window
    - Sale items are final sale
    """,
    "shipping_info": """
    Shipping Information:
    - Standard shipping: 5-7 business days
    - Express shipping: 2-3 business days
    - Free shipping on orders over $50
    - We ship to all 50 US states
    - International shipping available for additional fee
    """,
    "product_warranty": """
    Product Warranty:
    - Electronics: 1-year manufacturer warranty
    - Furniture: 5-year limited warranty
    - Clothing: 90-day quality guarantee
    - Extended warranties available at checkout
    """
}

def retrieve_context(query: str) -> str:
    """Simple keyword-based retrieval (use Bedrock KB in production)"""
    query_lower = query.lower()
    contexts = []

    if any(word in query_lower for word in ['return', 'refund', 'exchange']):
        contexts.append(KNOWLEDGE_BASE['return_policy'])
    if any(word in query_lower for word in ['ship', 'delivery', 'shipping']):
        contexts.append(KNOWLEDGE_BASE['shipping_info'])
    if any(word in query_lower for word in ['warranty', 'guarantee']):
        contexts.append(KNOWLEDGE_BASE['product_warranty'])

    if not contexts:
        contexts = list(KNOWLEDGE_BASE.values())

    return "\n\n".join(contexts)

def generate_response(query: str, context: str) -> str:
    """Generate response using Claude via Bedrock"""

    prompt = f"""You are a helpful customer service assistant.
Answer the question based on the provided context.
If the answer is not in the context, say you don't have that information.

Context:
{context}

Question: {query}

Answer:"""

    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 500,
        "messages": [
            {"role": "user", "content": prompt}
        ],
        "temperature": 0.3
    })

    response = bedrock_runtime.invoke_model(
        modelId="anthropic.claude-3-sonnet-20240229-v1:0",
        body=body
    )

    response_body = json.loads(response['body'].read())
    return response_body['content'][0]['text']

# Streamlit UI
st.title("🤖 Customer Service Assistant")
st.caption("Powered by Amazon Bedrock")

# Initialize chat history
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display chat history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Chat input
if prompt := st.chat_input("Ask me anything about our store..."):
    # Add user message
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    # Generate response
    with st.chat_message("assistant"):
        with st.spinner("Thinking..."):
            context = retrieve_context(prompt)
            response = generate_response(prompt, context)
            st.markdown(response)

    # Add assistant message
    st.session_state.messages.append({"role": "assistant", "content": response})

# Sidebar
with st.sidebar:
    st.header("About")
    st.write("This is a RAG-powered customer service chatbot.")
    st.write("Topics I can help with:")
    st.write("- Return Policy")
    st.write("- Shipping Information")
    st.write("- Product Warranty")

    if st.button("Clear Chat"):
        st.session_state.messages = []
        st.rerun()
```

#### Step 3: Run the Application

```bash
pip install -r requirements.txt
streamlit run app.py
```

#### Step 4: Test the Application

Try these queries:
- "What is your return policy?"
- "How long does shipping take?"
- "Do you offer warranty on electronics?"
- "Can I return a sale item?"

### Lab Questions
1. How does RAG improve response accuracy?
2. What would you need to change for production use?
3. How would you add more knowledge sources?

---

## Lab 13: Amazon Personalize Recommendations

### Objective
Build a product recommendation system using Amazon Personalize.

### Duration
60 minutes

### Steps

#### Step 1: Prepare Sample Data

Create `interactions.csv`:
```csv
USER_ID,ITEM_ID,TIMESTAMP,EVENT_TYPE
user_1,item_101,1705900000,purchase
user_1,item_102,1705900100,view
user_1,item_103,1705900200,purchase
user_2,item_101,1705900300,view
user_2,item_104,1705900400,purchase
user_3,item_102,1705900500,purchase
user_3,item_103,1705900600,view
user_3,item_105,1705900700,purchase
```

Create `items.csv`:
```csv
ITEM_ID,CATEGORY,PRICE
item_101,Electronics,299.99
item_102,Electronics,149.99
item_103,Home,89.99
item_104,Home,199.99
item_105,Clothing,49.99
```

#### Step 2: Create Dataset Group

1. Go to **Amazon Personalize** console
2. Click **Create dataset group**
3. **Name**: `aif-lab-recommendations`
4. **Domain**: E-commerce

#### Step 3: Import Data

1. Upload `interactions.csv` to S3
2. Create **Interactions dataset**
3. Define schema:
```json
{
  "type": "record",
  "name": "Interactions",
  "namespace": "com.amazonaws.personalize.schema",
  "fields": [
    {"name": "USER_ID", "type": "string"},
    {"name": "ITEM_ID", "type": "string"},
    {"name": "TIMESTAMP", "type": "long"},
    {"name": "EVENT_TYPE", "type": "string"}
  ],
  "version": "1.0"
}
```

#### Step 4: Create Solution

1. Click **Create solution**
2. **Solution name**: `product-recommendations`
3. **Recipe**: `aws-user-personalization`
4. Wait for training (can take 30+ minutes)

#### Step 5: Create Campaign

1. Click **Create campaign**
2. **Campaign name**: `product-recs-campaign`
3. **Solution version**: Select latest
4. Wait for deployment

#### Step 6: Get Recommendations

Using AWS CLI:
```bash
aws personalize-runtime get-recommendations \
  --campaign-arn "arn:aws:personalize:region:account:campaign/product-recs-campaign" \
  --user-id "user_1" \
  --num-results 5
```

### Lab Questions
1. What data does Personalize need to work?
2. How would you measure recommendation quality?
3. What are the different Personalize recipes for?

---

## Lab 14: Amazon Forecast Time Series

### Objective
Create demand forecasts using Amazon Forecast.

### Duration
45 minutes

### Steps

#### Step 1: Prepare Time Series Data

Create `sales_data.csv`:
```csv
item_id,timestamp,target_value
product_A,2025-01-01,100
product_A,2025-01-02,120
product_A,2025-01-03,110
product_A,2025-01-04,150
product_A,2025-01-05,200
product_A,2025-01-06,180
product_A,2025-01-07,160
product_B,2025-01-01,50
product_B,2025-01-02,60
product_B,2025-01-03,55
product_B,2025-01-04,70
product_B,2025-01-05,90
product_B,2025-01-06,85
product_B,2025-01-07,75
```

#### Step 2: Create Dataset Group

1. Go to **Amazon Forecast** console
2. Click **Create dataset group**
3. **Name**: `aif-lab-forecast`
4. **Forecasting domain**: Retail

#### Step 3: Import Target Time Series

1. Upload `sales_data.csv` to S3
2. Create dataset with schema:
```json
{
  "Attributes": [
    {"AttributeName": "item_id", "AttributeType": "string"},
    {"AttributeName": "timestamp", "AttributeType": "timestamp"},
    {"AttributeName": "target_value", "AttributeType": "float"}
  ]
}
```

#### Step 4: Create Predictor (AutoML)

1. Click **Create predictor**
2. **Predictor name**: `sales-predictor`
3. **Forecast horizon**: 7 days
4. **Algorithm**: AutoML (let Forecast choose best)
5. Wait for training

#### Step 5: Generate Forecast

1. Click **Create forecast**
2. Select your predictor
3. Wait for forecast generation

#### Step 6: Query Forecast

```bash
aws forecastquery query-forecast \
  --forecast-arn "arn:aws:forecast:region:account:forecast/sales-forecast" \
  --filters '{"item_id":"product_A"}'
```

### Lab Questions
1. What additional data could improve forecasts?
2. How do you evaluate forecast accuracy?
3. When would you use Forecast vs. custom ML?

---

## Lab 15: Multi-Model Comparison with Bedrock

### Objective
Compare multiple Foundation Models for a specific use case.

### Duration
30 minutes

### Steps

#### Step 1: Create Comparison Script

Create `model_comparison.py`:

```python
import boto3
import json
import time

bedrock_runtime = boto3.client('bedrock-runtime', region_name='us-east-1')

# Test prompts
test_cases = [
    {
        "name": "Summarization",
        "prompt": "Summarize in 2 sentences: Amazon Web Services provides cloud computing services including compute, storage, and AI/ML capabilities. Founded in 2006, AWS has become the leading cloud provider with millions of customers worldwide."
    },
    {
        "name": "Code Generation",
        "prompt": "Write a Python function to check if a string is a palindrome."
    },
    {
        "name": "Reasoning",
        "prompt": "If all roses are flowers and some flowers fade quickly, can we conclude that some roses fade quickly? Explain your reasoning."
    }
]

# Models to compare
models = [
    {
        "id": "anthropic.claude-3-haiku-20240307-v1:0",
        "name": "Claude 3 Haiku",
        "format": "anthropic"
    },
    {
        "id": "anthropic.claude-3-sonnet-20240229-v1:0",
        "name": "Claude 3 Sonnet",
        "format": "anthropic"
    },
    {
        "id": "amazon.titan-text-express-v1",
        "name": "Titan Text Express",
        "format": "titan"
    }
]

def invoke_anthropic(model_id, prompt):
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 300,
        "messages": [{"role": "user", "content": prompt}],
        "temperature": 0.3
    })
    response = bedrock_runtime.invoke_model(modelId=model_id, body=body)
    result = json.loads(response['body'].read())
    return result['content'][0]['text']

def invoke_titan(model_id, prompt):
    body = json.dumps({
        "inputText": prompt,
        "textGenerationConfig": {
            "maxTokenCount": 300,
            "temperature": 0.3
        }
    })
    response = bedrock_runtime.invoke_model(modelId=model_id, body=body)
    result = json.loads(response['body'].read())
    return result['results'][0]['outputText']

def run_comparison():
    results = []

    for test in test_cases:
        print(f"\n{'='*60}")
        print(f"Test: {test['name']}")
        print(f"Prompt: {test['prompt'][:100]}...")
        print('='*60)

        for model in models:
            start_time = time.time()

            try:
                if model['format'] == 'anthropic':
                    response = invoke_anthropic(model['id'], test['prompt'])
                else:
                    response = invoke_titan(model['id'], test['prompt'])

                latency = time.time() - start_time

                print(f"\n--- {model['name']} ({latency:.2f}s) ---")
                print(response[:500])

                results.append({
                    "test": test['name'],
                    "model": model['name'],
                    "latency": latency,
                    "response_length": len(response)
                })
            except Exception as e:
                print(f"\n--- {model['name']} (ERROR) ---")
                print(str(e))

    return results

if __name__ == "__main__":
    results = run_comparison()

    print("\n\n" + "="*60)
    print("SUMMARY")
    print("="*60)
    for r in results:
        print(f"{r['test']} | {r['model']}: {r['latency']:.2f}s, {r['response_length']} chars")
```

#### Step 2: Run Comparison

```bash
python model_comparison.py
```

#### Step 3: Use Bedrock Model Evaluation

1. Go to **Bedrock** → **Model Evaluation**
2. Click **Create evaluation job**
3. Select models to compare
4. Choose evaluation type:
   - **Automatic**: Uses built-in metrics
   - **Human**: Custom human evaluation
5. Review results

### Lab Questions
1. Which model performed best for each task?
2. How do latency and quality trade off?
3. What factors would influence your model selection?

---

## Lab 16: Security Implementation for AI Workloads

### Objective
Implement security best practices for AI applications.

### Duration
45 minutes

### Steps

#### Step 1: Create IAM Role for Bedrock

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-3-sonnet*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": [
                "bedrock:InvokeModel"
            ],
            "Resource": [
                "arn:aws:bedrock:*::foundation-model/anthropic.claude-3-opus*"
            ]
        }
    ]
}
```

#### Step 2: Enable CloudTrail for Bedrock

```bash
aws cloudtrail create-trail \
  --name bedrock-audit-trail \
  --s3-bucket-name your-audit-bucket

aws cloudtrail start-logging \
  --name bedrock-audit-trail
```

#### Step 3: Create VPC Endpoint for Bedrock

1. Go to **VPC** → **Endpoints**
2. Create endpoint for `bedrock-runtime`
3. Select your VPC and subnets
4. Attach security group

#### Step 4: Implement PII Detection

```python
import boto3
import json

comprehend = boto3.client('comprehend')
bedrock = boto3.client('bedrock-runtime')

def detect_pii(text):
    """Check for PII before sending to model"""
    response = comprehend.detect_pii_entities(
        Text=text,
        LanguageCode='en'
    )
    return response['Entities']

def safe_invoke_model(prompt):
    """Invoke model only if no PII detected"""
    pii_entities = detect_pii(prompt)

    if pii_entities:
        pii_types = [e['Type'] for e in pii_entities]
        raise ValueError(f"PII detected: {pii_types}. Request blocked.")

    # Safe to proceed
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 500,
        "messages": [{"role": "user", "content": prompt}]
    })

    response = bedrock.invoke_model(
        modelId="anthropic.claude-3-sonnet-20240229-v1:0",
        body=body
    )

    return json.loads(response['body'].read())

# Test
try:
    # This should work
    result = safe_invoke_model("What is machine learning?")
    print("Success:", result['content'][0]['text'][:100])

    # This should be blocked
    result = safe_invoke_model("My email is john@example.com, can you help?")
except ValueError as e:
    print("Blocked:", e)
```

### Lab Questions
1. Why use least privilege for AI workloads?
2. What should you log for AI compliance?
3. How does VPC endpoint improve security?

---

## Cleanup

After completing labs, delete resources:

```bash
# Delete Lambda functions
aws lambda delete-function --function-name aif-lab-order-status
aws lambda delete-function --function-name aif-lab-return-request

# Delete Bedrock agents (via console)

# Delete Personalize resources
aws personalize delete-campaign --campaign-arn <campaign-arn>
aws personalize delete-solution --solution-arn <solution-arn>

# Delete Forecast resources
aws forecast delete-forecast --forecast-arn <forecast-arn>
aws forecast delete-predictor --predictor-arn <predictor-arn>
```

---

## Additional AWS Workshops

For more hands-on practice:

- [Amazon Bedrock Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/a4bdb007-5600-4368-81c5-ff5b4154f518)
- [Generative AI on AWS](https://catalog.us-east-1.prod.workshops.aws/workshops/80ae1ed2-f415-4d3d-9eb0-e9118c147bd4)
- [Amazon SageMaker Immersion Day](https://catalog.us-east-1.prod.workshops.aws/workshops/63069e26-921c-4ce1-9cc7-dd882ff62575)
- [Build with Amazon Q](https://catalog.us-east-1.prod.workshops.aws/workshops/896da20f-0c01-4756-8cc5-9fced8a4c45c)

---

**Back to Series**: [AWS AI Practitioner (AIF-C01) - Complete Study Guide](/posts/aws-ai-practitioner-series/)

---

*Questions about advanced labs? Leave a comment below!*
