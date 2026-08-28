# Regression, drift & the release gate

## Prompt drift and model drift

Behaviour shifts when the model changes *or* when anything in the assembled context changes. Treat prompt drift exactly like model drift: something to detect and guard against, not something you notice from a support ticket.

The prompt is a **deployed artefact**. Version it, pin it, and stamp the version ID — along with the model ID and the tool-definition hash — into every trace. Without that, a quality shift is a mystery; with it, a shift is attributable to a specific change within minutes ([monitoring & observability](../06-operations/monitoring-and-observability.md)).

## Decide the gate before you see the numbers

"I changed a prompt — can I deploy?" is answerable only if the criteria were fixed in advance. Otherwise the result is negotiated after the fact, which is how every regression ships.

Pre-register the gate:

| Criterion | Typical rule |
| --- | --- |
| **Blocking guardrails** | Zero failures. Not a percentage ([guardrail evaluation](guardrail-evaluation.md)) |
| **Regression suite** | Not worse than baseline by more than δ, at k trials |
| **Format / schema validity** | Zero failures |
| **Cost per run** | Within +X% of baseline |
| **p95 latency** | Within +X% of baseline |
| **Capability suite** | Reported, not gating — it is allowed to move in either direction |

Pick δ explicitly. "No regression" without a number is unfalsifiable given sampling noise.

## Ask for non-inferiority, not improvement

The deploy question is not *"is the new prompt better?"* — it is *"am I confident the new prompt is not worse by more than δ?"* Those need different tests, and answering the first when you meant the second blocks harmless changes and waves through small consistent losses.

## Compare paired, on identical cases

Run old and new on the **same cases with the same seeds** and analyse the per-case differences, rather than comparing two independent pass rates:

- Question-level scores are strongly correlated between model versions (roughly 0.3–0.7 in Anthropic's measurements). Paired analysis exploits that correlation and is essentially a **free variance reduction** — a much tighter confidence interval for the same number of cases.
- Report **mean difference with a confidence interval**, not "82% → 84%". Two pass rates with overlapping intervals is not an improvement, and presenting it as one is the most common eval mistake in release reviews.
- **Cluster your standard errors** when cases aren't independent — several turns of one conversation, several questions on one document. Naive standard errors can be more than 3× too small, which turns noise into a confident "improvement".
- **Multiple trials per case** reduce the per-case sampling noise before it enters the comparison.

## Have enough cases to detect what you care about

Decide the smallest regression worth catching (say 3 percentage points), then run the power analysis: can the suite detect it at that variance? If not, "no regression detected" carries no information. For scale: an unpaired pass rate measured on 40 cases around 80% carries a 95% interval of roughly ±12 points. Pairing shrinks that substantially, more trials shrink it further, but a small suite fundamentally only catches large breaks — either grow it or state that limit in the gate.

## Don't tune on the gate

- **Frozen golden set** — never edited, never used to iterate on prompts. This is the honest gate.
- **Rolling set** — grows continuously from production failures ([online evaluation](online-evaluation.md)); used for development.

Iterating a prompt against the set that decides your release is overfitting to your own gate. The score rises; the product doesn't.

## Model swaps

A model upgrade is the largest regression event you will run, and deprecation schedules mean it is not optional. A repeatable checklist:

1. Full regression + guardrail suites, paired against the incumbent.
2. **Re-validate the judges** — they are models too, and a judge whose backing model changed is measuring something new ([LLM-as-judge](llm-as-judge.md)).
3. Re-check cost and latency; a better model at 3× the cost may fail the value case ([cost management](../06-operations/cost-management.md)).
4. Re-run the capability set to find what is now newly *possible* — upgrades often unlock scope, and the eval suite is how you notice.
5. Shadow or canary before full traffic.

## Deploy mechanics

The offline gate authorises a *canary*, not a fleet-wide switch:

- Route a percentage of traffic to the new version, tagged by version ID.
- Compare online metrics between canary and control — the free signals (edits, overrides, retries, escalations) plus sampled judge scores.
- Define the automatic rollback condition in advance, ideally as an error-budget burn rate rather than a raw threshold.
- Keep the old prompt version deployable; rollback must be a config change, not a rebuild.

Staged rollout mechanics live in [rollout & safety](../05-deployment/rollout-and-safety.md); the production metrics being watched are in [key metrics & SRE](../06-operations/metrics-and-sre.md).

## Drift you cannot catch offline

Your suite is a snapshot of a distribution that keeps moving: new user intents, new document formats, a new tenant, seasonal load, another language. Monitor the **input** distribution alongside the outputs — quality can degrade with nothing on your side having changed, and no offline suite will ever tell you that.

## References

- [Anthropic — A statistical approach to model evaluations](https://www.anthropic.com/research/statistical-approach-to-model-evals) and the paper, [Adding Error Bars to Evals](https://arxiv.org/abs/2411.00640) — paired differences as free variance reduction, clustered standard errors, and power analysis for eval sample sizes.
- [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — regression suites held near 100%, and using evals to adopt new models quickly.
