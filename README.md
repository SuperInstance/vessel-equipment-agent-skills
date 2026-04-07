# Vessel, Equipment, Agent, Skills
Cocapn Fleet Reference Architecture

A practical pattern for structuring agent systems, built and validated across the Cocapn Fleet. This model enforces a clean separation between infrastructure, data, models, and prompt logic to improve maintainability and debugging.

This is the four-layer model used across agents in the Cocapn Fleet. It is a production pattern for open-source agents running on platforms like Cloudflare Workers. Each layer has a distinct responsibility to prevent common architectural mistakes.

---

## Why this pattern exists

Many agent implementations conflate infrastructure, data pipelines, model choices, and prompting strategies. This leads to fragile systems where changing a model requires rewriting data logic, or scaling infrastructure breaks prompt templates. This model assigns a single, clear responsibility to each layer.

---

## The Four-Layer Model

A functional agent system is composed of four distinct layers. Their separation is the primary guard against complexity.

```
┌─────────────────────────────────────────────────────┐
│  VESSEL — The runtime environment                   │
│  "A Cloudflare Worker with a 10ms CPU limit"        │
│  "A Docker container on a Jetson Orin"              │
│  "An AWS Lambda function in us-east-1"              │
│  Defines compute, memory, network, and time limits. │
│  Exists independently of the agent it hosts.        │
├─────────────────────────────────────────────────────┤
│  EQUIPMENT — Data and tools outside the model       │
│  RAG pipelines, vector databases, file parsers      │
│  Structured folders of JSON, markdown, or SQLite    │
│  Filters and prepares input before the model sees it│
│  Determines WHAT information the agent can access.  │
├─────────────────────────────────────────────────────┤
│  AGENT — The model and its context architecture     │
│  "Claude-3.5-Sonnet with a 200K context window"     │
│  "Qwen2.5-32B with sliding window attention"        │
│  The model is the agent's core; the context         │
│  architecture defines how it remembers and reasons. │
├─────────────────────────────────────────────────────┤
│  SKILLS — Modifiers on the context architecture     │
│  Prompt templates, chain-of-thought, routing logic  │
│  System prompts and few-shot examples               │
│  Determines HOW the agent structures its reasoning. │
└─────────────────────────────────────────────────────┘
```

---

## 1. Vessel — The Runtime

A vessel is the objective runtime environment. Its constraints are real and must be accounted for.

| Property          | Example Constraint                          |
|-------------------|---------------------------------------------|
| **Compute Time**  | Cloudflare Worker (10ms CPU free / 30s paid)|
| **Memory**        | AWS Lambda (128MB - 10GB)                   |
| **Network**       | Edge location (latency, regional egress)    |
| **Persistence**   | Ephemeral or disk-based (KV, R2, /tmp)      |

**Principle:** Choose a vessel that fits your agent's resource profile. Do not force a large, stateful agent into a serverless function.

---

## 2. Equipment — Data and Tools

Equipment is everything that processes information before it reaches the agent.

*   **Data Loaders:** Parse PDFs, HTML, or APIs into clean text.
*   **Retrieval Systems:** Vector stores, keyword search, or graph databases.
*   **State Managers:** CRDTs for sync or simple KV for session state.
*   **Pre-processors:** Filtering, summarization, or validation logic.

**Principle:** Equipment should be model-agnostic. Swapping from GPT-4 to Claude should not require rewriting your RAG pipeline.

---

## 3. Agent — The Model and Context

The agent is the language model combined with its specific context management strategy.

*   **Model:** The actual LLM (e.g., `deepseek-coder`, `llama-3.1-70b`).
*   **Context Architecture:** How history is maintained (e.g., sliding window, summary-based, fixed buffer).
*   **Identity:** The agent's name and core purpose, expressed in its base system prompt.

**Principle:** The agent's identity and context strategy are coupled to the model's capabilities. A 7B model has a different context strategy than a 200K context model.

---

## 4. Skills — Reasoning Modifiers

Skills are reusable templates and techniques that shape the agent's reasoning for specific tasks.

*   **Prompt Templates:** Structured instructions for tasks like "analyze this log" or "write a summary."
*   **Reasoning Frameworks:** Chain-of-thought, reflection, or debate patterns.
*   **Routing Rules:** Logic to select which sub-task or tool to use next.

**Principle:** Skills are portable. A "code review" skill should work across different codebase-equipped agents.

---

## Quick Start

This repository is a reference architecture, not a framework. To apply it:

1.  **Define your Vessel:** Start with a simple Cloudflare Worker or a FastAPI server.
2.  **Add Equipment:** Connect a vector database or load a directory of documentation.
3.  **Choose an Agent:** Select a model (e.g., via OpenAI-compatible API) and a basic context window.
4.  **Implement a Skill:** Write a single prompt template for a clear task, like "answer questions based on the provided context."

A basic implementation might be 150 lines of code: a server (Vessel), a function to load documents (Equipment), an API call to an LLM (Agent), and a formatted prompt (Skill).

---

## Limitations

This pattern adds upfront structural discipline, which can feel like overhead for very small, throwaway prototypes. Its value becomes clear when maintaining or scaling a system beyond a single script.

---

## Attribution

This architectural model was developed and documented by **Superinstance & Lucineer (DiGennaro et al.)** as part of the Cocapn Fleet.

---

<div align="center">
  <sub>Part of the <a href="https://the-fleet.casey-digennaro.workers.dev">Cocapn Fleet</a> • <a href="https://cocapn.ai">Cocapn.ai</a></sub>
</div>