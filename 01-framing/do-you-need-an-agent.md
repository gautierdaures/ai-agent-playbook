# Do you actually need an agent?

Start from the business problem, not the technology.

## Deterministic chatbot vs. agent

- **Deterministic chatbot** — a decision tree with no access to external tools. For example, it cannot grant a credit in the real world; it can only follow branches.
- **Agent** — can perceive, reason, and act on the world through tools. Granting a credit "for real" requires an agentic system with tool access.

An LLM agent is built from three parts:

- **Perception** — text and memory (context).
- **Reasoning** — analyze, plan, reflect.
- **Action** — call tools, produce output.

## Start simple, go multi-agent only when justified

Prefer starting with a single simple agent. Move to multi-agent only when:

- there are several distinct domains of expertise;
- complexity is genuinely high;
- there is heavy parallelization to exploit.

When a task naturally splits into several sub-questions, that is a signal for a sub-agent or multi-agent design. See [multi-agent orchestration](../02-design/multi-agent-orchestration.md).

## Keep the human in the loop

The agent produces a report, not a verdict. Especially when the outcome touches people, a human makes the final call.
