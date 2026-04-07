# Vessel-Equipment-Agent-Skills: A Four-Layer Agent Architecture

You stop debugging tangled agent logic at 2 a.m. This pattern fixes that.

A reference implementation used in production across the Cocapn Fleet. It runs on Cloudflare Workers and has zero dependencies.

**Live URL:** [https://the-fleet.casey-digennaro.workers.dev](https://the-fleet.casey-digennaro.workers.dev)

## Quick Start

1.  Fork this repository.
2.  Deploy directly to Cloudflare Workers.
3.  Edit only the layer you need to change. You never modify the runtime to adjust a prompt.

## Why This Exists

Our agent code repeatedly became unmanageable. A single prompt change could break deployment. A model update could break the data pipeline. This architecture enforces separation between components that should not interact, so you will not find infrastructure code mixed with business logic.

## The Four Layers

Each layer has a single responsibility. There are no cross-layer dependencies.

1.  **Vessel:** The runtime environment. It manages compute, memory, and deployment. This layer is unaware of the agent it runs.
2.  **Equipment:** Data and tools. This includes RAG pipelines, data parsers, and external API clients. It defines *what* the agent can access.
3.  **Agent:** The reasoning engine. It handles model selection, context window management, and core interaction logic. It has no knowledge of your specific use case.
4.  **Skills:** Behavior and prompt logic. It contains prompt templates, reasoning patterns, and output formatting rules. It defines *how* the agent thinks for a given task.

## What It Solves

*   **Independent Upgrades:** Swap your LLM provider or data pipeline without redeploying the entire application.
*   **Localized Debugging:** Issues are confined to a single layer. You spend less time tracing errors through the stack.
*   **Full Ownership:** This is not a framework. You fork and own every line of code. There is no library version lock-in.
*   **Portability:** The agent logic is decoupled from the Cloudflare Workers runtime, making it easier to move to another platform later.

## Limitations

The stateless nature of Cloudflare Workers means there is no built-in persistence between requests. If your agent requires long-term memory or session state, you must implement it externally using a database or KV store.

<div style="text-align:center;padding:16px;color:#64748b;font-size:.8rem"><a href="https://the-fleet.casey-digennaro.workers.dev" style="color:#64748b">The Fleet</a> · <a href="https://cocapn.ai" style="color:#64748b">Cocapn</a></div>