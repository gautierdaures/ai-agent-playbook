# Tool design

Tools are the agent's hands — and designing them well is the highest-leverage decision in the whole system. A tool is a **contract between a deterministic system and a non-deterministic agent**: unlike a normal API with a known caller, the agent may misread the tool, call it at the wrong time, or pass odd arguments. Design for that. This note is about *designing* tools; for wiring and infrastructure see [RAG & tooling](../03-build/rag-and-tooling.md).

## The model reads your tool definitions

Every word in a tool's **name, description, and parameter docs** shapes how the agent uses it — these are prompt, not just documentation. Be explicit about *when* to use the tool, which parameters are required vs. optional, and what the output looks like. Vague or overlapping descriptions are a leading cause of wrong tool calls. Treat the description as something you iterate on against evals, not write once.

## Design for the agent, not your backend

- **Don't mirror your API 1:1.** A tool set that mirrors internal microservices forces the agent to orchestrate low-level calls. Build tools around the *tasks* the agent actually performs, consolidating several backend calls into one where that matches a real step.
- **Right granularity.** Too coarse and the tool is a black box the agent can't steer; too fine and the agent burns turns and context chaining calls. Aim for one tool per meaningful step in the [decomposition](from-framing-to-technical-design.md).
- **Few, distinct tools.** Many overlapping tools confuse the model about which to pick. Keep the set small and clearly separated; namespace related tools.

## Return high-signal, token-efficient output

Tool results land back in the context window, so they compete for the model's attention ([memory & context engineering](memory.md)). Return what the next decision needs — not a raw dump. Prefer identifiers and summaries over full payloads; let the agent fetch detail with a follow-up call when it needs it.

## Make errors actionable

When a tool fails, its return is the agent's only chance to recover. Return a clear, actionable error ("missing field `date`; expected ISO-8601") rather than a stack trace or a bare `500`. A good error message lets the agent self-correct; a bad one sends it into a [retry loop](error-handling.md).

## Validate inputs in code

The agent's arguments are non-deterministic, so **validate them in the tool, in code** (schema, regex, bounds) — never ask the model to sanitize its own input. Enforce least-privilege scope per tool here too. See [security & compliance](security-and-compliance.md).

## Iterate: prototype → evaluate → collaborate

Tool quality comes from iteration, not first draft. Prototype the tool, run the agent against realistic tasks, watch where it misfires ([evaluation methods](../04-evaluation/evaluation-methods.md)), and refine the description, schema, and return shape. You can even use an agent to critique and rewrite your tool definitions.

## References

- [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — tools as agent/system contracts; the prototype–evaluate–collaborate loop.
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — keeping the tool surface simple.
