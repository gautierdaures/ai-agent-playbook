# Frameworks & infrastructure

What to build the agent *with*: whether to use a framework at all, which one, and what sits around it to serve the agent as a system.

## You may not need a framework

A framework is worth its abstraction cost when you need its orchestration — durable state, checkpointing, hand-offs, streaming, retries. For a single agent with a handful of tools and a loop you control, the model provider's SDK plus your own `while` loop is often less code and far easier to debug, and it keeps the loop's behaviour explicit ([architecture](../02-design/architecture.md)).

Signals you want a framework: several agents to coordinate, runs that pause for a human, checkpointing and resume ([state & execution](state-and-execution.md)), or a team that benefits from a shared vocabulary. Signals you do not: one agent, three tools, and a clear spec.

Whichever way you go, keep the framework at the edges. The tools, prompts, and adapters should not import it — that is what makes swapping it, or dropping it, a contained change ([architecture — evolvability](../02-design/architecture.md)).

## Agent frameworks

There is no single best framework; they target different problems. Pick for explicit control, speed-to-prototype, or vendor affinity.

- **LangGraph** — orchestration as an explicit graph; precise control, state management, checkpointing, human-in-the-loop, streaming. Highest production readiness, pairs with LangSmith. Cons: more boilerplate, and graphs can be harder to read.
- **LangChain** — modular components and a large ecosystem of integrations; good glue for chains and tools. Cons: a bit verbose; the abstractions can get in the way for complex control flow (that is what LangGraph is for).
- **CrewAI** — role-based multi-agent setups with minimal code; fastest to a working multi-agent prototype. Cons: less low-level control for complex, stateful flows.
- **AutoGen / AG2** — frames multi-agent work as a conversation between agents (group chat). Strong for research and complex multi-agent dialogues. Cons: conversation model can be harder to make deterministic.
- **OpenAI Agents SDK** — lightweight, built-in tracing and guardrails, full streaming. Cons: tightest fit inside the OpenAI ecosystem.
- **Claude Agent SDK** — safety-first, native streaming, extended thinking; good production readiness. Cons: model affinity to Claude.
- **LlamaIndex** — best when retrieval is the central problem; streamlines data indexing and retrieval. Cons: less of an orchestration framework on its own.
- **Semantic Kernel / Microsoft Agent Framework** — natural pick when the stack is already Azure / .NET.

Rules of thumb: LangGraph for stateful production workflows where stakes are high; CrewAI when speed-to-prototype is the constraint; a lab SDK (OpenAI / Claude) when model affinity and tight vendor integration pay off; LlamaIndex when retrieval dominates.

### Choosing, in practice

Answer these in order and stop at the first that decides it:

1. **Does a run pause for a human, or need to survive a crash?** → you need durable state: LangGraph, Temporal-backed orchestration, or your own checkpointer ([state & execution](state-and-execution.md)).
2. **Is retrieval the hard part?** → LlamaIndex, or a plain pipeline you own ([RAG & tooling](rag-and-tooling.md)).
3. **Is the stack already Azure / .NET, or committed to one model vendor?** → Semantic Kernel, or that vendor's SDK.
4. **Is this a prototype whose job is to answer one question this month?** → whatever the team already knows; the prototype is meant to be thrown away ([scoping the first version](../01-framing/scoping-the-first-version.md)).
5. **None of the above** → the provider SDK and your own loop.

Then pin the version. Agent frameworks move fast and break interfaces between minor releases; an unpinned dependency turns into an unplanned migration in the middle of a sprint.

## API layer

**FastAPI is the default** for exposing an agent as a service, and the reasons are specific to agents rather than taste: runs are I/O-bound (model calls, tool calls), so `async` lets one worker hold many in-flight runs; `StreamingResponse` covers token streaming to a chat UI; Pydantic models are already the I/O contract from the agent spec, so the API schema comes free; and `BackgroundTasks` plus a task queue covers the submit-and-poll shape most agents need ([state & execution](state-and-execution.md)). Flask remains fine for a small synchronous service or an internal endpoint, but you will fight it as soon as runs get long or need streaming.

Around the API, expect to need: a **task queue and workers** (Celery, RQ, Arq, or a managed queue) for background runs, a **state store** (Redis or Postgres) for sessions and checkpoints, and a **vector store** if the design uses retrieval ([vector databases](vector-databases.md)).

## MCP

- Run an MCP server for tool discovery; think of it as client and server.
- See [architecture](../02-design/architecture.md) for where MCP sits in the layers, and [RAG & tooling](rag-and-tooling.md) for serving your own tools over it.
- It is also the cheapest hedge against framework lock-in: tools exposed over MCP are reachable from any client, whatever orchestrates the loop.

## References

- [LangChain — The best AI agent frameworks](https://www.langchain.com/resources/ai-agent-frameworks)
- [Turing — Comparison of top AI agent frameworks](https://www.turing.com/resources/ai-agent-frameworks)
- [Firecrawl — Best open source agent frameworks](https://www.firecrawl.dev/blog/best-open-source-agent-frameworks)
- [FastAPI documentation](https://fastapi.tiangolo.com/) — async endpoints, streaming responses, background tasks.
