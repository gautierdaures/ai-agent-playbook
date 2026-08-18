# Do you actually need an agent?

Start from the business problem, not the technology. An agent is not always the right tool; often a simpler, more deterministic design is cheaper and more reliable.

## Deterministic chatbot vs. agent

- **Deterministic chatbot** — a decision tree with no access to external tools. For example, it cannot grant a credit in the real world; it can only follow branches.
- **Agent** — can perceive, reason, and act on the world through tools. Granting a credit "for real" requires an agentic system with tool access.

The internal makeup of an agent (perception, reasoning, action) is covered in [agent anatomy & prompting](../02-design/agent-anatomy-and-prompting.md).

## How to decide

There is no hard industry rule, but the prevailing guidance (see Anthropic's *Building Effective Agents*) is a ladder of increasing complexity — only climb a rung when the one below demonstrably falls short:

1. **Single LLM call** — a prompt, optionally with retrieval and a few in-context examples. Enough for most tasks.
2. **Workflow** — LLM and tool calls orchestrated through *predefined code paths* (prompt chaining, routing, parallelization). Predictable and consistent; use it when the steps are known in advance.
3. **Agent** — the *model* decides the next step and when to stop. Use it only when you cannot hardcode the path: open-ended problems where the number of steps is hard to predict.

Rules of thumb:

- **Prefer the most deterministic option that works.** Determinism buys predictability, lower latency, lower cost, and easier evaluation.
- **Go agentic when the path is genuinely unknown** and needs model-driven decisions at run time.
- **Mind the trade-off** — agents trade latency and cost for flexibility. Make sure that trade is worth it.

## Start simple, go multi-agent only when justified

Prefer starting with a single simple agent. Move to multi-agent only when:

- there are several distinct domains of expertise;
- complexity is genuinely high;
- there is heavy parallelization to exploit.

When a task naturally splits into several sub-questions, that is a signal for a sub-agent or multi-agent design. See [multi-agent orchestration](../02-design/multi-agent-orchestration.md).

## Keep the human in the loop

The agent produces a report, not a verdict. Especially when the outcome touches people, a human makes the final call.

## References

- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
