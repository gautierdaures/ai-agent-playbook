# Cost management

Cost is a live production concern, not a one-off estimate: token spend moves with traffic, model choice, and prompt changes, so it is watched and controlled continuously in operations. The build-time budget and TCO projection are set earlier (see [scoping the first version](../01-framing/scoping-the-first-version.md) and [feasibility & readiness](../01-framing/feasibility-and-readiness.md)); this note is about keeping the running system economical.

## Master your LLM costs

Track, per run:

- tokens in;
- tokens out;
- number of LLM calls;
- which model is used.

> **Caution:** Audio and video inputs are expensive; watch them closely.

## No useless LLM calls

- If an agent or LLM step does not call a tool and does not modify the state, question whether the LLM call is needed at all.
- Route easy steps to a cheaper model; reserve the frontier model for the steps that need it (see [architecture — evolvability](../02-design/architecture.md)).

## Semantic caching

A **semantic cache** returns the result of a similar previous query instead of recomputing it — matching on embedding similarity rather than an exact key. It cuts both latency and cost by removing repeat calls for near-duplicate queries. It is the highest-leverage single optimisation for a high-traffic agent with repetitive inputs.

Cost per run is also a [production metric](metrics-and-sre.md) — track it against the baseline cost per task from [measuring value](../01-framing/measuring-value.md).
