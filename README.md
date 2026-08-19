# AI Agent Playbook

Notes and thinking on building AI agents — from framing a project through to running it in production.

## Lifecycle

| Phase | Folder | Scope |
|-------|--------|-------|
| 1. Framing | [`01-framing/`](01-framing/) | Deciding what to build and why: problem, users, feasibility, success criteria. |
| 2. Design | [`02-design/`](02-design/) | Architecture, agent loop, tools, memory, prompts, guardrails. |
| 3. Build | [`03-build/`](03-build/) | Implementation, integrations, tooling, development workflow. |
| 4. Evaluation | [`04-evaluation/`](04-evaluation/) | Measuring quality: evals, benchmarks, human review, regression testing. |
| 5. Deployment | [`05-deployment/`](05-deployment/) | Shipping: rollout, infra, release strategy, gating. |
| 6. Operations | [`06-operations/`](06-operations/) | Running in production: monitoring, cost, incidents, iteration. |

The phases are a **map, not a waterfall.** Real projects iterate Scrum-style — evaluation feeds back into framing, production signals reopen design — so treat the order as the dependency structure, not a one-way gate.

Some concerns are **cross-cutting**: value, evaluation, cost, observability, security, and adoption are all *designed* in an early phase but *run* later. To keep them from being re-explained in every section, each has **one canonical home** and is linked from everywhere else:

| Cross-cutting concern | Canonical home |
|---|---|
| Value, KPIs, freed-time ROI | [`01-framing/measuring-value`](01-framing/measuring-value.md) |
| Autonomy levels | [`01-framing/do-you-need-an-agent`](01-framing/do-you-need-an-agent.md) |
| Human-in-the-loop design | [`02-design/human-in-the-loop`](02-design/human-in-the-loop.md) |
| Evaluation techniques | [`04-evaluation/evaluation-methods`](04-evaluation/evaluation-methods.md) |
| Staged rollout / progressive autonomy | [`05-deployment/rollout-and-safety`](05-deployment/rollout-and-safety.md) |
| Adoption & change management | [`05-deployment/adoption-and-change-management`](05-deployment/adoption-and-change-management.md) |
| Cost | [`06-operations/cost-management`](06-operations/cost-management.md) |
| Logging & observability | [`06-operations/`](06-operations/) |
| Security, privacy & compliance | [`02-design/security-and-compliance`](02-design/security-and-compliance.md) |

## How to use

Each folder holds atomic notes as markdown files. Start a note whenever a thought
doesn't fit an existing one. Keep this README as the map.
