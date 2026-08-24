# Cost management

Cost is a live production concern, not a one-off estimate: token spend moves with traffic, model choice, and prompt changes, so it is watched and controlled continuously in operations. But it is **decided long before it is watched** — most of a system's unit cost is fixed by design choices, not by tuning afterwards. This note is the canonical home for cost across the whole lifecycle: the design-time levers first, then running the system economically.

## Cost across the lifecycle

| Phase | What happens to cost |
| --- | --- |
| [Framing](../01-framing/scoping-the-first-version.md) | Set the **budget**: a target cost per task and a TCO projection at full scale, anchored to the baseline cost of doing the task today ([measuring value](../01-framing/measuring-value.md)). Check it is achievable at the target volume ([feasibility & readiness](../01-framing/feasibility-and-readiness.md)). |
| [Design](../02-design/architecture.md) | **Yes — cost is a design constraint.** Architecture decides the number of LLM calls, how much context each one carries, and which model runs them. Record cost per run as a non-functional budget in the [technical framing doc](../02-design/from-framing-to-technical-design.md). |
| [Build](../03-build/README.md) | Measure the real cost per run as soon as anything works, and compare it with the design estimate. |
| [Evaluation](../04-evaluation/evaluation-methods.md) | Report cost per run alongside quality — a 2-point quality gain for 3× the cost is a decision, not an improvement. |
| [Deployment](../05-deployment/rollout-and-safety.md) | Ship with a spend cap and an alert before widening traffic. |
| Operations | Track live spend, hunt waste, and keep unit cost flat as volume grows. |

## Cost as a design constraint

The levers that matter are structural, and they are cheap to choose and expensive to retrofit:

- **Autonomy level** — an agent that decides its own path costs several times a fixed workflow for the same job, because every extra loop iteration is another full-context call. This is the strongest single lever ([do you need an agent?](../01-framing/do-you-need-an-agent.md)).
- **Number of LLM calls per run** — a step that neither calls a tool nor changes state usually does not need a model call at all.
- **Context size per call** — retrieval `top_k`, how much history is carried, how large the tool definitions are. Context is paid for on *every* turn, so a bloated system prompt is a recurring bill ([memory & context engineering](../02-design/memory.md)).
- **Model routing** — a cheap model for classification and extraction, the frontier model only for the steps that need it ([architecture — evolvability](../02-design/architecture.md)).
- **Caching** — prompt caching for the stable prefix, semantic caching for repeat queries (below). Both are design decisions: they need a stable prompt layout to work.
- **Stop conditions and caps** — `max_steps`, timeouts, and a per-run token budget bound the worst case. Without them one pathological run can cost more than a thousand normal ones ([error handling](../02-design/error-handling.md)).
- **Human review cost** — the reviewer's time is part of unit cost. A design that escalates 40% of runs may be more expensive overall than the model bill suggests ([human-in-the-loop](../02-design/human-in-the-loop.md)).

### Estimate before you build

Back-of-envelope arithmetic at design time is enough to catch an unaffordable design, and takes ten minutes. Sketch the run, then multiply:

```
per run:  3 LLM calls
          input  ~6,000 tokens/call  ->  18,000 in
          output   ~800 tokens/call  ->   2,400 out

cost/run  = 18,000/1e6 x $3   +  2,400/1e6 x $15   =  $0.054 + $0.036  ~=  $0.09
monthly   = $0.09 x 4,000 runs                                          ~=  $360
```

Then compare with the two numbers framing gave you: the **target cost per task** and the **baseline** cost of a human doing it (11 min at a loaded rate is a couple of euros — so $0.09 leaves room, and the human review step, not the tokens, is what to watch). If the estimate lands within ~2× of the target, the design is viable and the work is tuning; if it is 10× off, change the design, not the prompt.

> **Note:** Use the model's current pricing page for the rates — they change often. The point of the arithmetic is the *order of magnitude* and the comparison between two candidate designs, not a precise forecast.

## Master your LLM costs in production

Track, per run:

- tokens in;
- tokens out;
- number of LLM calls;
- which model is used.

Attribute each of those to a run id, a prompt version, and a use case, so a spend increase can be traced to the change that caused it ([monitoring & observability](monitoring-and-observability.md)). Watch the **p95 cost per run**, not just the mean — runaway loops hide in the tail.

> **Caution:** Audio and video inputs are expensive; watch them closely.

## No useless LLM calls

- If an agent or LLM step does not call a tool and does not modify the state, question whether the LLM call is needed at all.
- Route easy steps to a cheaper model; reserve the frontier model for the steps that need it (see [architecture — evolvability](../02-design/architecture.md)).

## Semantic caching

A **semantic cache** returns the result of a similar previous query instead of recomputing it — matching on embedding similarity rather than an exact key. It cuts both latency and cost by removing repeat calls for near-duplicate queries. It is the highest-leverage single optimisation for a high-traffic agent with repetitive inputs.

Cost per run is also a [production metric](metrics-and-sre.md) — track it against the baseline cost per task from [measuring value](../01-framing/measuring-value.md).
