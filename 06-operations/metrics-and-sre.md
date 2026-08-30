# Key metrics & SRE follow-up

## Key metrics

- Latency
- Success rate
- Cost per run
- Hallucination rate
- Average number of iterations
- Coverage per tool and per agent

## SRE follow-up

- Follow-up metrics for SRE: success rate, cost, API coverage, and number of calls per run or session, versus what was anticipated for the LLM.
- Retry and fallback, for example on the run or on the ranker.

Turning these metrics into alerts that fire usefully is [alerting & anomaly detection](alerting-and-anomaly-detection.md). Prompt changes are gated by [the release gate](../04-evaluation/regression-and-drift.md); cost detail lives in [cost management](cost-management.md); the retry & fallback pattern is designed in [error handling](../02-design/error-handling.md).
