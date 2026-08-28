# Frameworks & infrastructure

What to build the agent *with*: whether to use a framework at all, which one, and what sits around it to serve the agent as a system.

## You may not need a framework

A framework earns its abstraction cost when you need its orchestration — durable state, checkpointing, hand-offs, retries. For one agent, a few tools, and a loop you control, the provider SDK and your own `while` loop is less code and easier to debug ([architecture](../02-design/architecture.md)).

Either way, keep it at the edges: tools, prompts, and adapters must not import it. That is what makes dropping it a contained change ([evolvability](../02-design/architecture.md)).

## Agent frameworks

Any roster dates fast; the axes don't.

**Control model** — who decides the next step:

| Control model | How the next step is chosen | Examples |
| --- | --- | --- |
| **Explicit graph** | You declare nodes and edges; the framework runs the state machine | LangGraph, Microsoft Agent Framework |
| **Model-driven loop** | The model picks tools until it stops; the framework does the plumbing | OpenAI Agents SDK, Claude Agent SDK, Strands Agents, Pydantic AI, Agno, Smolagents |
| **Role-based / conversational** | Agents with roles hand off or talk to each other | CrewAI, AutoGen / AG2 |
| **Retrieval-centric** | The indexing/retrieval pipeline is the product; the loop is thin | LlamaIndex, Haystack |
| **Compiled / optimised** | You declare the input/output signature and a metric; the framework *generates and tunes the prompt* rather than you hand-writing it | DSPy |

**Google ADK** spans the first two rows: `LlmAgent` is a model-driven loop, `Sequential`/`Parallel`/`Loop` workflow agents are explicit composition, nested into hierarchies. Gemini/Vertex-optimised, not Gemini-only (LiteLLM); the real differentiator is Agent Engine as a managed runtime plus first-class A2A.

**DSPy** is a different bet rather than a peer: prompts stop being files you edit and become artefacts compiled against a metric, which changes [prompt versioning](rag-and-tooling.md) and evals.

### The LangChain family stacks, it doesn't compete

- **LangGraph** — the runtime: graph execution, persistence, checkpointing, HITL interrupts.
- **LangChain** — above it: model/tool abstractions, integrations, and `create_agent`, a minimal harness running on LangGraph.
- **Deep Agents** (`deepagents`) — above both: planning, filesystem context management, subagents.

The question is which layer you own. LangGraph costs boilerplate and buys control; `create_deep_agent` costs visibility into the loop and buys a working agent. Note that `LLMChain` and friends are not what "LangChain" means in current versions — most tutorial material still shows them.

Axes cutting across the table:

- **Durable state** — can a run checkpoint, pause, and resume in a *different process*? Few do this natively (LangGraph's checkpointers being the exception), and the gap is the most common reason a prototype gets rebuilt ([state & execution](state-and-execution.md)).
- **Language** — Python for most; TypeScript has Mastra, the Vercel AI SDK, LangGraph.js.
- **Model coupling** — agnostic (LangGraph, CrewAI, Pydantic AI) vs. optimised for a provider or cloud (Claude Agent SDK, OpenAI Agents SDK, ADK for Gemini/GCP, Strands for Bedrock, Microsoft Agent Framework — successor to Semantic Kernel — for Azure/.NET). Optimised rarely means locked: what binds you is the deployment and observability story, not the model call.
- **Typing** — whether the spec's I/O contract is enforced at runtime or is a docstring. Pydantic AI is built on this; others bolt it on.
- **What comes free** — tracing, evals, guardrails, deployment. Often the actual reason to adopt one, and the thing comparison posts omit.

## Choosing, in practice

Two framing errors do most of the damage:

**Durability is orthogonal to framework choice.** "My run pauses for a human" selects a *persistence strategy*, not a framework — a checkpointer, a workflow engine underneath any framework (Temporal, Restate, Inngest), or your own state table. Conflating the two is how teams adopt a heavyweight framework for one feature.

**Retrieval quality is not a framework property.** Chunking, hybrid retrieval, and reranking decide it, under any of them ([RAG & tooling](rag-and-tooling.md)). "Retrieval is the hard part" argues for owning the pipeline, not for the best-marketed loader set.

Defaults:

- **Deterministic step order, high stakes, needs to resume** → an explicit graph plus real checkpointing.
- **A genuinely open-ended loop over a tool set** → a model-driven SDK; the graph buys you nothing when the edges are "whatever the model decides".
- **Already committed to a cloud or model vendor** → their SDK, for the integration and support story rather than the abstractions.
- **A prototype answering one question this month** → whatever the team already knows; it is meant to be thrown away ([scoping the first version](../01-framing/scoping-the-first-version.md)).
- **One agent, a few tools, a clear spec** → the provider SDK and your own loop.

Pin the version — these break interfaces between minor releases. And the cost of reversing a wrong choice is set entirely by how far the framework's types leaked past the edges.

## API layer

**FastAPI is the default**, for agent-specific reasons: runs are I/O-bound, so `async` holds many in flight per worker; `StreamingResponse` covers token streaming; the spec's Pydantic models are already the schema; `BackgroundTasks` plus a queue covers submit-and-poll ([state & execution](state-and-execution.md)). Flask is fine until runs get long or need streaming.

Around it: a **task queue and workers** (Celery, RQ, Arq, managed) for background runs, a **state store** (Redis, Postgres) for sessions and checkpoints, a **vector store** if the design retrieves ([vector databases](vector-databases.md)).

### Talking to the front end

Transport follows from the execution shape ([state & execution](state-and-execution.md)); picking it late is expensive because it reaches into the UI.

| Mode | Fits | Notes |
| --- | --- | --- |
| **Plain request/response** | Sub-10 s runs, server-to-server callers | No streaming machinery. Watch the proxy/load-balancer idle timeout — commonly 30–60 s |
| **SSE** (`text/event-stream`) | Token streaming to a chat UI — the default | One-way, plain HTTP, passes proxies, and the browser reconnects on drop sending `Last-Event-ID`. Assign an id per event so resume is possible |
| **WebSocket** | Genuinely bidirectional: barge-in, live cancellation, collaborative or voice UIs | Full duplex, but you now own reconnection, backpressure, and auth on a long-lived connection. Do not take this on just to stream tokens |
| **Submit + poll** | Multi-minute runs, anything with a human gate | `202` with a run id; the client polls status. Survives client disconnects and page reloads, which the streaming modes do not |
| **Webhook / push** | Server-to-server, or notifying a user who has closed the tab | Needs an endpoint, retries, and signature verification on the receiving side |

Two consequences worth deciding up front:

- **SSE reconnection is not run resumption.** `Last-Event-ID` re-attaches the browser to a stream; it does nothing if the run died with the process. Resumability comes from the checkpoint plus a replayable cursor — the transport only carries it. MCP dropped `Last-Event-ID` in its 2026-07-28 revision for this reason.
- **Long runs need two channels.** A run id from a `POST` that returns immediately, plus a stream or poll against that id. The single-long-request design dies on the first page reload — guaranteed if there is a human gate.

Buffering is the recurring bug: an intermediary that buffers `text/event-stream` turns streaming into a pause then a dump. Set `X-Accel-Buffering: no` (or the CDN equivalent) and verify through the real path, not the dev server.

## MCP

- Run an MCP server for tool discovery; think of it as client and server.
- See [architecture](../02-design/architecture.md) for where MCP sits in the layers, and [RAG & tooling](rag-and-tooling.md) for serving your own tools over it.
- It is also the cheapest hedge against framework lock-in: tools exposed over MCP are reachable from any client, whatever orchestrates the loop.

**MCP transport is a separate axis** from the table above: that one is how users reach your app, this is how your loop reaches its tools. Two bindings:

- **stdio** — newline-delimited JSON-RPC over a subprocess's standard streams. Local, single-user.
- **Streamable HTTP** — every message is a POST to one endpoint (conventionally `/mcp`); the server answers with `application/json` or a request-scoped SSE stream.

"Streamable HTTP" is an MCP term, not a web standard — a binding over ordinary HTTP, not a new wire protocol. What MCP adds is the JSON-RPC schema plus body fields mirrored into headers (`MCP-Protocol-Version`, `Mcp-Method`, `Mcp-Name`) so intermediaries can route without parsing bodies.

Two version traps: the **HTTP+SSE** transport (separate POST and GET-SSE endpoints, 2024-11-05) has been deprecated since 2025-03-26 and is no longer a defined binding — a guide pairing a long-lived `/sse` with `/messages` predates it. And revision **2026-07-28 removed sessions (`Mcp-Session-Id`), the GET stream, and `Last-Event-ID` resumability**, so requests are self-contained and need no sticky routing; much published material still describes the session-based shape.

## References

- [LangChain — The best AI agent frameworks](https://www.langchain.com/resources/ai-agent-frameworks)
- [LangChain — Deep Agents vs LangChain vs LangGraph](https://www.langchain.com/blog/deep-agents-vs-langchain-vs-langgraph) — which layer of that stack to build on.
- [Google — Agent Development Kit](https://google.github.io/adk-docs/) — workflow agents, multi-agent hierarchies, and the Agent Engine deployment path.
- [DSPy](https://dspy.ai/) — programming, not prompting: signatures, modules, and compiling against a metric.
- [Langfuse — Comparing open-source AI agent frameworks](https://langfuse.com/blog/2025-03-19-ai-agent-comparison) — a comparison written from tracing real runs rather than from docs.
- [Firecrawl — Best open source agent frameworks](https://www.firecrawl.dev/blog/best-open-source-agent-frameworks)
- [FastAPI documentation](https://fastapi.tiangolo.com/) — async endpoints, streaming responses, background tasks.
- [MDN — Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) — the event stream format, `Last-Event-ID`, and reconnection behaviour.
- [MCP specification — transports](https://modelcontextprotocol.io/specification/latest/basic/transports) — stdio and Streamable HTTP, and the compatibility rules for older revisions.
