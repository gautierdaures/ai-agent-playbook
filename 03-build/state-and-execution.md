# State & execution

Where the run's state lives, and how a run actually executes in a process. This is the part of the build that decides whether a run can be paused for a human, resumed after a crash, or cancelled when the user closes the tab — and it is far easier to get right at the start than to retrofit around a working loop.

## Where state lives

Three horizons, deliberately separated:

- **Working state** — the run's beliefs, goals, and intermediate results, held in a semi-structured object for the duration of the run ([architecture — BDI](../02-design/architecture.md)).
- **Session state** — what persists between turns of a conversation: history, summary, the user's current task. A cache or database (Redis, Postgres), keyed by session.
- **Long-term memory** — what persists across sessions: the user profile, durable facts, embeddings. A design decision, not a default ([memory](../02-design/memory.md)).

Keep the working state **serialisable**. That single constraint is what makes checkpointing, resuming, and debugging possible; a loop that hides state in local variables and closures can do none of them.

## Checkpoint every step

Persist the working state after each step of the loop, keyed by run id. It costs a write per step and buys four things:

- **Resume after a crash or deploy** instead of restarting the run — and re-running a completed step means re-doing its side effects.
- **Pause for a human**, which is what an approval gate actually is (below).
- **Debugging by replay** — reload the state as of step 4 and step through what the agent saw.
- **Cancellation** — a stopped run leaves a coherent record rather than a half-written world.

Record with each checkpoint what you will want during an incident: model, prompt version, tool calls and results, tokens and cost, retrieved `source_id`s ([monitoring & observability](../06-operations/monitoring-and-observability.md)) — and no PII ([logging](../06-operations/logging.md)).

## A human gate is a pause, not a blocking call

This is the design decision that most often surprises teams at build time. If the flow is *agent proposes → human approves → agent executes*, the run may sit idle for hours or overnight. You cannot hold an HTTP request, a database transaction, or an in-memory Python object open across that wait.

So a gate turns the run into a **durable, resumable workflow**: the agent runs to the gate, checkpoints, and exits. The approval — from a UI, an email link, a Slack action — resumes the run from that checkpoint in a fresh process. Everything the continuation needs must be in the checkpoint, and every gate needs a timeout policy: what happens to a proposal nobody approves by Friday ([human-in-the-loop](../02-design/human-in-the-loop.md)).

## Execution shapes

| Shape | Fits | Notes |
| --- | --- | --- |
| **Synchronous request/response** | Short runs, roughly under 10 s | Simplest; a single slow tool blows the budget |
| **Streaming** | Chat and copilot UIs | Same runtime, better perceived latency: stream tokens and tool-call progress. Does not make the run faster, only visible |
| **Background job + poll/webhook** | Multi-minute runs, anything with a gate | Submit returns a run id; the client polls or is called back. The default for real agents |
| **Batch / scheduled** | Queues of work with no user waiting | Cheapest per run: high concurrency, off-peak, and no latency budget to defend |

Most production agents end up in the third row, and the migration from the first is invasive — pick it early if the design has a gate or a long tail.

## Concurrency, timeouts, cancellation

- **Three levels of timeout**: per tool call, per model call, per run. Only the last one bounds the worst case, and it is the one usually missing.
- **Cap concurrency per dependency**, so one slow system cannot occupy every worker ([integrations & auth](integrations-and-auth.md)).
- **Propagate cancellation** — when a run is cancelled or times out, in-flight tool calls should be cancelled too, not left to complete against a run that no longer exists.
- **Bound the queue**, and shed load rather than accepting work you cannot serve within the latency budget.
- **Parallelise independent steps** (several retrieval calls, several read tools) — usually the largest latency win available, and free if the steps are genuinely independent.

## Retries at the run level

Distinguish the two:

- **Resume** — continue from the last checkpoint. Correct for infrastructure failures: no completed side effect is repeated.
- **Restart** — run again from the beginning. Only safe when every write is idempotent, or the run has no side effects.

Make it explicit per run type; guessing produces the duplicate-refund class of bug. And cap automatic run-level retries: an expensive run retried three times is three times the cost, with the same failure at the end ([cost management](../06-operations/cost-management.md)).

## References

- [LangGraph — persistence](https://docs.langchain.com/oss/python/langgraph/persistence) — checkpointers, threads, resuming, and human-in-the-loop interrupts.
- [Temporal — understanding durable execution](https://docs.temporal.io/evaluate/understanding-temporal) — the general pattern for workflows that pause, resume, and survive process death.
- [The Twelve-Factor App — processes & disposability](https://12factor.net/processes) — why state belongs in a backing store, not the process.
