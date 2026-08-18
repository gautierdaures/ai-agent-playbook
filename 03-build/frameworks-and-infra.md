# Frameworks & infrastructure

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

## API layer

- **Flask vs. FastAPI** — open question for exposing the agent as a service. FastAPI is the common modern default (async, typed, automatic OpenAPI docs); Flask is fine for small synchronous services.

## MCP

- Run an MCP server for tool discovery; think of it as client and server.
- See [architecture](../02-design/architecture.md) for where MCP sits in the layers.

## References

- [LangChain — The best AI agent frameworks](https://www.langchain.com/resources/ai-agent-frameworks)
- [Turing — Comparison of top AI agent frameworks](https://www.turing.com/resources/ai-agent-frameworks)
- [Firecrawl — Best open source agent frameworks](https://www.firecrawl.dev/blog/best-open-source-agent-frameworks)
