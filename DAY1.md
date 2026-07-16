# 📘 Azure AI Foundry Notes

## 1. Azure AI Foundry ⭐

### Definition

Azure AI Foundry is Microsoft's end-to-end platform for developing, deploying, and managing AI applications on Azure.

It provides everything needed to build AI applications in one place.

### It includes

* AI Models
* AI Agents
* AI Tools
* Knowledge Sources
* Azure AI Foundry Portal
* Azure AI Foundry SDK
* Resource & Project Management

### Real-world analogy

Think of Azure AI Foundry as a software development platform for AI, similar to how Visual Studio is a platform for software development.

---

## 2. Azure AI Foundry Portal

### Definition

The Azure AI Foundry Portal is the web-based graphical interface (GUI) used to develop and manage AI projects.

### What can you do?

* Browse AI models
* Deploy models
* Create AI agents
* Test prompts
* Configure tools
* Connect knowledge sources
* View API keys
* Manage projects

### Analogy

Like using GitHub's website instead of Git commands.

---

## 3. Azure AI Foundry SDK

### Definition

The Azure AI Foundry SDK lets developers interact with Azure AI Foundry through code.

### Example

```python
from azure.ai.projects import AIProjectClient
```

Instead of clicking buttons in the portal, you write code.

### What can it do?

* Connect to projects
* Create agents
* Manage models
* Deploy resources
* Automate tasks

### Analogy

* **Portal = Manual**
* **SDK = Programmatic**

---

## 4. Azure AI Foundry Resource ⭐

### Definition

A Foundry Resource is the parent Azure resource that provides the infrastructure for AI projects.

### It provides

* Compute
* Storage
* Networking
* Authentication
* AI Services

A resource can contain multiple projects.

### Example

```text
Azure Subscription
        │
Resource Group
        │
Foundry Resource
```

Think of it as the building that contains several offices (projects).

---

## 5. Azure AI Foundry Project ⭐

### Definition

A Project is a workspace for one AI application.

Everything related to a single AI application is stored inside the project.

---

# 6.Foundry IQ

## Definition

Foundry IQ provides a centralized knowledge connection for a project.

Instead of connecting an agent to many knowledge sources individually, Foundry IQ lets you create one MCP-based connection that can access multiple sources.

## Example

### Without Foundry IQ

```text
Agent
├── PDF
├── SQL
├── SharePoint
└── Blob Storage
```

### With Foundry IQ

```text
PDF
SQL
SharePoint
Blob Storage
     │
     ▼
 Foundry IQ
     │
     ▼
   Agent
```

## Benefits

* Centralized knowledge management
* Simplified integration
* Easier retrieval of information

# Model vs Agent vs Tool vs Knowledge

| Feature                   | **Model**                                                           | **Agent**                                                                | **Tool**                                                                | **Knowledge**                                                 |
| ------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Definition**            | An AI model (LLM) that understands prompts and generates responses. | An AI application that uses a model to reason, plan, and complete tasks. | A capability or service that an agent uses to perform specific actions. | External data that provides additional context to the agent.  |
| **Primary Purpose**       | Generate text, code, images, or other AI outputs.                   | Coordinate models, tools, and knowledge to solve user requests.          | Execute actions or retrieve information.                                | Supply relevant information for accurate responses.           |
| **Reasoning & Planning**  | Generates responses based on the prompt.                            | ✅ Plans and decides the steps needed to complete a task.                 | ❌ No                                                                    | ❌ No                                                          |
| **Uses External Systems** | ❌ No                                                                | ✅ Yes (through tools)                                                    | ✅ Yes (e.g., APIs, databases, web search)                               | ❌ No                                                          |
| **Stores Information**    | Only learned parameters from training.                              | ❌ No                                                                     | ❌ No                                                                    | ✅ Yes (documents, databases, files, etc.)                     |
| **Examples**              | GPT-4.1, GPT-4o, Phi, Llama                                         | HR Agent, Customer Support Agent, Travel Agent                           | Web Search, SQL API, Code Interpreter, Function Calling                 | PDFs, SharePoint, SQL Database, Blob Storage, Azure AI Search |

## Relationship

```text
                User
                  │
                  ▼
               Agent
                  │
      ┌───────────┼───────────┐
      │           │           │
      ▼           ▼           ▼
    Model       Tool     Knowledge
      │           │           │
      │     Performs Tasks    │
      │                       │
      └───────────┬───────────┘
                  ▼
          Generates Response
```

