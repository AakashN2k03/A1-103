# Foundry Models Catalog

The **Foundry Models Catalog** is a library of AI models available in Azure AI Foundry.

It lets you:

* Browse available models
* Compare models
* Read model details
* Deploy models

## Examples of Models

* GPT-4.1
* GPT-4o
* Phi
* Llama
* Mistral
* Cohere

## Workflow

```text
Foundry Models Catalog
          │
          ▼
   Select a Model
          │
          ▼
    Deploy the Model
          │
          ▼
 Use the Deployed Model
 in your AI Application
```

# Azure AI Model Deployment & Playground

## Deployment Types

| Deployment Type            | Easy Explanation                                                                                                                          | Best For                                                                                                            | Real-world Analogy                                                                                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Global Standard**        | Microsoft hosts your model on its **global shared infrastructure**. Azure automatically routes your requests to available capacity.       | Most applications that need good scalability at a lower cost.                                                       | 🌍 Using a **global cloud service** like Gmail—you don't care which server responds, as long as it works. |
| **Standard**               | Your model is deployed in a **specific Azure region** (for example, East US or Central India). Your requests are served from that region. | Applications that require **data residency**, regional compliance, or lower latency for users in a specific region. | 🏢 Choosing a **local office** near your city instead of a global office.                                 |
| **Provisioned Throughput** | You **reserve dedicated processing capacity** for your model. That capacity is always available for your application.                     | High-volume, mission-critical applications that require consistent performance.                                     | 🚄 Reserving your own **private train** instead of taking a public train.                                 |

---

# Virtual Machine SKU

## Definition

**VM SKU** stands for **Virtual Machine Stock Keeping Unit**.

It defines the type of Azure VM used to host the model.

Think of it as selecting the hardware.

## Example

### Laptop Options

* i5 / 16 GB RAM
* i7 / 32 GB RAM
* Ryzen 9 / 64 GB RAM

Similarly, Azure offers different VM sizes.

---

# Instance Count (Sometimes Informally Called "Instance SKU")

## Definition

An **instance** is one running copy of your deployed model.

If you choose:

* **VM SKU:** `Standard_NC24`
* **Instances:** `3`

Azure creates:

```text
Instance 1
Instance 2
Instance 3
```

All three instances serve requests together.

## Why Multiple Instances?

To improve:

* Scalability
* High Availability
* Load Balancing

---

# What is a Playground?

## Definition

A **Playground** is an interactive testing environment where you can experiment with AI models **without writing any code**.

## It lets you

* Test prompts
* Compare different AI models
* Adjust model settings
* View responses
* Experiment before integrating the model into your application

---

# Playground Workflow

```text
Select Model
      │
      ▼
Enter Prompt
      │
      ▼
Adjust Parameters
(Temperature, Top-p, Max Tokens, etc.)
      │
      ▼
Generate Response
      │
      ▼
Evaluate & Iterate
```

