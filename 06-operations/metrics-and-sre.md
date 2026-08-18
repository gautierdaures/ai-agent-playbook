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

Prompt changes are gated by [regression testing](../04-evaluation/regression-and-drift.md); cost detail lives in [cost management](../05-deployment/cost-management.md).
