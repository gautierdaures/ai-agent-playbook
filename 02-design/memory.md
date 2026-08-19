# Memory & context engineering

Memory is one piece of a bigger job: **context engineering** — deciding which tokens the model sees on each turn of the [control loop](architecture.md). As models moved past clever prompt wording, the real craft became *curating the context*: system prompt, tool definitions, retrieved documents, working state, and history, all competing for a finite attention budget.

## The goal: the right information, not the most

A bigger context window is not a free lunch. Attention is limited, and past a point more tokens *hurt* — the model loses track and recall degrades, an effect known as **context rot**. Good context engineering maximises useful signal and minimises noise: include what this decision needs, and nothing else.

## What goes in the window each turn

Think of every turn's context as an assembled budget:

- **System prompt** — role, constraints, output contract ([agent anatomy & prompting](agent-anatomy-and-prompting.md)).
- **Tool definitions** — kept small and distinct ([tool design](tool-design.md)).
- **Working state** — the agent's beliefs/goals/intentions, held in a semi-structured object ([architecture](architecture.md)).
- **Retrieved knowledge** — pulled on demand, not pre-loaded ([RAG & tooling](../03-build/rag-and-tooling.md)).
- **Relevant history** — a summary or the pertinent slice, not the whole transcript.

## Two memory horizons

- **Short-term memory** — the current session, living in the context window and working state.
- **Long-term memory** — persistence across sessions: a vector store for semantic recall, and/or a structured store (the user profile, key facts) for exact recall.

## You don't need the whole history

Don't stuff the entire conversation into context. Keep what matters accessible and pull it in when needed:

- Keep a **user profile** and durable facts in structured storage.
- Store summaries or embeddings, and **retrieve on demand** via an API or history lookup.
- **Compact and prune** as the session grows — summarize older turns, drop what's no longer relevant. This is the mitigation for context overflow ([error handling](error-handling.md)).

## References

- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — curating the context window; the finite-attention / context-rot problem.
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — retrieval and memory as components you add only when needed.
