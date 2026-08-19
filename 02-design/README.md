# 2. Design

How the agent is put together.

_Scope: framing-to-design translation, architecture, the agent loop, tool design, memory & context, prompting, guardrails & safety._

## Notes

- [from-framing-to-technical-design.md](from-framing-to-technical-design.md) — turning framing artefacts into design decisions; the agent spec; the technical framing doc.
- [agent-anatomy-and-prompting.md](agent-anatomy-and-prompting.md) — Perception/Reasoning/Action; prompt frameworks (COSTAR/RISEN/CCR) and good practices.
- [memory.md](memory.md) — short-term (session) vs. long-term (vector store); what to keep.
- [architecture.md](architecture.md) — the core components, the control loop, BDI, layered & hexagonal separation, MCP, evolvability.
- [multi-agent-orchestration.md](multi-agent-orchestration.md) — peer-to-peer, central and multi-level orchestrators; planning without an LLM.
- [error-handling.md](error-handling.md) — hallucinations, loops, tool errors, context overflow.
- [security-and-compliance.md](security-and-compliance.md) — prompt injection, least privilege, GDPR & AI Act.
