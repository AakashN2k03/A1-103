# AI Agents

## What is an AI Agent?

An **AI Agent** is software that understands, decides, and performs tasks autonomously using **Generative AI**.

Unlike traditional programs (fixed rules), AI agents:
- Understand context
- Make decisions
- Take actions to achieve a goal

### Formula

```text
LLM + Tools + Reasoning + Actions = AI Agent
```

---

# Why AI Agents are Useful

- **Automation** – Performs repetitive tasks automatically.
- **Better Decision Making** – Analyzes data and provides recommendations.
- **Scalability** – Handles more work without increasing manpower.
- **24/7 Availability** – Works continuously.

---

# Common Use Cases

| Agent | Purpose | Example |
|--------|---------|---------|
| Personal Productivity | Schedule meetings, emails | Microsoft 365 Copilot |
| Research | Gather information | Market research, medical research |
| Sales | Lead generation, follow-ups | Sales automation |
| Customer Service | Answer customer queries | Chatbots |
| Developer | Code generation, bug fixes | GitHub Copilot |

---

# Security Risks

| Risk | Meaning |
|------|---------|
| Data Leakage | Sensitive data exposed |
| Prompt Injection | User tricks agent into ignoring instructions |
| Unauthorized Access | Agent gets permissions it shouldn't |
| Data Poisoning | Bad training/context data causes wrong outputs |
| Supply Chain Attack | External APIs/plugins become compromised |
| Over-Autonomy | Agent performs unintended actions |
| Poor Logging | Difficult to trace agent actions |
| Model Inversion | Attackers extract sensitive training information |

---

# Security Best Practices

- RBAC (Role-Based Access Control)
- Least Privilege
- Prompt Validation
- Human-in-the-loop approval for sensitive actions
- Logging & Auditing
- Validate third-party plugins
- Monitor and retrain models

---

# Microsoft Foundry Agent Service

## Definition

A **fully managed Microsoft service** for building, deploying, and scaling AI agents without managing infrastructure.

### Main Advantages

- Less coding (agents can be built in under 50 lines of code)
- Microsoft manages compute and storage
- Easy deployment
- Enterprise security

---

# Types of Agents in Microsoft Foundry

## 1. Declarative Agents

Configured rather than coded.

### Prompt-based Agent
- Single agent with prompts, tools, and instructions.
- Most common type.

### Workflow Agent
- Multiple agents collaborating.
- Defined in YAML.

## 2. Hosted Agents

- Built in code.
- Packaged as containers.
- Hosted and scaled by Microsoft Foundry.

---

# Key Features of Foundry Agent Service

- **Automatic Tool Calling** – Agent automatically invokes tools.
- **Managed Conversation State** – Responses API handles conversation history.
- **Built-in Tool Catalog** – File Search, Web Search, Code Execution, Azure integrations, APIs.
- **Multiple Model Support** – Choose models based on cost and performance.
- **Enterprise Security** – Authentication, privacy, and content safety.
- **Flexible Storage** – Microsoft-managed or Azure Blob Storage.
- **Observability & Tracing** – Monitor, debug, and optimize agent behavior.
