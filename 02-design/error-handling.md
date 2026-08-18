# Error handling

Design failure modes in up front. Four common ones and their mitigations:

- **Hallucinations** — grounding; require a valid or validated output.
- **Infinite loops** — timeout, max iterations, and an expert debate to break ties.
- **Tool errors** — retry with a fallback, plus proper error handling.
- **Context overflow** — summarize, prune, reflection, memory.

Additional notes:

- Always have an absolute stop condition. In production that means timeout, max iterations, and max items; see [rollout & production safety](../05-deployment/rollout-and-safety.md).
- Expert debate, several agents arguing, is also useful as an [evaluation technique](../04-evaluation/evaluation-methods.md).
