# Microsoft Azure AI Foundry – Endpoints, SDKs, Authentication & Responses API

This document provides an overview of Azure AI Foundry concepts including endpoints, SDKs, authentication methods, Chat APIs, and the modern Responses API.

---

# 1. Endpoints

An **endpoint** is the URL your application uses to communicate with Azure AI Foundry or Azure OpenAI.

Azure AI Foundry commonly uses two endpoints.

## A. Project Endpoint

**Example**

```text
https://myproject.services.ai.azure.com
```

Used with the **Azure AI Foundry SDK**.

Provides access to:

- AI Models
- AI Agents
- Connections
- Azure AI Search
- Evaluations
- Tracing
- Other project resources

Architecture

```text
Application
      │
      ▼
Project Endpoint
      │
      ▼
Azure AI Foundry Project
      │
 ┌────┼──────────────┐
 ▼    ▼              ▼
Models Agents Search Connections
```

Think of it as the **main entrance** to your entire Azure AI Foundry project.

---

## B. Azure OpenAI Endpoint

**Example**

```text
https://myopenai.openai.azure.com
```

Used with the **OpenAI SDK**.

Provides access to:

- GPT models
- Embedding models
- Chat models
- Image models (if deployed)

Architecture

```text
Application
      │
      ▼
Azure OpenAI Endpoint
      │
      ▼
Model Deployment
      │
      ▼
GPT-4.1
```

Think of it as a **direct entrance** to your AI model deployment.

---

## Endpoint Comparison

| Project Endpoint | Azure OpenAI Endpoint |
|-----------------|----------------------|
| Azure AI Foundry SDK | OpenAI SDK |
| Access to full Foundry project | Access to deployed models |
| Agents, Search, Evaluations, Connections | Mainly model inference |
| Best for enterprise AI apps | Best for chat applications |

---

# 2. Client SDKs

An **SDK (Software Development Kit)** provides libraries that simplify communication with Azure AI services.

## Azure AI Foundry SDK

Import

```python
from azure.ai.projects import AIProjectClient
```

Use it when working with:

- AI Agents
- Connections
- Azure AI Search
- Evaluations
- Tracing
- Project resources
- Model inference

Flow

```text
Python Application
        │
        ▼
Azure AI Foundry SDK
        │
        ▼
Project Endpoint
        │
        ▼
Azure AI Foundry Project
```

---

## OpenAI SDK

Import

```python
from openai import OpenAI
```

Use it for:

- Chat applications
- GPT models
- Embeddings
- Images
- Audio
- Text generation

Flow

```text
Python Application
        │
        ▼
OpenAI SDK
        │
        ▼
Azure OpenAI Endpoint
        │
        ▼
GPT Model
```

---

## Which SDK Should You Choose?

### Use the OpenAI SDK when

- Building chatbots
- Text generation
- Summarization
- Translation
- Embeddings

### Use the Azure AI Foundry SDK when

- Building AI Agents
- Using Azure AI Search
- Running Evaluations
- Using Tracing
- Managing Foundry resources

---

# 3. Authentication

Authentication verifies that your application has permission to access Azure AI resources.

## A. Microsoft Entra ID (Recommended)

Flow

```text
Application
      │
      ▼
Microsoft Entra ID
      │
      ▼
Access Token
      │
      ▼
Azure AI Foundry
```

Advantages

- Most secure
- No API keys
- Enterprise ready
- Recommended for production

---

## B. API Key Authentication

Example

```text
abc123xyz789
```

Flow

```text
Application
      │
      ▼
API Key
      │
      ▼
Azure AI
```

Advantages

- Easy to use
- Good for learning
- Ideal for prototypes

Disadvantage

- Must be kept secret

---

## C. Token Authentication

Flow

```text
Application
      │
      ▼
Access Token
      │
      ▼
Azure AI
```

Advantages

- Temporary credentials
- More secure than permanent API keys

---

## Authentication Comparison

| Method | Recommended | Best For |
|---------|-------------|----------|
| Microsoft Entra ID | ✅ Production | Enterprise applications |
| API Key | ✅ Learning | Testing and prototypes |
| Access Token | ✅ Temporary access | OAuth-based scenarios |

---

# 4. Chat APIs

After authentication, your application sends prompts to AI models.

---

## Chat Completions API

The traditional API for chat-based interactions.

Example

```python
response = client.chat.completions.create(
    model="gpt-4.1",
    messages=[
        {
            "role": "user",
            "content": "Explain AI"
        }
    ]
)
```

Flow

```text
User
   │
   ▼
Prompt
   │
   ▼
Chat Completions API
   │
   ▼
GPT Model
   │
   ▼
Response
```

---

## Responses API (Recommended)

The modern API for interacting with AI models.

Example

```python
response = client.responses.create(
    model="gpt-4.1",
    input="Explain AI"
)
```

Flow

```text
User
   │
   ▼
Prompt
   │
   ▼
Responses API
   │
   ▼
GPT Model
   │
   ▼
Response
```

---

## Chat API Comparison

| Chat Completions | Responses API |
|------------------|---------------|
| Older API | Modern API |
| Uses `messages` | Uses `input` |
| Widely supported | Recommended for new applications |
| Great for existing projects | Supports future AI capabilities |

---

# 5. Responses API

The **Responses API** is Microsoft's modern interface for interacting with AI models.

It provides a single API that supports multiple AI capabilities.

## Key Features

### Modern API

- Recommended for new development
- Replaces Chat Completions with a unified interface
- Supports Azure AI Foundry and Azure OpenAI models
- Compatible with OpenAI SDKs

---

### Stateful Conversations

Maintains conversation context across multiple interactions.

Benefits

- Multi-turn conversations
- Better user experience
- Less conversation history to manage
- Reduced need to resend previous messages

---

### Unified Experience

One API supports:

- Chat
- Tool calling
- Structured outputs
- Reasoning models
- Future AI capabilities

---

### Foundry Direct Models

Works with models hosted directly in the Azure AI Foundry Model Catalog, including:

- OpenAI
- Meta Llama
- Mistral
- Phi
- DeepSeek
- Other supported foundation models

---

### OpenAI-Compatible Integration

Uses the familiar OpenAI SDK.

```python
from openai import OpenAI
```

This makes migration from existing OpenAI applications straightforward.

---

### Creating Conversational Experiences

Supports natural conversations by:

- Maintaining context
- Understanding previous messages
- Producing coherent multi-turn interactions
- Powering chatbots and AI assistants

---

### Manual Conversation Chaining

Instead of using built-in conversation state, the application stores and resends the conversation history.

Advantages

- Complete control
- Works with every model

Disadvantages

- More code
- Higher token usage
- Larger requests

---

### Context Window

The maximum number of tokens a model can process in one request.

Includes

- System prompt
- Chat history
- Current user input
- Tool outputs
- Assistant responses

When the context window becomes full, applications typically:

- Remove older messages
- Summarize previous conversations
- Trim unnecessary context

---

### Async Usage

Asynchronous requests allow applications to continue running while waiting for AI responses.

Benefits

- Non-blocking execution
- Better scalability
- Concurrent AI requests
- Improved user experience

---

# Complete Architecture

```text
                    Your Application
                           │
                  Choose an SDK
                ┌──────────┴──────────┐
                │                     │
      Azure AI Foundry SDK      OpenAI SDK
                │                     │
        Project Endpoint      Azure OpenAI Endpoint
                │                     │
Authenticate (Entra ID / API Key / Access Token)
                           │
                           ▼
                  Model Deployment (GPT-4.1)
                           │
                   Choose an API
                ┌──────────┴──────────┐
                │                     │
      Chat Completions API     Responses API
                │                     │
                ▼                     ▼
          AI-generated Response
```

---
