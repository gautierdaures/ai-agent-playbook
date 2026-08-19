# Measuring the value of an agentic system

If you cannot say how you will measure value **before** building, you are not ready to build. This note is a practical guide for the whole team — technical *and* business — to frame value, agree KPIs, and avoid the traps that make working agents look like failures.

## Why this matters: the measurement gap

The industry has a proof problem, not just a technology problem:

- **88%** of organisations use AI in at least one function, but only **39%** report enterprise-level EBIT impact (McKinsey, *State of AI 2025*).
- **79%** see productivity gains, yet only **29%** can measure ROI confidently.
- **72%** report *no* measurable ROI from AI despite real investment (ActiveOps).
- Gartner forecasts **40% of agentic AI projects will be scrapped by 2027** — largely for *ambiguous business value*, not broken tech.

The lesson from every framework below is the same: **the winners decide what to measure, and what to do with the value, before they deploy.** Rigorous measurement is itself a driver of better outcomes.

Deciding *what to do with the value* is a strategic and ethical choice, not a detail — cut costs, reduce headcount, or redirect freed time to higher-value work? It is covered in depth in [what do you do with the time earned?](#what-do-you-do-with-the-time-earned) below.

## Start from a baseline

Value is a *delta*, so you need the "before". Do a **process map** of the current, agent-free process and capture:

- How long the task takes today, and how often it is done (include peak load, seasonality, edge-case frequency).
- What one run costs — people-time, tooling, and error rework.
- Current quality — error rate, rework rate, escalations.

Without this baseline, any post-launch number is unanchored and the business can dispute it. Skipping the baseline is the single most common reason ROI cannot be proven later.

## The value levers

Consulting frameworks converge on a handful of value dimensions. McKinsey's business-value lens groups them as **efficiency, revenue, capital, and risk**; Gartner's "holistic ROI" adds **process, customer, and option (future-flexibility) value**:

- **Efficiency / time saved** — the task takes less human time (or none).
- **Cost saved / avoided** — lower cost per task, less rework, fewer errors, avoided penalties and churn.
- **Throughput / capacity** — more volume handled with the same headcount.
- **Quality & consistency** — fewer mistakes, uniform output, better compliance.
- **Speed / turnaround** — faster cycle time unlocks new offers (same-day vs. next-week).
- **Revenue** — more conversions, upsell, retention, or offerings the process could not support before.
- **Risk reduction** — fewer compliance breaches, better auditability.
- **Option value** — capabilities and organisational learning that compound over time.

Name the **one primary lever** for *this* project. A system optimised for time saved is not the same as one optimised for quality, and the KPI set follows the lever.

## Risk-adjust the value: the cost of failure

Value is upside *minus* the cost of getting it wrong, times how often that happens. Weigh it explicitly, because a high cost of failure can erase an attractive efficiency gain:

- **How bad is a wrong action, and can it be undone?** High-stakes, irreversible decisions — granting a loan, moving money, a medical or legal call, a purchase — carry a cost of failure that can dwarf the time saved.
- **When the cost of failure is high, the answer may be "not autonomous — or not AI at all."** Keep a human decision-maker, narrow the agent to *preparing* the case rather than deciding it, or use a deterministic rule. The agent produces a report, not a verdict (see [do you need an agent](do-you-need-an-agent.md)).
- **Price the guardrails into the value case.** Human review, audit, and escalation are real recurring costs that lower net value; a use case that only pays *without* oversight is not viable. See [feasibility & readiness](feasibility-and-readiness.md) and [error handling](../02-design/error-handling.md).

## What do you do with the time earned?

This is the question most business cases skip, and it is where ROI is won or lost. **Time saved is only value if the freed capacity is redeployed** — a trap named the *benefit dilution effect* (ActiveOps) or *phantom savings*:

- Only about **half** of AI time savings convert to measurable output when the workforce response is left unmanaged.
- **66%** of AI users get *no guidance* on how to reinvest saved time (BCG).
- Freed time absorbs into longer breaks, lower-priority admin, or a slower pace — the "productivity rebound effect" (St. Louis Fed). Senior workers can show **0%** net gain when routine tasks are automated but nothing replaces them.
- Only about **28%** of organisations are positioned to turn AI deployment into high-value outcomes — the differentiator is people and culture, not the model ([EY Work Reimagined 2025](https://www.ey.com/en_gl/insights/workforce/work-reimagined-survey)).

Decide, at framing time, which destination applies — and make it concrete:

- **Reallocation** — people move to higher-value work the agent cannot do. Value = the difference between the old task and the new work. *This only pays if the new work is genuinely worth more than the old.*
- **Capacity / throughput** — the team absorbs more volume without hiring. Value = the avoided cost of the headcount you did not add.
- **Headcount reduction** — a real but sensitive lever; be explicit if this is the intent, because it changes adoption and trust dynamics (see [adoption & change management](../05-deployment/adoption-and-change-management.md)). Hiding a cost-cutting intent behind "freeing people for higher-value work" corrodes the trust the rollout depends on.
- **Turnaround** — the same work ships faster, unlocking a business capability (a service level, an SLA, a customer promise).

> **Caution:** "We saved 200 hours a month" is a *phantom saving* unless those hours become reallocated work, absorbed volume, or a cut cost line. The business will not feel the value, the ROI model never materialises, and the project looks like a failure even when the agent works perfectly. Tie earned time to a named destination up front, and put visibility on where the freed time actually goes.

## KPIs

Track two layers. Business KPIs prove the value; system KPIs explain and protect it. The critical discipline, stressed by every source, is to **shift from activity metrics to outcome metrics** — from "invoices processed / calls handled" to "revenue influenced / churn prevented / cost avoided".

### Business KPIs (outcome-based)

- **Time saved per task** and **total time saved** (× volume).
- **Cost per task / interaction** — before vs. after (see formula below).
- **Automation / resolution rate** — share of cases handled end-to-end with no human.
- **Throughput** — volume processed per period.
- **Turnaround time** — request to completion.
- **Quality** — error rate, rework rate, compliance rate, first-contact resolution.
- **Customer / user satisfaction** — CSAT, NPS, complaint rate.
- **Revenue influenced / costs avoided** — where the process touches sales, retention, or risk.

### System & adoption KPIs

- **Adoption** — active users, and the share of *eligible* tasks actually run through the agent. A great agent nobody uses creates zero value.
- **Human override / escalation rate** — how often people correct or take over; a live signal of trust and quality.
- **Accuracy / task success rate** and **hallucination rate** — see [evaluation methods](../04-evaluation/evaluation-methods.md).
- **Latency and cost per run** — the operational price of the value; see [cost management](../06-operations/cost-management.md).

Pick a **small** set that maps to the primary lever: one north-star business KPI plus a handful of guardrail KPIs beats a dashboard nobody reads.

### Cost per task — the most honest number

```
Cost per task = (LLM cost + infrastructure + escalation/human-rescue cost) / tasks completed
```

This is clarifying because it folds retries, failures, and human rescues into a single figure — a far more realistic picture than a raw success rate. Track it against the baseline cost per task; the gap, times volume, is your efficiency value.

## A worked example

A back-office team handles 4,000 cases/month; each takes 30 min of an analyst's time (loaded cost ~€40/hour).

- **Baseline:** 4,000 × 0.5 h × €40 = **€80,000/month**.
- **With the agent:** 70% automated end-to-end; the rest still need ~10 min of review.
  - Human time: 4,000 × 30% × (10/60) h × €40 = €8,000
  - Agent run cost (cost-per-task × volume): 4,000 × €0.50 = €2,000
  - New total ≈ **€10,000/month** → gross saving **€70,000/month**.
- **The earned-time decision:** those ~1,850 analyst-hours/month only become value if redeployed — e.g. absorbing a forecast 25% volume increase without hiring, or moving analysts to exception-handling. State which, or the €70k stays theoretical.

Numbers are illustrative — the point is the *structure*: baseline → post-launch → explicit destination for the saving.

## Tie it into the framing lifecycle

Value measurement threads through the framing meetings (see [project framing](project-framing.md)), it is not a separate step:

1. **Define the job & the lever** — what the agent does, and the *one* value lever it pulls. Align it to a specific business outcome ("cut churn 10%→8%", "save 20,000 labour hours/year").
2. **Baseline** — process-map today's time, cost, and quality *with* the business.
3. **Target & destination** — set KPI targets, and decide what happens to the value earned (reallocation, capacity, turnaround…).
4. **Instrument** — commit to observability and evaluation from day one so KPIs are measurable in production, not reconstructed later.
5. **Review on a realistic timeline** — don't judge at month three. Expect ~20–30% gains in pilot, ~50–60% of projections in early production, and typical payback around **12 months**, with value compounding after. Keep or kill on evidence, not vibes.

A project with no baseline, no named lever, and no destination for the earned time is not framed — it is a demo waiting to disappoint.

## Framing checklist

Before building, the team (business + technical) should be able to answer:

- [ ] What is the **one primary value lever**?
- [ ] What is the **baseline** (time, cost, quality) — measured, not guessed?
- [ ] What are the **target KPIs** — one north-star, a few guardrails, all outcome-based?
- [ ] **Where does the earned value go** (reallocation / capacity / headcount / turnaround)?
- [ ] How will KPIs be **instrumented** in production?
- [ ] When and how will we **review** against the baseline?

## References

- [McKinsey — Cost versus value: Managing agentic AI system performance](https://www.mckinsey.com/capabilities/quantumblack/our-insights/cost-versus-value-managing-agentic-ai-system-performance)
- [McKinsey — Seizing the agentic AI advantage](https://www.mckinsey.com/capabilities/quantumblack/our-insights/seizing-the-agentic-ai-advantage)
- [ActiveOps — The Benefit Dilution Effect: why AI reduces work but companies aren't saving time](https://activeops.com/resources/blog/the-benefit-dilution-effect-why-ai-is-reducing-work-but-companies-are-not-saving-time/)
- [The Datawire — The smartest AI business cases put freed capacity back to work](https://www.thedatawire.com/news/redeployment-business-case-prashant-parida-ibm)
- [Enterprise AI Agent ROI: Measuring Value Beyond Cost Reduction (agility-at-scale)](https://agility-at-scale.com/ai/agents/enterprise-ai-agent-roi/)
- [How to Measure ROI from AI Projects: KPIs & Frameworks (Softermii)](https://softermii.com/blog/artificial-intelligence/how-to-measure-roi-from-ai-projects-kpis-frameworks-and-templates/)
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
