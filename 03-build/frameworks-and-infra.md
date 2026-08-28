# Frameworks & infrastructure

What to build the agent *with*: whether to use a framework at all, which one, and what sits around it to serve the agent as a system.

## You may not need a framework

A framework is worth its abstraction cost when you need its orchestration — durable state, checkpointing, hand-offs, streaming, retries. For a single agent with a handful of tools and a loop you control, the model provider's SDK plus your own `while` loop is often less code and far easier to debug, and it keeps the loop's behaviour explicit ([architecture](../02-design/architecture.md)).

Signals you want a framework: several agents to coordinate, runs that pause for a human, checkpointing and resume ([state & execution](state-and-execution.md)), or a team that benefits from a shared vocabulary. Signals you do not: one agent, three tools, and a clear spec.

Whichever way you go, keep the framework at the edges. The tools, prompts, and adapters should not import it — that is what makes swapping it, or dropping it, a contained change ([architecture — evolvability](../02-design/architecture.md)).

## Agent frameworks

The landscape is large and turns over fast, so any roster is a snapshot. The **axes** below are the durable part; the names are examples, not a shortlist.

**Control model** — who decides the next step. This is the axis you actually feel every day:

| Control model | How the next step is chosen | Examples |
| --- | --- | --- |
| **Explicit graph** | You declare nodes and edges; the framework runs the state machine | LangGraph, Microsoft Agent Framework |
| **Model-driven loop** | The model picks tools until it stops; the framework does the plumbing | OpenAI Agents SDK, Claude Agent SDK, Strands Agents, Pydantic AI, Agno, Smolagents |
| **Role-based / conversational** | Agents with roles hand off or talk to each other | CrewAI, AutoGen / AG2 |
| **Retrieval-centric** | The indexing/retrieval pipeline is the product; the loop is thin | LlamaIndex, Haystack |
| **Compiled / optimised** | You declare the input/output signature and a metric; the framework *generates and tunes the prompt* rather than you hand-writing it | DSPy |

Two qualifications on that table. **Google ADK** deliberately spans the first two rows: `LlmAgent` is a model-driven loop, while its `Sequential`/`Parallel`/`Loop` workflow agents are explicit composition, and you nest them into a hierarchy. It is Gemini- and Vertex-optimised but not Gemini-only (other providers via LiteLLM), and its distinguishing feature is the deployment path — Vertex AI Agent Engine as a managed runtime — plus first-class A2A for cross-framework agent-to-agent calls. And **DSPy** is less an alternative to the others than a different bet: if you adopt it, prompts stop being files you edit and become artefacts compiled against a metric, which changes how [prompt versioning](rag-and-tooling.md) and evals work.

### The LangChain family is a stack, not three competitors

Worth separating, because "LangGraph vs LangChain vs Deep Agents" is a comparison people make and it is the wrong axis — they sit on top of each other:

- **LangGraph** — the low-level runtime: graph execution, persistence, checkpointing, human-in-the-loop interrupts. This is the part that matters for production and the reason to be in this family at all.
- **LangChain** — the application layer above it: model/tool abstractions, integrations, and `create_agent`, a minimal agent harness that itself runs on LangGraph.
- **Deep Agents** (`deepagents`) — a batteries-included harness above both, adding planning, filesystem-based context management, and subagents for long-horizon tasks.

So the real question is *which layer you want to own*, not which library to back. Dropping to LangGraph gives you control and costs boilerplate; taking `create_agent` or `create_deep_agent` gives you a working agent and costs you visibility into the loop. Note also that LangChain's older chain abstractions (`LLMChain` and friends) are not what "LangChain" means in current versions — a lot of tutorial material still shows them.

The other axes cut across that table:

- **Durable state** — can a run checkpoint, pause, and resume in a *different process*? Few frameworks do this natively (LangGraph's checkpointers being the notable one). Most do not, which is the single most common reason a prototype has to be rebuilt ([state & execution](state-and-execution.md)).
- **Language** — Python for most of the above; TypeScript teams have Mastra, the Vercel AI SDK, and LangGraph.js. Do not pick a Python framework for a TypeScript product to get a feature you could write in a day.
- **Model coupling** — model-agnostic (LangGraph, CrewAI, Pydantic AI) versus optimised for one provider or cloud (Claude Agent SDK, OpenAI Agents SDK, Google ADK for Gemini/GCP, Strands for Bedrock, Microsoft Agent Framework — the successor to Semantic Kernel — for Azure/.NET). "Optimised for" rarely means "locked to": most of these reach other providers through an adapter. The coupling that actually binds you is the deployment and observability story, not the model call.
- **Typing and validation** — whether the I/O contract from the agent spec is enforced at runtime or is a docstring. Pydantic AI is built around this; others bolt it on.
- **What it gives you for free** — tracing, evals, guardrails, and a deployment story. Often the real reason to adopt one, and the thing most comparison posts omit.

## Choosing, in practice

Two warnings about how this decision is usually made, because both produce regret:

**Durability is orthogonal to framework choice.** "My run pauses for a human" does not by itself select a framework — it selects a *persistence strategy*. You can get it from a framework's checkpointer, from a workflow engine underneath any framework (Temporal, Restate, Inngest), or from your own state table. Deciding these together is what leads teams to adopt a heavyweight framework for one feature.

**Retrieval quality is not a framework property.** Chunking, hybrid retrieval, and reranking decide it, and you can run that pipeline under any of them ([RAG & tooling](rag-and-tooling.md)). "Retrieval is the hard part" argues for owning the pipeline, not for adopting the framework with the best-marketed loader set.

With those separated, the choice is mostly about control model and ergonomics, and the honest defaults are short:

- **Deterministic step order, high stakes, needs to resume** → an explicit graph plus real checkpointing.
- **A genuinely open-ended loop over a tool set** → a model-driven SDK; the graph buys you nothing when the edges are "whatever the model decides".
- **Already committed to a cloud or model vendor** → their SDK, for the integration and support story rather than the abstractions.
- **A prototype answering one question this month** → whatever the team already knows; it is meant to be thrown away ([scoping the first version](../01-framing/scoping-the-first-version.md)).
- **One agent, a few tools, a clear spec** → the provider SDK and your own loop.

Then two things that matter more than which one you picked:

- **Pin the version.** Agent frameworks break interfaces between minor releases; an unpinned dependency becomes an unplanned migration mid-sprint.
- **Keep it at the edges** — see above. If the choice turns out wrong, the cost of reversing it is set entirely by how far the framework's types leaked into your tools, prompts, and adapters.

## API layer

**FastAPI is the default** for exposing an agent as a service, and the reasons are specific to agents rather than taste: runs are I/O-bound (model calls, tool calls), so `async` lets one worker hold many in-flight runs; `StreamingResponse` covers token streaming to a chat UI; Pydantic models are already the I/O contract from the agent spec, so the API schema comes free; and `BackgroundTasks` plus a task queue covers the submit-and-poll shape most agents need ([state & execution](state-and-execution.md)). Flask remains fine for a small synchronous service or an internal endpoint, but you will fight it as soon as runs get long or need streaming.

Around the API, expect to need: a **task queue and workers** (Celery, RQ, Arq, or a managed queue) for background runs, a **state store** (Redis or Postgres) for sessions and checkpoints, and a **vector store** if the design uses retrieval ([vector databases](vector-databases.md)).

### Talking to the front end

The transport follows from the execution shape ([state & execution](state-and-execution.md)), and picking it late is expensive because it reaches into the UI:

| Mode | Fits | Notes |
| --- | --- | --- |
| **Plain request/response** | Sub-10 s runs, server-to-server callers | No streaming machinery. Watch the proxy/load-balancer idle timeout — commonly 30–60 s |
| **SSE** (`text/event-stream`) | Token streaming to a chat UI — the default | One-way, plain HTTP, passes proxies, and the browser reconnects on drop sending `Last-Event-ID`. Assign an id per event so resume is possible |
| **WebSocket** | Genuinely bidirectional: barge-in, live cancellation, collaborative or voice UIs | Full duplex, but you now own reconnection, backpressure, and auth on a long-lived connection. Do not take this on just to stream tokens |
| **Submit + poll** | Multi-minute runs, anything with a human gate | `202` with a run id; the client polls status. Survives client disconnects and page reloads, which the streaming modes do not |
| **Webhook / push** | Server-to-server, or notifying a user who has closed the tab | Needs an endpoint, retries, and signature verification on the receiving side |

Two things worth deciding explicitly rather than discovering:

- **SSE reconnection is not run resumption.** `Last-Event-ID` lets the browser re-attach to a stream; it does nothing if the run itself died with the process. Real resumability comes from the run being checkpointed and the stream being replayable from a stored cursor — the transport only carries it.
- **Long runs usually need two channels, not one.** A run id from a `POST` that returns immediately, plus a stream (SSE) or poll attached to that id. The single-long-lived-request design fails the first time someone reloads the page mid-run — and with a human gate in the flow, that is guaranteed.

Buffering is the recurring bug: an intermediary that buffers a `text/event-stream` response turns streaming into a long pause followed by everything at once. Disable response buffering on the path (nginx `X-Accel-Buffering: no`, or the CDN's equivalent) and verify it end to end rather than against the dev server.

## MCP

- Run an MCP server for tool discovery; think of it as client and server.
- See [architecture](../02-design/architecture.md) for where MCP sits in the layers, and [RAG & tooling](rag-and-tooling.md) for serving your own tools over it.
- It is also the cheapest hedge against framework lock-in: tools exposed over MCP are reachable from any client, whatever orchestrates the loop.

**MCP's transport is a different question from the one above.** The table in [talking to the front end](#talking-to-the-front-end) is about your app's users; MCP transport is about how your loop reaches its tools. The spec defines two bindings:

- **stdio** — newline-delimited JSON-RPC over a subprocess's standard streams. For a server on the same machine, single-user. The default for local tooling.
- **Streamable HTTP** — every message is an HTTP POST to one endpoint (conventionally `/mcp`); the server replies with either a JSON object or a request-scoped SSE stream. This is what a remote MCP server should serve.

The older **HTTP+SSE** transport (separate POST and GET-SSE endpoints, from protocol version 2024-11-05) was deprecated in the 2025-03-26 spec and is no longer a defined binding — it survives only as backward compatibility. If a guide has you opening a long-lived `/sse` endpoint alongside a `/messages` one, it predates the change. Note that SSE did not disappear: it is now a response mode *inside* Streamable HTTP rather than a transport of its own.

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
