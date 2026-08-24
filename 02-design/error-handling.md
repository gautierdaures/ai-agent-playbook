# Error handling

An agent is a non-deterministic system calling other systems, so failure is the normal case, not the exception. Design the failure modes in up front. Four are common; each has a matching mitigation.

## Hallucinations

The agent asserts something false or invents a result.

- **Ground it** — answer from retrieved sources rather than model memory ([RAG & tooling](../03-build/rag-and-tooling.md)), and have it cite what it used.
- **Require validated output** — enforce a schema (for example Pydantic) and reject anything that doesn't parse or fails a business check.

## Infinite loops

The agent keeps acting without converging — a real risk in [peer-to-peer topologies](multi-agent-orchestration.md).

- **Always have an absolute stop condition** — max iterations, a timeout, and a max-items cap. Make it a first-class part of the [control loop](architecture.md), not an afterthought; in production it is mandatory ([rollout & production safety](../05-deployment/rollout-and-safety.md)).
- **Break ties** — an expert debate (several agents arguing) can resolve a stuck decision; it doubles as an [evaluation technique](../04-evaluation/evaluation-methods.md).

## Tool errors

A tool call fails, times out, or returns something unexpected.

- **Retry with backoff, then fall back** — retry the transient failure; if it persists, degrade gracefully to an alternative path or a safe default.
- **Return actionable errors** — the tool's error message is the agent's only chance to self-correct, so make it specific ([tool design](tool-design.md)). A vague error turns a recoverable failure into a retry loop.
- **Fail closed on irreversible actions** — when in doubt about a side-effecting call, stop and escalate rather than guess ([human-in-the-loop](human-in-the-loop.md)).

## Context overflow

The conversation outgrows the window and the model starts losing the thread.

- **Summarize, prune, and retrieve on demand** — compact older turns and keep history accessible instead of resident ([memory & context engineering](memory.md)).

## Detecting failures in production

You can only handle what you can see. Journal every decision and tool result, and alert on the signals these failures produce — runaway step counts, retry storms, schema-rejection rates. See [monitoring & observability](../06-operations/monitoring-and-observability.md) and [metrics & SRE](../06-operations/metrics-and-sre.md).
