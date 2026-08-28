# Multi-turn & memory evaluation

Single-turn evals miss the failures that actually generate complaints. State accumulates: the agent contradicts what it said at turn 3, forgets the constraint the user gave at turn 2, re-asks for the account number, or quietly drops a requirement after summarisation. None of that is visible one turn at a time.

## Three ways to produce a conversation

| Method | Determinism | Catches | Cost |
| --- | --- | --- | --- |
| **Transcript replay** — fixed user turns, replayed against the agent | High | Regressions on known dialogues, memory recall, contradiction | Low |
| **Simulated user** — an LLM playing a persona with a goal and a stopping rule | Low | Branching, misunderstanding, recovery, unhappy paths you never scripted | High |
| **Live session review** — sampled real sessions, human- or judge-scored | n/a | The real distribution, which the other two only approximate | Ongoing |

**Replay is the regression tier.** It is deterministic enough to gate a release: same user turns every run, so a diff is a real change. Its weakness is that a fixed turn 5 may become nonsense if the agent's turn 4 changed — so assert on state and constraints, not on conversational flow, and re-record the script when the agent's behaviour legitimately changes.

**Simulation is the capability tier.** Give the simulated user a persona, a goal, hidden facts to reveal only when asked, and a stopping controller; then let it branch. This is the only cheap way to test how the agent handles a user who changes their mind, gives contradictory information, or gets annoyed. It is also non-deterministic and slow, which puts it in the nightly/pre-release tier ([CI & triggers](eval-in-ci.md)).

Reliability matters more here than anywhere else: simulation multiplies the variance of the agent by the variance of the simulated user. τ-bench found frontier models dropping to roughly 25% at pass^8 on tasks they pass most of the time individually — the same task, run eight times, all eight succeeding. Report **pass^k on sessions**, not average turn quality.

## Score the session, not the turns

Turn-level averages hide the failure. A session where turn 14 leaks the wrong customer's data still scores 93%.

Grade at session level:

- **Task completion** — did the goal get achieved, verified against **end state** (the booking exists, the refund is recorded), not the closing message.
- **Policy adherence across the whole session** — the rail must hold at every turn, including after the user pushes back three times ([guardrail evaluation](guardrail-evaluation.md)).
- **Consistency** — no contradiction of an earlier assertion.
- **Friction** — turns to completion, repeated questions, number of clarifications. A "successful" 20-turn resolution of a 3-turn task is a failure the outcome check will call a pass.
- **Recovery** — after a user correction ("no, the other account"), does the agent actually switch, and stay switched?

## Testing memory, concretely

Split by horizon, because they fail for different reasons ([memory & context engineering](../02-design/memory.md)).

**Short-term (within a session):**

| Test | Construction | Failure it catches |
| --- | --- | --- |
| **In-session needle** | Give a constraint at turn 2, require it at turn 20 with filler in between | Attention decay / context rot |
| **Correction** | State a fact, correct it later, then ask | Agent uses the stale value |
| **No re-asking** | Provide the account number once | Agent asks again — the top-ranked user complaint in support agents |
| **Post-compaction** | Force summarisation/truncation mid-session, then probe pre-compaction facts | The compaction step silently dropped a requirement |
| **Reference resolution** | "Do the same for the other one" | Ambiguous anaphora resolved wrongly and acted on |

**Long-term (across sessions):**

| Test | Construction | Failure it catches |
| --- | --- | --- |
| **Cross-session recall** | Session A states a preference; session B (days later) must use it | Memory never written, or never retrieved |
| **Knowledge update** | Session A: "I live in Lyon". Session C: "I moved to Berlin". Session D asks | Both facts retrieved, older one wins |
| **Temporal reasoning** | "What did I order last month?" over dated sessions | No time-awareness in retrieval |
| **Abstention** | Ask about something never mentioned in any session | Confabulation instead of "I don't have that" |
| **Scope isolation** | User B asks a question whose answer exists in user A's memory | Cross-tenant memory leak — a security bug, not a quality one ([security & compliance](../02-design/security-and-compliance.md)) |

These map onto LongMemEval's five abilities — information extraction, multi-session reasoning, temporal reasoning, knowledge updates, and abstention — which is a usable checklist even if you never run the benchmark itself. Its headline result is the reason to test at all: commercial assistants and long-context models lose around 30% accuracy on information carried across sustained interaction.

**Instrument the memory pipeline separately.** Memory is indexing → retrieval → reading. When a memory case fails, you need to know *which* stage failed: log what was written, what was retrieved for the query, and what reached the context. Grading only the final answer tells you memory is broken without telling you where — and the fix for a retrieval miss (query expansion, time-aware search) has nothing to do with the fix for a reading failure (the fact was in context and the model ignored it).

**Abstention deserves its own metric.** An agent that answers everything scores well on recall tests and lies to users about what it remembers.

## References

- [Sierra — τ-bench](https://sierra.ai/blog/benchmarking-ai-agents) and [the τ²/τ³-bench repo](https://github.com/sierra-research/tau2-bench) — LLM user simulator, database-state grading, pass^k.
- [Wu et al. — LongMemEval: benchmarking chat assistants on long-term interactive memory (ICLR 2025)](https://arxiv.org/abs/2410.10813) — the five memory abilities, the ~30% drop, and the indexing/retrieval/reading decomposition. [Code](https://github.com/xiaowu0162/LongMemEval).
- [DeepEval — conversation simulator](https://deepeval.com/docs/conversation-simulator) — persona, scenario, stopping controller, and multi-turn test cases in practice.
