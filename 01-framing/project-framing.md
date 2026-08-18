# Framing the project

Once you know the use case is worth building ([use-case selection](use-case-selection.md)) and buildable ([feasibility & readiness](feasibility-and-readiness.md)), framing turns it into a project: the right people, a shared understanding of the job, and the non-negotiables you commit to from day one.

## Build with people who know the domain

Staff the project cross-functionally. An agent touches business process, data, risk, and software, so no single function can frame it alone:

- **AI / product owner** — owns the business outcome and priorities; the "business lead" of the project.
- **Domain experts** — the people who do the task today; they hold the ground truth and the edge cases.
- **Data team** — owns availability, quality, and access to the data the agent needs.
- **Engineering** — builds the agent, tools, and integrations.
- **Risk / security / compliance** — especially in regulated settings; brought in early, not at the end. See [security & compliance](../02-design/security-and-compliance.md).

## Assign accountability clearly — and keep it human

Use a lightweight RACI (Responsible / Accountable / Consulted / Informed) so every part of the work has a clear owner. One rule matters most for agents:

> **An AI agent can be Responsible, Consulted, or Informed — never Accountable.** Accountability always stays with a named person, and every agent-held Responsible role needs a human review gate.

This keeps the human-in-the-loop principle structural, not aspirational.

## Map the stakeholders

Beyond the core team, sort the people affected by a power/interest grid and engage accordingly: work closely with the high-power/high-interest sponsors, keep high-power/low-interest players satisfied, keep high-interest allies (often the end users) informed, and give everyone else light-touch updates. Adoption depends on the people who do the work today feeling brought along, not replaced by surprise.

## The framing meetings

- **Meeting 1** — What will the AI actually do? Define the job in plain terms, name the users, and the one primary value lever.
- **Meeting 2** — Reformulate the steps and validate them with the business. Agree the baseline and the success criteria.
- **Meeting 3** — Implementation with the technical team, with a feedback loop back to the business.

Keep a **one-page use-case card** (see [use-case selection](use-case-selection.md)) as the living artefact these meetings refine.

## Design goals to commit to from day one

Architect for these from the start; they are not afterthoughts, and retrofitting them is far more expensive:

- **Observability** — you cannot improve or debug what you cannot see. See [monitoring & observability](../06-operations/monitoring-and-observability.md).
- **Continuous evaluation** — a repeatable way to measure quality as the system changes. See [evaluation methods](../04-evaluation/evaluation-methods.md).
- **Production monitoring** — live health, cost, and quality signals. See [metrics & SRE](../06-operations/metrics-and-sre.md).
- **Guardrails** — safety, permissions, and human gates on irreversible actions. See [error handling](../02-design/error-handling.md).

## Running projects in parallel

> **Caution:** Running several agent projects in parallel is risky. Split attention and diverging approaches make each one harder to land. Prove the pattern on one narrow project first (see [scoping the first version](scoping-the-first-version.md)), then reuse what the team learned.

## References

- [AI agents in a RACI chart: where they fit (Profit.co)](https://www.profit.co/blog/project-management/ai-agents-in-your-raci-where-they-fit/)
- [RACI matrix for AI accountability (agility-at-scale)](https://agility-at-scale.com/ai/governance/raci-matrix-for-ai-accountability/)
- [How to assign RACI roles when AI agents join the team (PMI)](https://www.pmi.org/blog/stakeholder-management-raci)
