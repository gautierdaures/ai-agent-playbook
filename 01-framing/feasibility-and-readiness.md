# Feasibility & readiness

A use case can be valuable and still be un-buildable *right now*. Feasibility assessment answers a blunt question: **can we actually deliver this, with the data, tools, and access we have?** Getting this wrong is the quiet killer — Gartner expects **60% of AI projects to be abandoned through 2026 for lack of AI-ready data foundations** ([Gartner, 2025](https://www.gartner.com/en/newsroom/press-releases/2025-02-26-lack-of-ai-ready-data-puts-ai-projects-at-risk)).

> **Note:** This is a *different* Gartner prediction from the "over 40% of agentic AI projects cancelled by 2027" figure cited in [do you need an agent](do-you-need-an-agent.md). They are not in conflict: the 60% covers *all* AI projects through 2026 and blames *data readiness*; the 40% covers *agentic* projects through 2027 and blames cost, unclear value, and weak risk controls. Both point at the same lesson — most failures are framing and foundations, not the model.

## Data readiness is the binding constraint

Model capability is rarely the bottleneck; data usually is.

- Only about **7%** of enterprises say their data is completely ready for AI ([Cloudera / Harvard Business Review Analytic Services, 2026](https://www.cloudera.com/about/news-and-blogs/press-releases/2026-03-05-only-7-percent-of-enterprises-say-their-data-is-completely-ready-for-ai-according-to-new-report-from-cloudera-and-harvard-business-review-analytic-services-reveals.html)).
- Winning AI programmes earmark **50–70% of timeline and budget for data readiness** (Gartner). On a €100k project, that is €50–70k of work that vanishes if you didn't plan for it.

Check, before committing:

- **Availability** — does the data the task needs exist, and can the agent reach it at run time?
- **Quality** — is it accurate, complete, current, and consistent enough to act on?
- **Access & permissions** — who owns it, and is access allowed for this use? See [security & compliance](../02-design/security-and-compliance.md).
- **Ground truth** — do you have labelled examples or historical outcomes to evaluate against? See [evaluation methods](../04-evaluation/evaluation-methods.md).

> **Caution:** Do the data and governance work *before* infrastructure spend, not after. A polished agent on top of unready data fails in production in ways demos never reveal.

## Tool & system feasibility

An agent's reach is defined by its tools (see [RAG & tooling](../03-build/rag-and-tooling.md)). Map, for the target workflow:

- Every system the agent must **read from** and **write to**.
- Whether APIs / integrations exist, or must be built.
- Which actions are reversible vs. irreversible (the irreversible ones need a human gate and shape the design — see [error handling](../02-design/error-handling.md)).
- Where the workflow **fails safely** if a tool is unavailable or wrong.

## Technical feasibility

- **Task complexity vs. autonomy needed** — can the path be hardcoded (workflow) or must the model decide at run time (agent)? See [do you need an agent](do-you-need-an-agent.md).
- **Latency and cost budget** — is the required cost-per-task and response time achievable at the target volume? See [cost management](../05-deployment/cost-management.md).
- **Quality bar** — is the acceptable error/escalation rate reachable with today's models on this task?

## Build vs. buy

Treat this as one of the connected framing decisions, not an afterthought:

- **Buy / configure** an off-the-shelf agent when the problem is common (e.g. customer-support triage), speed matters more than fit, and vendors already solve it well. Solving a problem *now* often beats solving it perfectly.
- **Build** when the workflow is core to your differentiation, deeply tied to proprietary data/systems, or when long-term control and unit economics favour ownership.
- **Hybrid** — buy the platform/orchestration, build the domain-specific logic and tools on top. Common and often the pragmatic answer.

Weigh total cost of ownership, integration effort, data-privacy constraints, and lock-in — not just the sticker price. See [frameworks & infra](../03-build/frameworks-and-infra.md).

## Organisational readiness

Feasibility is not only technical:

- **Ownership** — is there a named accountable owner and an engaged business sponsor? See [project framing](project-framing.md).
- **Adoption & change management** — will the people who do this work actually use it? A feasible agent nobody adopts still delivers zero value. This is big enough to own its own workstream — see [adoption & change management](../05-deployment/adoption-and-change-management.md).
- **Governance** — are review gates, audit, and escalation paths defined before launch, not bolted on?

## References

- [Is your data ready for agents and AI? (NexusOne)](https://nx1.medium.com/is-your-data-ready-for-agents-and-ai-9f0ebb5de063)
- [Practical guide to AI feasibility studies (Geniusee)](https://geniusee.com/single-blog/ai-feasibility)
- [Build vs. buy AI agents (Retool)](https://retool.com/blog/build-vs-buy-ai-agents)
- [How to measure AI readiness — assessment guide (OvalEdge)](https://www.ovaledge.com/blog/measuring-ai-readiness)
