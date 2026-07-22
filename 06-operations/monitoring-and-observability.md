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

- **LangSmith** — dashboard, hosted in their cloud; a bit expensive; pairs naturally with LangChain.
- **LangFuse** — a good open-source alternative; based on OpenTelemetry; reportedly not very stable yet, as of the training.

Traceability requirements come from [rollout & production safety](../05-deployment/rollout-and-safety.md).
