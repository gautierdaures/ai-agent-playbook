# From business framing to technical design

Framing ([section 1](../01-framing/README.md)) answers *what* and *why*. Design ([section 2](README.md)) answers *how*. Between them sits a translation step that is easy to skip and expensive to skip: turning the framing artefacts — the use-case card, the one value lever, the success criteria — into technical decisions. This note is that bridge.

## What framing hands over

Before you draw a single box, gather the outputs framing already produced. If any are missing, you are not ready to design — go back:

- **The job in plain terms** and the reformulated steps ([project framing](../01-framing/project-framing.md), meetings 1–2).
- **The one primary value lever** and the **baseline** it improves on ([measuring value](../01-framing/measuring-value.md)).
- **Success criteria expressed as numbers**, plus the definition of "done" ([scoping the first version](../01-framing/scoping-the-first-version.md)).
- **The autonomy level** you committed to — single call, workflow, or agent ([do you need an agent?](../01-framing/do-you-need-an-agent.md)).
- **Feasibility notes** — what data and which systems the agent may touch ([feasibility & readiness](../01-framing/feasibility-and-readiness.md)).
- **Non-negotiables** — compliance boundaries, human review gates, the RACI ([project framing](../01-framing/project-framing.md)).

## The translation

Each framing output forces a concrete design question. Work down the list; the right-hand column is where you spend section 2.

| Framing output | Design question it forces | Where it's answered |
| --- | --- | --- |
| The job, decomposed into steps | Which steps are LLM reasoning vs. deterministic code? | [architecture](architecture.md), [multi-agent](multi-agent-orchestration.md) |
| The one value lever | What does the architecture optimise for — latency, quality, or cost? | [architecture](architecture.md), [cost management](../06-operations/cost-management.md) |
| Success criteria as numbers | What is the eval set and which metrics gate a release? | [evaluation methods](../04-evaluation/evaluation-methods.md) |
| Autonomy level | Agent vs. workflow; how much the control loop decides; how tight the guardrails | [architecture](architecture.md), [error handling](error-handling.md) |
| Data readiness | Short-term vs. long-term memory; RAG or not | [memory](memory.md), [RAG & tooling](../03-build/rag-and-tooling.md) |
| Tool / system access | The tool inventory, MCP surface, and permission scope per tool | [RAG & tooling](../03-build/rag-and-tooling.md), [security & compliance](security-and-compliance.md) |
| Compliance non-negotiables | PII boundaries, audit trail, where humans must sign off | [security & compliance](security-and-compliance.md) |
| RACI & review gates | Where the human-in-the-loop sits inside the flow | [project framing](../01-framing/project-framing.md) |

## The agent spec — the smallest unit of design

Before the system diagram, pin down each agent as a short spec. Elaborate personas are rarely needed; a role, a goal, and constraints usually suffice. Capture:

- **Role & goal** — who the agent is and the single outcome it owns.
- **Input / output contract** — what it receives and the exact shape it returns (for example a Pydantic schema — see [agent anatomy & prompting](agent-anatomy-and-prompting.md)).
- **Tools and permission scope** — what it may call, and with what least-privilege rights ([security & compliance](security-and-compliance.md)).
- **Stop conditions** — when it is done, and the hard limits (max steps, budget, timeout) that stop a runaway loop ([error handling](error-handling.md)).
- **Success criteria** — the numbers from framing, restated at the agent level so they become evals.

## Write it down: the technical framing doc

The artefact of this step is a short **solution-design / technical-framing doc** — treat it like a PRD or system-design doc, with acceptance criteria and KPIs carried straight over from framing. It contains: the step decomposition, the agent spec(s), the chosen architecture pattern, the tool and data inventory, the non-functional budgets (latency, cost, availability), the eval plan, and the known risks. Keep it traceable back to the one-page use-case card so a reviewer can follow a design choice all the way to the business reason for it.

## Don't over-design

Start with the simplest thing that can hit the success criteria, and escalate only when the numbers demand it: a single call, then a fixed workflow, then an agent, then multiple agents. Autonomy is a cost, not a feature ([do you need an agent?](../01-framing/do-you-need-an-agent.md)). The technical framing doc should justify each step up that ladder with a measurement, not a preference.

## References

- [How to Write a Good Spec for AI Agents](https://www.oreilly.com/radar/how-to-write-a-good-spec-for-ai-agents/) (O'Reilly) — treating the agent spec as a structured, outcome-grounded document.
- [Choose a design pattern for your agentic AI system](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system) (Google Cloud Architecture Center) — mapping requirements to an architecture pattern.
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — start simple, add autonomy only when it pays for itself.
