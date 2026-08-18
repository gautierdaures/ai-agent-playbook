# Measuring the value of an agentic system

If you cannot say how you will measure value **before** building, you are not ready to build. The framing phase must produce a baseline, a target, and a plan for what to do with the value once it lands.

## Start from a baseline

Value is a *delta*, so you need the "before". Measure the current process without the agent:

- How long does the task take today, and how often is it done?
- What does one run cost (people-time, tooling, error rework)?
- What is the current quality — error rate, rework rate, escalations?

Without this baseline, any post-launch number is unanchored and the business can dispute it.

## The value levers

Most agentic systems create value through one or more of these:

- **Time saved** — the task takes less human time (or none).
- **Cost saved** — lower cost per task, less rework, fewer errors.
- **Throughput / capacity** — more volume handled with the same headcount.
- **Quality & consistency** — fewer mistakes, more uniform output, better compliance.
- **Speed / latency** — faster turnaround changes what the business can offer (e.g. same-day vs. next-week).
- **Revenue** — more conversions, upsell, or new offerings the process could not support before.
- **Risk reduction** — fewer compliance breaches, better auditability.

Name the primary lever for *this* project. A system optimised for time saved is not the same as one optimised for quality.

## What do you do with the time earned?

This is the question most business cases skip, and it is where ROI is won or lost. **Time saved is only value if the freed capacity is redeployed.** Decide, at framing time, which of these applies:

- **Reallocation** — people move to higher-value work the agent cannot do. The value is the difference between the old task and the new work.
- **Capacity / throughput** — the team absorbs more volume without hiring. The value is the avoided cost of the headcount you did not add.
- **Headcount reduction** — a real but sensitive lever; be explicit if this is the intent, because it changes adoption dynamics and trust.
- **Turnaround** — the same work ships faster, unlocking a business capability (a service level, an SLA, a customer promise).

> **Caution:** "We saved 200 hours a month" is a *phantom saving* if those hours don't turn into reallocated work, absorbed volume, or a cut cost line. The business will not feel the value, and the project looks like it failed even when the agent works. Tie earned time to a concrete destination up front.

## KPIs

Track two layers. Business KPIs prove the value; system KPIs explain and protect it.

### Business KPIs

- **Time saved per task** and **total time saved** (× volume).
- **Cost per task** — before vs. after.
- **Automation / resolution rate** — share of cases handled end-to-end with no human.
- **Throughput** — volume processed per period.
- **Turnaround time** — request to completion.
- **Quality** — error rate, rework rate, compliance rate.
- **Customer / user satisfaction** — CSAT, NPS, complaint rate.
- **Revenue impact** — where the process touches sales.

### System & adoption KPIs

- **Adoption** — active users, share of eligible tasks actually run through the agent. A great agent nobody uses creates zero value.
- **Human override / escalation rate** — how often people correct or take over. Trends here signal trust and quality.
- **Accuracy / task success rate** — see [evaluation methods](../04-evaluation/evaluation-methods.md).
- **Latency and cost per run** — the operational price of the value; see [cost management](../05-deployment/cost-management.md).

Pick a **small** set that maps to the primary lever. One north-star business KPI plus a handful of guardrail KPIs beats a dashboard nobody reads.

## Tie it into the framing lifecycle

Value measurement is not a separate step — it threads through the framing meetings (see [project framing](project-framing.md)):

1. **Define the job** — what the agent does, and the *one* lever it pulls.
2. **Baseline** — measure today's time, cost, and quality with the business.
3. **Target & destination** — set the KPI targets, and decide what happens to the value earned (reallocation, capacity, turnaround…).
4. **Instrument** — commit to observability and evaluation from day one so the KPIs are measurable in production, not reconstructed later.
5. **Review** — compare against the baseline after launch; keep or kill on evidence, not vibes.

A project with no baseline, no named lever, and no destination for the earned time is not framed — it is a demo waiting to disappoint.

## References

- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
