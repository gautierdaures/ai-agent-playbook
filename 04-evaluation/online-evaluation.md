# Online evaluation

**Are cold evals enough?** No — an offline suite only covers inputs you thought of, and production distribution moves. **Should you therefore put an LLM judge in your product code?** Usually also no. Those are two different questions that get collapsed into one, and the answer depends entirely on whether the check *changes what the user sees*.

## The dividing line

| | Guardrail | Online eval | Offline eval |
| --- | --- | --- | --- |
| **Runs** | In the request path, synchronously | On the trace stream, asynchronously | In CI / pre-release |
| **Effect** | Blocks, rewrites, or escalates | Changes only what you know | Gates a deploy |
| **Coverage** | 100% of traffic | Sampled | Fixed dataset |
| **Latency budget** | Hard — it's user-facing | None | None |
| **Must handle** | Its own failure (fail open or closed?) | Nothing; it can lag or drop | Flakiness |
| **Cost model** | Per request, forever | Sampling rate × judge cost | Per run |

**The rule: if it changes the response, it is a guardrail and belongs in the product code with a latency budget, a timeout, and an explicit fail-open/fail-closed decision. If it only produces a number, it runs out of band on the trace.**

Putting a judge in the synchronous path *just to record a metric* is the common mistake: it doubles cost and latency, and it adds a dependency that can fail the user request in order to compute a statistic nobody reads in real time.

Where a check must block, keep it cheap and preferably deterministic — schema validation, regex, an amount cap, a small classifier — and run it concurrently with the model call rather than after it, so the check hides inside the generation latency ([guardrail evaluation](guardrail-evaluation.md)).

## Sample deliberately

- **100% coverage** for anything cheap and deterministic: output schema valid, citation IDs resolve, refusal detected, tool error occurred, step limit hit, PII pattern present. These are effectively free and give you dense signal.
- **1–10%** for judge-scored dimensions. That is enough to move a trend line; it is not enough to catch a rare individual failure, and it isn't meant to.
- **Stratify, don't sample uniformly.** Always score: escalations, low-confidence runs, runs where the user retried or abandoned, high-value transactions, new prompt/model versions, new tenants. Uniform sampling spends your budget on the happy path.

## The free signals beat the judge

Before spending on judges, harvest what production already tells you. These are closer to ground truth than any rubric:

- **The user edited the draft** — and the diff shows exactly what was wrong.
- **The user retried, rephrased, or abandoned** mid-session.
- **The human override / escalation rate** ([human-in-the-loop](../02-design/human-in-the-loop.md)).
- **Downstream outcome** — the ticket reopened, the invoice was corrected, the PR was reverted.
- **Explicit feedback**, which is sparse and biased toward anger, but cheap.

Edits and overrides are the strongest quality signal an agent product produces, and they cost nothing to collect. Wire them into the trace from day one.

## Detecting a behavioural anomaly

This is the quality half of "something is wrong"; the infrastructure half — endpoint down, 429, expired credential — is [alerting & anomaly detection](../06-operations/alerting-and-anomaly-detection.md).

Metrics whose *distribution* is the alarm, not their value:

- Refusal rate, escalation rate, "I don't know" rate
- Tool-call count and iterations per run
- Output length distribution
- Retrieval hit rate / empty-result rate
- Judge score moving average, per dimension
- Schema-repair rate

Practical rules:

- **Alert on change against a rolling baseline**, not a fixed threshold. Any absolute threshold on a noisy quality metric either pages constantly or never.
- **Segment before you alert.** A global average hides a collapse in one tenant, one language, or one intent. The aggregate is where regressions go to be invisible.
- **Attribute by version.** Every trace carries the prompt version, model version, and tool-definition hash, so a shift lands on a specific change instead of a mystery ([regression & drift](regression-and-drift.md)).
- **Watch the input distribution too.** Users bringing new intents, longer inputs, or another language degrades quality with nothing having changed on your side — and that is a product signal, not a bug.

## Close the loop

Every online failure — a breached policy, a bad judge score, a user override — becomes an offline case within the week. That flow is the only thing that keeps the offline suite representative, and it is what makes production monitoring an evaluation activity rather than a dashboard.

A/B testing sits on top: once traffic is sufficient, compare versions on live outcomes rather than on eval scores. It is the most trustworthy comparison available and the slowest — offline gates the deploy, A/B confirms the win ([rollout & safety](../05-deployment/rollout-and-safety.md)).

## References

- [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — the progression from CI evals to production monitoring, A/B testing, and ongoing transcript review.
- [Langfuse — human annotation](https://langfuse.com/docs/evaluation/evaluation-methods/annotation) — attaching human scores to production traces, sessions, and observations.
- [Google SRE Workbook — alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) — burn-rate alerting, applicable to a quality SLO as much as an availability one.
