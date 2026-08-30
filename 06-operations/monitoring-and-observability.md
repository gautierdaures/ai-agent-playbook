# Monitoring & observability

## What to monitor

Capture, per run or session:

- the model used;
- session duration;
- number of iterations;
- number of tool calls;
- the execution graph;
- which agents were used.

## Observability tooling

These platforms treat the LLM trace as the primary object — nested spans across agents, retrievers, and tools, with evaluation scores attached to production traffic.

**AI-native trace + eval platforms**

- **LangSmith** — deep tracing, debugging, and evaluation; pairs naturally with LangChain/LangGraph. Hosted in their cloud; a bit expensive.
- **Langfuse** — the most-used open-source option; tracing, evals, prompt management, metrics; model- and framework-agnostic with self-hosting. (Earlier notes flagged it as not very stable; it has matured and now supports OpenTelemetry export.)
- **Arize Phoenix** — source-available, built on OpenTelemetry; strong on evaluation and drift detection.
- **Braintrust** — evaluation-centric platform for iterating on prompts and scoring.

**AI gateways (proxy in front of the model)**

- **Helicone**, **Portkey**, **LiteLLM** — sit between app and provider, adding logging, caching, cost tracking, and routing with minimal code changes.

**Standards**

- Prefer tools built on **OpenTelemetry** (Phoenix, OpenLLMetry, and OTel export from Langfuse) to avoid vendor lock-in and reuse existing infra like SigNoz or Datadog.

Traceability requirements come from [rollout & production safety](../05-deployment/rollout-and-safety.md). What to *alert* on once these traces exist is [alerting & anomaly detection](alerting-and-anomaly-detection.md); scoring production traffic for quality is [online evaluation](../04-evaluation/online-evaluation.md).

## References

- [MarkTechPost — Top LLM observability & evaluation platforms in 2026](https://www.marktechpost.com/2026/08/09/top-llm-observability-and-evaluation-platforms-in-2026-langfuse-langsmith-braintrust-arize-and-more-compared/)
- [SigNoz — LLM observability tools](https://signoz.io/comparisons/llm-observability-tools/)
