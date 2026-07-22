# 2. Design

How the agent is put together.

_Scope: architecture, the agent loop, tool design, memory & context, prompting, guardrails & safety._

## Notes

- [agent-anatomy-and-prompting.md](agent-anatomy-and-prompting.md) — Perception/Reasoning/Action; prompt frameworks (COSTAR/RISEN/CCR) and good practices.
- [memory.md](memory.md) — short-term (session) vs. long-term (vector store); what to keep.
- [architecture.md](architecture.md) — BDI, layered separation of concerns, MCP, hexagonal, evolvability.
- [multi-agent-orchestration.md](multi-agent-orchestration.md) — peer-to-peer, central and multi-level orchestrators; planning without an LLM.
- [error-handling.md](error-handling.md) — hallucinations, loops, tool errors, context overflow.
- [security-and-compliance.md](security-and-compliance.md) — prompt injection, least privilege, GDPR & AI Act.
