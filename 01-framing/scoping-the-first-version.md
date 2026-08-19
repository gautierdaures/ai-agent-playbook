# Scoping the first version

The last framing decision is how *small* to make version one. The goal of the first project is not a finished platform — it is **a working production agent and a team that now knows how to ship the next three.** Keep it narrow.

## Stages, and re-framing at each gate

Yes — run the work in stages, and re-frame (lightly) at each gate rather than framing once and hoping:

- **PoC (proof of concept)** — does the approach work at all on real cases? Throwaway is fine. Frame it around one question and a pass/fail bar.
- **MVP / pilot** — the smallest version real users run on live work, instrumented for value and safety. This is the scope this note is mostly about.
- **Production** — hardened, integrated, rolled out with monitoring and support.

Between stages, hold a short **stage-gate**: re-confirm the use case still pays ([measuring value](measuring-value.md)), the data and integration assumptions held ([feasibility & readiness](feasibility-and-readiness.md)), and the scope for the next stage. Framing is not repeated in full each time — it *narrows and sharpens* with the evidence you now have. Kill or pivot here if the evidence says so; that is the gate doing its job, not a failure.

## Narrow beats ambitious

Most first agent projects die in "pilot purgatory" — an estimated **88% of AI proof-of-concepts never reach production** ([IDC / Lenovo, via CIO](https://www.cio.com/article/3850763/88-of-ai-pilots-fail-to-reach-production-but-thats-not-all-on-it.html); for every 33 POCs launched, only four graduate) — often because the scope was too broad to ever finish. Counter it deliberately:

- **One use case, one workflow, one clear input and output.** A multi-use-case, multi-vendor pilot is not a proof of concept; it is a procurement project.
- **Choose a workflow a human already does today**, so you inherit a baseline and real historical cases to test on.
- **Time-box it: 4–6 weeks, with a hard stop** and defined phases (setup → build → test → evaluate). The deadline creates urgency and forces scope discipline.

## Define "done" as numbers, before writing code

Agree the success criteria up front, in concrete figures — not "it works well":

- **Capability target** — success rate on real historical cases (e.g. "≥90% on 200 past tickets").
- **Safety target** — acceptable escalation / human-override rate and maximum tolerable error.
- **Operational target** — acceptable latency and **cost per task**, and a **TCO** (total cost of ownership — build, run, LLM/API usage, human review, and maintenance) projection at full scale.

The question a modern agent PoC must answer is no longer "does the model respond well?" but **"can the system reliably carry the work to completion without constant human intervention?"** Write the pass/fail line down before you start, so the review is evidence, not opinion.

## Design the safe path from day one

Even in a narrow pilot, decide:

- Every system the agent reads from and writes to (from [feasibility & readiness](feasibility-and-readiness.md)).
- **How it fails safely** — fallbacks, timeouts, and where a human takes over. See [error handling](../02-design/error-handling.md).
- A **staged rollout** that widens autonomy gradually as evidence accumulates:

  1. **Shadow mode** — the agent runs alongside the human, output compared, nothing shipped.
  2. **Human-in-the-loop** — the agent proposes, a human approves before anything takes effect.
  3. **Supervised autonomy** — the agent acts, humans audit a sample and handle exceptions.
  4. **Autonomous** — only for proven, low-risk paths.

See [rollout & safety](../05-deployment/rollout-and-safety.md) for the deployment mechanics.

## Plan the value capture, not just the demo

A technically successful pilot still fails the business if the value evaporates. Before scaling, confirm:

- The **baseline** is recorded so the delta is provable (see [measuring value](measuring-value.md)).
- The **earned time / capacity has a named destination** — reallocation, absorbed volume, or turnaround — or the saving is phantom.
- **Adoption** is designed for, not assumed (see [adoption & change management](../05-deployment/adoption-and-change-management.md)).

> **Caution:** Expect roughly **50–60% of pilot projections to materialise in production** — the gap is integration complexity, change management, and adoption, not the model. Budget for the pilot-to-production step; don't treat a green pilot as a finished result.

## References

- [How to scope your first AI agent project (Gaper)](https://gaper.io/how-to-scope-ai-agent-project)
- [AI pilot projects: PoCs that actually prove something (agility-at-scale)](https://agility-at-scale.com/ai/strategy/pilot-projects-and-proof-of-concept/)
- [AI agent proof-of-concept checklist (Gravity)](https://gravity.fast/blog/ai-agent-proof-of-concept-checklist/)
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
