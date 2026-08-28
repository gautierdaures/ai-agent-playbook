# Evals in CI: triggers & cost control

Evals are slow, non-deterministic, and cost real money — so running the full suite on every commit is not an option, and running it only before a release is how prompt regressions reach production. The workable rule: **trigger on what changed, gate on what's risky, cost-bound everything.**

## What runs when

| Change | What runs | Where |
| --- | --- | --- |
| App code not touching prompts, tools, or retrieval | Unit / loop / guardrail tests only, no model ([testing agent code](../03-build/testing-agent-code.md)) | Every PR |
| **Prompt change** (system prompt, template, few-shot examples) | Regression suite for the affected agent + blocking guardrail suite | Every PR |
| **Tool definition change** — name, description, parameter docs, schema | Tool-selection eval + regression suite. Descriptions are prompt: an edit changes behaviour | Every PR |
| **Retrieval change** — chunking, embedding model, index, reranker | Retrieval metrics (recall@k, empty-result rate) then end-to-end | Every PR |
| **Guardrail / policy change** | Full guardrail suite, both directions ([guardrail evaluation](guardrail-evaluation.md)) | Every PR, blocking |
| **Model version change** | Everything: capability + regression + guardrails + judge re-validation | On demand, pre-release |
| **Memory or context config** (compaction threshold, retention, summarisation) | Multi-turn & memory suite ([multi-turn & memory](multi-turn-and-memory.md)) | On demand, nightly |
| Nothing — scheduled | Full suite nightly; simulation and adversarial suites weekly | Cron |

## Detecting "the prompt changed" reliably

Path filters (`paths: ['prompts/**']`) are the standard trigger and they work — until the prompt changes without any file under `prompts/` changing: a template variable, a config default, a tool description in another module, a model version bump in an env var, a system-prompt fragment pulled from a database.

The robust version: **render the artefacts and hash them.** In CI, assemble the full system prompt and the serialised tool definitions exactly as the runtime would, hash the result, and commit the hashes as snapshots. Then:

- The hash changing **is** the trigger — regardless of which file caused it.
- The snapshot diff is a readable review artefact: a reviewer sees the actual prompt delta, not a config diff they have to mentally compile.
- A prompt change that nobody realised was a prompt change stops being possible.

This extends the tool-definition snapshotting from [testing agent code](../03-build/testing-agent-code.md) into the eval trigger.

## Tier the suites

| Tier | Size | Budget | Trigger |
| --- | --- | --- | --- |
| **Smoke** | 10–30 cases, 1 trial | < 5 min, a few cents | Every PR that trips a hash |
| **Regression** | Full known-good set, 3 trials | 15–45 min | Merge to main, pre-release |
| **Capability** | Hard set | Nightly | Cron, model changes |
| **Expensive** — simulation, adversarial, multi-turn memory | Hours | Nightly/weekly + on demand via a PR label or `/eval full` comment | Pre-release, model swap |

Two rules that keep this honest:

- **Cut cases, never the model.** Running the smoke tier against a cheaper model measures a different system and gives a green light for something you are not shipping.
- **Never gate on a single trial.** Any suite whose result gates a merge runs k trials and reports the spread, or it will page people about noise until they stop looking ([evaluation methods](evaluation-methods.md#reliability-one-run-tells-you-nothing)).

## Keeping the bill down

- **Cache model responses** keyed on (prompt, params, model). Reruns after a test-harness fix then cost nothing, and identical cases across suites are paid for once.
- **Batch API for anything not blocking a human.** Nightly and capability runs have no latency requirement and batch pricing is materially cheaper ([cost management](../06-operations/cost-management.md)).
- **Fail fast on the blocking rails** before spending on judge-scored dimensions — if a blocking policy failed, the rest of the score is irrelevant.
- **Cap concurrency to your rate limits** rather than discovering them as flaky failures; a suite that intermittently 429s is a suite people learn to re-run instead of read ([alerting & anomaly detection](../06-operations/alerting-and-anomaly-detection.md)).
- **Budget the suite explicitly.** "The eval suite costs €X per full run and runs Y times a day" belongs in the cost model, not as a surprise line item.

## Make the result reviewable

A pass rate in a CI log changes nobody's mind. What a reviewer needs on the PR:

- **The delta against the baseline**, per suite, with the confidence interval ([regression & drift](regression-and-drift.md)).
- **The list of cases that flipped**, in both directions — a case that started passing is as informative as one that broke.
- **A link to the failing traces**, so diagnosis starts one click away.

Posting that as a PR comment is what turns evals from a gate people route around into a review artefact people argue about.

## References

- [promptfoo — GitHub Action](https://www.promptfoo.dev/docs/integrations/github-action/) — path-filtered triggering on prompt changes, before/after comparison posted to the PR, and response caching.
- [promptfoo — CI/CD integration](https://www.promptfoo.dev/docs/integrations/ci-cd/) — pass thresholds and pipeline wiring.
- [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — automated evals in CI/CD as the first line of defence on each agent change and model upgrade.
