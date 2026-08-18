# Choosing and prioritising the use case

Before asking *how* to build, ask *which* problem is worth building for. Most first agent projects fail because the **scope was wrong**, not because the model was weak. Picking the right use case is the highest-leverage decision in the whole lifecycle.

## What makes a good first use case

Look for a task that is:

- **High-volume and repetitive** — the value of any per-task saving multiplies with volume.
- **Already done by a human today** — you get a built-in baseline and ground-truth examples (see [measuring value](measuring-value.md)).
- **Clear input, clear output** — a well-defined job beats a fuzzy "assistant for everything".
- **Tolerant of a human review gate** — so early mistakes are caught, not shipped.
- **Backed by accessible data and tools** — feasibility is often the binding constraint (see [feasibility & readiness](feasibility-and-readiness.md)).

Avoid, for a first project: safety-critical decisions with no human in the loop, tasks with no measurable outcome, and anything where the required data does not yet exist.

## Prioritise on value vs. feasibility

The consensus framework (Gartner's *value vs. feasibility*, Deloitte's *value-to-effort matrix*) scores each candidate on two axes and plots them:

- **Business value** — impact on the primary lever: cost, revenue, throughput, quality, risk.
- **Feasibility** — data availability, tool/API access, technical complexity, and governance readiness.

|                     | **Low value**        | **High value**            |
|---------------------|----------------------|---------------------------|
| **High feasibility**| Quick wins (do some) | **Start here** — flagship |
| **Low feasibility** | Drop                 | Strategic bets (later)    |

Keep the scoring lightweight: **3–4 criteria, a 3-point scale, one page per candidate.** A whole enterprise portfolio can be scored in 3–5 weeks; a single team's shortlist in an afternoon.

## Add filters that catch hidden effort

Two extra filters stop attractive-looking candidates from sinking later:

- **Time-to-value** — Deloitte suggests deprioritising anything that cannot show value in **under ~12 weeks**. Long-payoff bets are fine, but not as your *first* project.
- **Governance / risk** — in regulated settings (finance, health, public sector) the binding constraint is often not value or feasibility but *whether the use case survives regulatory scrutiny*. Weight this heavily enough that it can veto an otherwise attractive candidate. See [security & compliance](../02-design/security-and-compliance.md).

> **Caution:** GenAI use cases carry effort that traditional automation does not — output-quality variance, human-review overhead, and governance work. These are easy to underestimate at the scoring stage and routinely inflate the "feasibility" cost. Score them in explicitly.

## The one-page use-case card

Force each candidate onto a single page so they can be compared like-for-like:

- **Problem** — the job, in plain business terms.
- **Users** — who does this today, who will use the agent, who is affected.
- **Primary value lever** and rough size (volume × per-task value).
- **Baseline** — current time, cost, quality.
- **Feasibility** — data, tools, complexity, governance flags.
- **Success criteria** — what "good enough to ship" looks like (see [scoping the first version](scoping-the-first-version.md)).

## References

- [Gartner-style value vs. feasibility — AI use-case identification & prioritisation (agility-at-scale)](https://agility-at-scale.com/ai/strategy/ai-use-case-identification-and-prioritization/)
- [Use Case Prioritization Framework for AI Products (Toptal)](https://www.toptal.com/product-managers/artificial-intelligence/use-case-prioritization-framework)
- [AI use-case prioritisation matrix (Zuci Systems)](https://www.zucisystems.com/blogs/ai-use-case-prioritization-matrix/)
