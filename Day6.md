# Microsoft Foundry Agent Development

## Overview

Microsoft Foundry provides two primary approaches for developing AI agents:

1. **Microsoft Foundry Portal** – A visual, no-code environment for creating and managing AI agents.
2. **Visual Studio Code (Foundry Extension)** – A developer-focused, code-first environment with Git integration and local development capabilities.

Choose the approach that best fits your workflow and team requirements.

---

# Microsoft Foundry Portal

The **Microsoft Foundry Portal** is a web-based interface that allows you to create and manage AI agents without writing code.

## Features

- No-code development experience
- Quick prototyping
- Visual configuration of agents
- Centralized management across projects
- Collaboration with non-technical stakeholders
- Monitor:
  - Token usage
  - Latency
  - Evaluation metrics
- No installation required

---

# Visual Studio Code (Foundry Extension)

The **Microsoft Foundry VS Code Extension** provides a developer-centric experience for building AI agents alongside application code.

## Features

- Code-first development
- Git/Version Control integration
- YAML configuration editing
- Local development and debugging
- Integrated Agent Designer

### Resources

Manage Foundry resources directly from VS Code:

- Model Deployments
- AI Agents
- Connections
- Vector Stores

### Development Tools

- Model Catalog
- Model Playground
- Agent Playground
- Local Visualizer
- Deploy Hosted Agents

---

# When to Use

## Use Microsoft Foundry Portal

Choose the portal when you need:

- Quick prototyping
- Visual UI
- Centralized management
- Team collaboration
- No local setup

---

## Use Visual Studio Code

Choose VS Code when you need:

- Developer-centric workflows
- Application integration
- Git version control
- Local development
- Debugging support
- YAML-based configuration

---

# Typical Agent Development Workflow

1. Connect to a Microsoft Foundry Project
2. Create an AI Agent
3. Configure agent instructions
4. Add tools
5. Test using the Playground
6. Iterate and improve
7. Deploy the agent
8. Integrate it into your application

---

# Required Azure Resources

The following Azure resources are required:

- Microsoft Foundry Project
- Model Deployment
  - GPT-4.1
  - Claude Sonnet
  - Other supported foundation models

---

# Optional Azure Services

Depending on your solution, you may integrate:

| Azure Service | Purpose |
|--------------|---------|
| Azure AI Search | Knowledge retrieval |
| Azure Storage | Store files for agents |
| Azure Key Vault | Secure secrets and credentials |
| Azure Functions | Custom tools and business logic |

---

# Foundry Portal vs VS Code

| Feature | Foundry Portal | VS Code Extension |
|----------|---------------|------------------|
| Development Style | No-code | Code-first |
| Interface | Visual | Code Editor |
| Git Integration | ❌ | ✅ |
| Local Development | ❌ | ✅ |
| YAML Editing | ❌ | ✅ |
| Quick Prototyping | ✅ | ✅ |
| Team Collaboration | ✅ | Limited |
| Debugging | Basic | Advanced |
| Best For | Business users & rapid prototyping | Developers & production applications |

---

# One-Line Difference

**Microsoft Foundry Portal**
> Visual, no-code, and ideal for rapid prototyping.

**Visual Studio Code**
> Code-first, Git-integrated, and designed for developer workflows.

---

# Summary

- **Foundry Portal** is best for quickly building AI agents using a visual interface without writing code.
- **VS Code Extension** is ideal for developers who want full control, version management, local debugging, and seamless application integration.
- Both approaches follow the same agent development lifecycle and use the same Azure Foundry resources.
