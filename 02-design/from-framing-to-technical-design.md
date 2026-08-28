# From business framing to technical design

Framing ([section 1](../01-framing/README.md)) answers *what* and *why*. Design ([section 2](README.md)) answers *how*. Between them sits a translation step that is easy to skip and expensive to skip: turning the framing artefacts — the use-case card, the one value lever, the success criteria — into technical decisions. This note is that bridge.

## What framing hands over

Before you draw a single box, gather the outputs framing already produced. If any are missing, you are not ready to design — go back:

- **The job in plain terms** and the reformulated steps ([project framing](../01-framing/project-framing.md), meetings 1–2).
- **The one primary value lever** and the **baseline** it improves on ([measuring value](../01-framing/measuring-value.md)).
- **Success criteria expressed as numbers**, plus the definition of "done" ([scoping the first version](../01-framing/scoping-the-first-version.md)).
- **The autonomy level** you committed to — single call, workflow, or agent ([do you need an agent?](../01-framing/do-you-need-an-agent.md)).
- **Feasibility notes** — what data and which systems the agent may touch ([feasibility & readiness](../01-framing/feasibility-and-readiness.md)).
- **Non-negotiables** — compliance boundaries, human review gates, the RACI ([project framing](../01-framing/project-framing.md)).

## The translation

Each framing output forces a concrete design question. Work down the list; the right-hand column is where you spend section 2.

| Framing output | Design question it forces | Where it's answered |
| --- | --- | --- |
| The job, decomposed into steps | Which steps are LLM reasoning vs. deterministic code? | [architecture](architecture.md), [multi-agent](multi-agent-orchestration.md) |
| The one value lever | What does the architecture optimise for — latency, quality, or cost? | [architecture](architecture.md), [cost management](../06-operations/cost-management.md) |
| Success criteria as numbers | What is the eval set and which metrics gate a release? | [evaluation methods](../04-evaluation/evaluation-methods.md) |
| Autonomy level | Agent vs. workflow; how much the control loop decides; how tight the guardrails | [architecture](architecture.md), [error handling](error-handling.md) |
| Data readiness | Short-term vs. long-term memory; RAG or not | [memory](memory.md), [RAG & tooling](../03-build/rag-and-tooling.md) |
| Tool / system access | The tool inventory, MCP surface, and permission scope per tool | [RAG & tooling](../03-build/rag-and-tooling.md), [security & compliance](security-and-compliance.md) |
| Compliance non-negotiables | PII boundaries, audit trail, where humans must sign off | [security & compliance](security-and-compliance.md) |
| RACI & review gates | Where the human-in-the-loop sits inside the flow | [project framing](../01-framing/project-framing.md) |

## How to run it: one working session

The translation is a meeting with an artefact, not a solo write-up. It follows framing meeting 3 ([project framing](../01-framing/project-framing.md)) and takes about half a day.

1. **Before (30 min, tech lead).** Paste the use-case card, the success criteria, and the feasibility notes into an empty technical framing doc. Anything you cannot paste is a gap — list it as an open question on the spot.
2. **Step decomposition (45 min, with the domain expert).** Write the job as a numbered list of steps *as a human does it today*. Then mark each step `LLM` (judgement, language, ambiguity), `CODE` (deterministic rule, lookup, calculation), or `HUMAN` (irreversible, regulated, or low-confidence). Rule of thumb: if you can write the rule down without the word "usually", it is `CODE`, not `LLM`. This marked list *is* your architecture draft — `CODE` steps become tools, `LLM` steps become the agent's turns, `HUMAN` steps become gates.
3. **Tool & data inventory (30 min, with data/engineering).** For every `CODE` step, name the system, the call (read or write), whether it is reversible, and who owns access. A step with no reachable system is a blocker, not a detail.
4. **Agent spec(s) (45 min).** Fill the template below — one per agent. Start with exactly one.
5. **Budgets & risks (30 min).** Turn the success criteria into per-run numbers (latency p95, cost per run, escalation rate), and list the top risks with a mitigation each.
6. **After (1 h).** Finish the doc, circulate it to the framing group, and get an explicit "yes, this is the thing we asked for" from the business owner before code starts.

### Worked example

Take a common first use case: **level-1 support tickets — classify the ticket, gather account context, draft a reply for a human agent to approve.** Framing produced: value lever = handling time (baseline 11 min/ticket, ~4,000 tickets/month); success criteria = correct category ≥90% on 300 past tickets, draft accepted without edit ≥60%, escalation on low confidence; autonomy = workflow with a human gate; non-negotiable = no PII leaves the EU region, no reply sent without human approval.

The decomposition and its marking:

| # | Step (as done today) | Marked | Becomes |
| --- | --- | --- | --- |
| 1 | Read the ticket, classify it | `LLM` | one model call with the category taxonomy in the prompt |
| 2 | Look up the customer and their contract tier | `CODE` | `get_account(ticket_id)` tool |
| 3 | Check for similar past tickets | `CODE` + `LLM` | retrieval over the ticket archive ([RAG & tooling](../03-build/rag-and-tooling.md)), model picks what is relevant |
| 4 | Draft the reply | `LLM` | one model call, output constrained to a schema |
| 5 | Send the reply | `HUMAN` | approval gate; the agent never sends ([human-in-the-loop](human-in-the-loop.md)) |

Which settles the design questions: it is a **fixed workflow, not an agent** (the step order never varies — [do you need an agent?](../01-framing/do-you-need-an-agent.md)); memory is **session-only** plus a retrieval index, no long-term store; three tools with read-only scope; the EU constraint fixes model region and forbids the ticket body in logs ([logging](../06-operations/logging.md)); the eval set is the 300 historical tickets, and the release gate is the ≥90% category accuracy on them.

## The agent spec — the smallest unit of design

Before the system diagram, pin down each agent as a short spec. Elaborate personas are rarely needed; a role, a goal, and constraints usually suffice. Capture:

- **Role & goal** — who the agent is and the single outcome it owns.
- **Input / output contract** — what it receives and the exact shape it returns (for example a Pydantic schema — see [agent anatomy & prompting](agent-anatomy-and-prompting.md)).
- **Tools and permission scope** — what it may call, and with what least-privilege rights ([security & compliance](security-and-compliance.md)).
- **Stop conditions** — when it is done, and the hard limits (max steps, budget, timeout) that stop a runaway loop ([error handling](error-handling.md)).
- **Success criteria** — the numbers from framing, restated at the agent level so they become evals.

Keep it in the repository next to the code, in a format you can diff — the spec changes as the agent does. A filled example for the support workflow above:

```yaml
agent: ticket_triage
role: >
  Classifies an inbound support ticket and drafts a reply for a human agent to approve.
goal: a correctly categorised ticket with a draft reply, or an explicit escalation.
input:
  ticket_id: str
  ticket_body: str          # EU region only; never written to logs
output:                     # enforced by a Pydantic model, not by asking nicely
  category: enum[billing, technical, account, other]
  confidence: float         # 0-1
  draft_reply: str | null   # null when escalating
  sources: list[str]        # ids of the past tickets used
tools:
  - get_account(ticket_id) -> Account        # read-only, CRM
  - search_past_tickets(query, k=5) -> list  # read-only, archive index
stop_conditions:
  - output validates against the schema
  - max_steps: 6
  - timeout_s: 30
  - confidence < 0.7 -> escalate, no draft
guardrails:
  - never sends a reply; approval gate owns that action
  - PII redacted before the retrieval call
success_criteria:
  - category accuracy >= 90% on the 300-ticket eval set
  - draft accepted unedited >= 60%
  - p95 latency <= 8 s, cost per run <= EUR 0.05
```

Two things make this useful rather than decorative: every `success_criteria` line is a number you can run an eval against, and every `tools` line has an owner who has confirmed the access exists.

## Write it down: the technical framing doc

The artefact of this step is a short **solution-design / technical-framing doc** — treat it like a PRD or system-design doc, with acceptance criteria and KPIs carried straight over from framing. Aim for 3–6 pages; if it is longer, you are designing beyond the first version. A workable outline:

| Section | What goes in it | Done when |
| --- | --- | --- |
| 1. Context & value lever | Two paragraphs from the use-case card; the baseline number | A reader outside the project can say what the agent is for |
| 2. Scope | In / out of scope for v1, and what is deliberately deferred | The "out" list is non-empty |
| 3. Step decomposition | The marked table (`LLM` / `CODE` / `HUMAN`) | Every step has exactly one mark and an owner |
| 4. Agent spec(s) | One block per agent, as above | Each spec's criteria are numbers |
| 5. Architecture | The pattern chosen and *why*, with the component diagram ([architecture](architecture.md)) | The "why" cites a criterion, not a preference |
| 6. Tools & data | Tool inventory with scope, data sources, retention, PII handling | Access confirmed by the owning team |
| 7. Non-functional budgets | Latency p95, cost per run ([cost management](../06-operations/cost-management.md)), availability, throughput | Each is a figure with a unit |
| 8. Eval plan | The eval set, its size and provenance, the release gate ([evaluation methods](../04-evaluation/evaluation-methods.md)) | The set exists, or building it is a task with a date |
| 9. Risks & open questions | Top risks, mitigation, owner, decision date | No risk is owned by "the team" |

Keep it traceable back to the one-page use-case card so a reviewer can follow a design choice all the way to the business reason for it. A practical test: pick any line in section 5 and ask "which framing output forced this?" — if there is no answer, it is over-design.

This doc is also what build starts from: [from design to code](../03-build/from-design-to-code.md) picks up the handover.

## Don't over-design

Start with the simplest thing that can hit the success criteria, and escalate only when the numbers demand it: a single call, then a fixed workflow, then an agent, then multiple agents. Autonomy is a cost, not a feature ([do you need an agent?](../01-framing/do-you-need-an-agent.md)). The technical framing doc should justify each step up that ladder with a measurement, not a preference.

Concretely, in the example above: a single classification call scores 82% on the eval set — under the 90% gate, so it earns the step up to a workflow that first retrieves similar past tickets, which reaches 93%. That measurement is the justification, and it belongs in section 5 of the doc. Adding a second "quality reviewer" agent on top, with no failing metric pointing at it, is not.

## References

- [How to Write a Good Spec for AI Agents](https://www.oreilly.com/radar/how-to-write-a-good-spec-for-ai-agents/) (O'Reilly) — treating the agent spec as a structured, outcome-grounded document.
- [Choose a design pattern for your agentic AI system](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system) (Google Cloud Architecture Center) — mapping requirements to an architecture pattern.
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — start simple, add autonomy only when it pays for itself.
