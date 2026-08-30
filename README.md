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

Some concerns are **cross-cutting** — value, evaluation, error handling, cost, observability, security, adoption. Some are designed in an early phase and run later; others are native to one phase but referenced from all of them. Either way, each has **one canonical home** and is linked from everywhere else rather than re-explained in every section:

| Cross-cutting concern | Canonical home |
|---|---|
| Value, KPIs, freed-time ROI | [`01-framing/measuring-value`](01-framing/measuring-value.md) |
| Autonomy levels | [`01-framing/do-you-need-an-agent`](01-framing/do-you-need-an-agent.md) |
| Human-in-the-loop design | [`02-design/human-in-the-loop`](02-design/human-in-the-loop.md) |
| Error handling, retries & stop conditions | [`02-design/error-handling`](02-design/error-handling.md) |
| Evaluation techniques | [`04-evaluation/evaluation-methods`](04-evaluation/evaluation-methods.md) |
| Human review & annotation | [`04-evaluation/eval-datasets`](04-evaluation/eval-datasets.md) |
| Release gating on evals | [`04-evaluation/regression-and-drift`](04-evaluation/regression-and-drift.md) |
| Staged rollout / progressive autonomy | [`05-deployment/rollout-and-safety`](05-deployment/rollout-and-safety.md) |
| Adoption & change management | [`05-deployment/adoption-and-change-management`](05-deployment/adoption-and-change-management.md) |
| Cost (designed in 2, run in 6) | [`06-operations/cost-management`](06-operations/cost-management.md) |
| Logging & observability | [`06-operations/monitoring-and-observability`](06-operations/monitoring-and-observability.md) |
| Alerting & anomaly detection | [`06-operations/alerting-and-anomaly-detection`](06-operations/alerting-and-anomaly-detection.md) |
| Incident response | [`06-operations/incident-response`](06-operations/incident-response.md) |
| Production feedback loop & iteration | [`06-operations/continuous-improvement`](06-operations/continuous-improvement.md) |
| Security, privacy & compliance | [`02-design/security-and-compliance`](02-design/security-and-compliance.md) |

## How to use

Each folder holds atomic notes as markdown files. Start a note whenever a thought
doesn't fit an existing one. Keep this README as the map.
