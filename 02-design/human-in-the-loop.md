# Human-in-the-loop

Autonomy is granted per action, not once for the whole agent. Where a human sits inside the loop is a design decision that follows straight from the autonomy level you set in framing ([do you need an agent?](../01-framing/do-you-need-an-agent.md)) and the accountability rule that **an agent is never Accountable** ([project framing](../01-framing/project-framing.md)). This note is how you make the human-in-the-loop (HITL) principle structural rather than aspirational.

## Three oversight patterns

- **Pre-execution approval** — the agent pauses before a consequential action and asks for explicit confirmation. Use it for irreversible or high-blast-radius actions (sending, publishing, paying, deleting).
- **Post-execution review** — the agent acts but surfaces the result for inspection before it commits or continues. Use it when acting is cheap to undo but you still want a check.
- **Escalation triggers** — the agent runs autonomously and only halts to ask for input when a risk signal fires: sensitive data, an irreversible operation, or confidence below a threshold.

## Review the decision, not the whole run

Gating every action creates a bottleneck and trains reviewers to rubber-stamp. The stronger pattern is to intervene at **decision points**: when the agent reaches a fork, it presents the options with their trade-offs and a human chooses direction — then it executes. Put gates where the cost of being wrong is high, and let the rest run.

## Where to place a gate

Drive placement from the action's blast radius, not a blanket rule:

- **Irreversible or external side effects** — always gated (send, publish, transfer, delete).
- **Sensitive-data access** — gated or escalated ([security & compliance](security-and-compliance.md)).
- **Low model confidence** — routed to a human via an escalation trigger.
- **Routine, reversible reads** — no gate; gating them only adds latency.

## Make gates selective and durable

- **Selective** — gating everything defeats the point; reserve gates for the actions that warrant them.
- **Durable** — a synchronous "wait for a click" gate fails when the approver is away. Design an asynchronous path: queue the request, hold state, resume on approval, and time out safely (fail closed on irreversible actions). This ties to the agent's [stop conditions and state](architecture.md). In build, this is what turns a run into a resumable workflow — see [state & execution](../03-build/state-and-execution.md).

## Progressive autonomy

Autonomy can be earned. Start supervised, measure the approval rate per action type, and raise the auto-approve threshold for an action once it has proven itself (for example, a high sustained approval rate over a large sample). This lets a system tighten oversight where it's risky and relax it where it's boring — but only on evidence from [production monitoring](../06-operations/metrics-and-sre.md).

## References

- [Human-in-the-Loop Patterns: Approval, Input, and Escalation Workflows](https://understandingdata.com/posts/human-in-the-loop-patterns/) — the pre-execution / post-execution / escalation split.
- [Human-in-the-Loop Escalation Design for AI Agents](https://www.digitalapplied.com/blog/human-in-the-loop-escalation-design-ai-agents-2026) — reviewing the decision not the run; selective, durable gates; progressive autonomy.
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — the risk basis for tiering oversight by autonomy level.
