# AI Agents & Microsoft Foundry Agent Service - Quick Notes

## What is an AI Agent?

An **AI Agent** is software that uses **Generative AI** to understand tasks, make decisions, and perform actions autonomously.

Unlike traditional applications that follow predefined rules, AI agents:
- Understand context
- Make decisions
- Use tools
- Perform actions to achieve goals

### Formula

```
LLM + Tools + Reasoning + Actions = AI Agent
```

---

# Why AI Agents?

AI agents improve productivity and business processes by automating tasks and making intelligent decisions.

## Benefits

- **Automation** – Handles repetitive tasks automatically.
- **Better Decision Making** – Analyzes data and provides recommendations.
- **Scalability** – Handles increased workload without additional manpower.
- **24/7 Availability** – Works continuously without interruption.

---

# Common AI Agent Use Cases

| Agent Type | Purpose | Example |
|------------|---------|---------|
| Personal Productivity | Scheduling, emails, documents | Microsoft 365 Copilot |
| Research Agent | Data collection and analysis | Market & Medical Research |
| Sales Agent | Lead generation and follow-ups | Sales Automation |
| Customer Service Agent | Customer support | AI Chatbots |
| Developer Agent | Code generation and debugging | GitHub Copilot |

---

# AI Agent Security Risks

| Risk | Description |
|------|-------------|
| Data Leakage | Sensitive information exposed |
| Prompt Injection | Malicious prompts override agent instructions |
| Unauthorized Access | Agent gains excessive permissions |
| Data Poisoning | Corrupted training/context data affects decisions |
| Supply Chain Vulnerabilities | Third-party tools/plugins become compromised |
| Over-Autonomous Actions | Agent performs unintended actions |
| Poor Logging | Difficult to trace agent activities |
| Model Inversion | Sensitive training data extracted from model outputs |

---

# Security Best Practices

- Implement **Role-Based Access Control (RBAC)**
- Follow the **Least Privilege Principle**
- Validate and filter prompts
- Use **Human-in-the-Loop** approval for sensitive actions
- Enable logging and auditing
- Regularly audit third-party integrations
- Continuously validate and retrain models

---

# Microsoft Foundry Agent Service

Microsoft Foundry Agent Service is a **fully managed platform** for building, deploying, and scaling AI agents without managing infrastructure.

## Advantages

- Minimal coding (can build agents in **< 50 lines of code**)
- Managed compute and storage
- Easy deployment
- Enterprise-grade security
- Automatic scaling

---

# Types of Agents

## 1. Declarative Agents (Most Common)

Configured rather than coded.

### Prompt-Based Agent
- Single AI agent
- Uses prompts, instructions, tools, and models
- Easiest way to build an AI agent

### Workflow Agent
- Multiple AI agents collaborate
- Defined using YAML
- Used for complex workflows

---

## 2. Hosted Agents

- Developed completely in code
- Containerized
- Hosted and managed by Microsoft Foundry

---

# Key Features of Microsoft Foundry Agent Service

## Automatic Tool Calling
Automatically invokes tools such as APIs, file search, or code execution.

## Managed Conversation State
Uses the **Responses API** to maintain conversation history automatically.

## Tool Catalog
Supports built-in tools like:
- Web Search
- File Search
- Code Interpreter
- Azure Services
- External APIs

## Model Selection
Choose different AI models based on:
- Cost
- Performance
- Accuracy

## Enterprise Security
Provides:
- Authentication
- Data privacy
- Content safety
- Compliance

## Flexible Storage
Supports:
- Microsoft-managed storage
- Azure Blob Storage

## Observability & Tracing
Monitor and debug agent execution in production.

---

# Quick Revision

## AI Agent

```
AI Agent = LLM + Tools + Reasoning + Actions
```

---

## Benefits

- Automation
- Better Decisions
- Scalability
- 24×7 Availability

---

## Common Examples

- Microsoft 365 Copilot
- GitHub Copilot
- Customer Support Bots
- Research Agents
- Sales Agents

---

## Security Risks

- Data Leakage
- Prompt Injection
- Unauthorized Access
- Data Poisoning
- Supply Chain Attacks
- Over-Autonomous Actions
- Poor Logging
- Model Inversion

---

## Security Practices

- RBAC
- Least Privilege
- Prompt Validation
- Human Approval
- Logging & Auditing
- Validate Third-Party Tools
- Continuous Model Monitoring

---

## Microsoft Foundry Agent Service

**Purpose:** Build, deploy, and scale AI agents with minimal code and managed infrastructure.

### Agent Types

- Prompt-Based Agent
- Workflow Agent
- Hosted Agent

---

## Key Features

- Automatic Tool Calling
- Responses API (Conversation State)
- Built-in Tool Catalog
- Multiple AI Models
- Enterprise Security
- Azure Storage Support
- Observability & Tracing

---

# Interview Questions

### What is an AI Agent?
Software that uses Generative AI to understand context, make decisions, and perform actions autonomously.

### Difference between Traditional Application and AI Agent?

| Traditional App | AI Agent |
|-----------------|----------|
| Rule-based | AI-driven |
| Fixed workflow | Dynamic decision making |
| No reasoning | Can reason |
| No autonomy | Autonomous |

### What are the benefits of AI Agents?
- Automation
- Better Decision Making
- Scalability
- 24/7 Availability

### Name three types of agents in Microsoft Foundry.
- Prompt-Based Agent
- Workflow Agent
- Hosted Agent

### What does the Responses API do?
Maintains conversation state automatically.

### Name some built-in tools in Foundry.
- Web Search
- File Search
- Code Execution
- Azure Services
- External APIs

### What are common security risks?
- Data Leakage
- Prompt Injection
- Unauthorized Access
- Data Poisoning

### How do you secure AI Agents?
- RBAC
- Least Privilege
- Prompt Validation
- Human Approval
- Logging & Auditing
