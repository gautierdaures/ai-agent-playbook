# 2. Design

How the agent is put together.

_Scope: framing-to-design translation, architecture, the agent loop, tool design, memory & context, prompting, human-in-the-loop, guardrails & safety._

## Notes

- [from-framing-to-technical-design.md](from-framing-to-technical-design.md) — turning framing artefacts into design decisions; the agent spec; the technical framing doc.
- [architecture.md](architecture.md) — the core components, the control loop, BDI, layered & hexagonal separation, MCP, evolvability.
- [agent-anatomy-and-prompting.md](agent-anatomy-and-prompting.md) — Perception/Reasoning/Action; prompt frameworks (COSTAR/RISEN/CCR) and good practices.
- [tool-design.md](tool-design.md) — tools as agent/system contracts; descriptions the model reads, granularity, actionable errors, iteration.
- [memory.md](memory.md) — context engineering; what goes in the window each turn; short- vs. long-term memory.
- [multi-agent-orchestration.md](multi-agent-orchestration.md) — peer-to-peer, central and multi-level orchestrators; planning without an LLM.
- [human-in-the-loop.md](human-in-the-loop.md) — approval / review / escalation patterns; where gates go; progressive autonomy.
- [error-handling.md](error-handling.md) — hallucinations, loops, tool errors, context overflow; detecting failures in production.
- [security-and-compliance.md](security-and-compliance.md) — prompt injection, least privilege, GDPR & AI Act.
