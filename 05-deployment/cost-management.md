# Cost management

## Master your LLM costs

Track, per run:

- tokens in;
- tokens out;
- number of LLM calls;
- which model is used.

> **Caution:** Audio and video inputs are expensive; watch them closely.

## No useless LLM calls

- If an agent or LLM step does not call a tool and does not modify the state, question whether the LLM call is needed at all.
- A [semantic cache](../04-evaluation/evaluation-methods.md) removes repeat calls for similar queries.

Cost is also a [production metric](../06-operations/metrics-and-sre.md): cost per run.
