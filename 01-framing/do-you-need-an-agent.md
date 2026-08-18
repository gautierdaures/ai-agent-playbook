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

Treating workflows and autonomous agents as interchangeable causes one of two failures: **over-engineering** a simple, well-understood task with needless autonomy, or **under-engineering** a genuinely open-ended problem by forcing it into a rigid pipeline that breaks the moment reality deviates. For most organisations, an AI-driven *workflow* — not a fully autonomous agent — hits the right balance today.

## Think in autonomy levels

It helps to see autonomy as a dial, not a switch. A common ladder:

- **L0 — none:** classic ML / rules for narrow tasks.
- **L1 — assistive:** a model handles a single step under direct instruction.
- **L2 — partial:** agentic workflow runs multi-step tasks with human oversight. *This is where most production value sits in 2025–2026.*
- **L3 — high:** goal-oriented, mostly self-directed, occasional guidance.
- **L4 — full:** end-to-end autonomy.

Start at the lowest level that solves the problem, and **widen autonomy gradually as the agent earns trust** through evaluation and safe rollout (see [scoping the first version](scoping-the-first-version.md) and [rollout & safety](../05-deployment/rollout-and-safety.md)).

## A caution on the market

Agentic AI is powerful but oversold. Gartner forecasts **over 40% of agentic AI projects will be cancelled by 2027** — mostly for unclear business value and weak governance, not model limitations. The antidote is framing discipline: the right use case ([use-case selection](use-case-selection.md)), honest feasibility ([feasibility & readiness](feasibility-and-readiness.md)), and a measurable value case ([measuring value](measuring-value.md)).

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
- [Anthropic — Measuring AI agent autonomy in practice](https://www.anthropic.com/research/measuring-agent-autonomy)
- [The 4 levels of AI agents: when to use workflows vs autonomous systems (Barnacle)](https://www.barnacle.ai/blog/2025-09-25-agents-intro)
