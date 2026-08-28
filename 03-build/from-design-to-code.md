# From design to code

Design ([section 2](../02-design/README.md)) decides *what to build*; build turns it into a running system. The handover artefact is the [technical framing doc](../02-design/from-framing-to-technical-design.md) and the agent spec(s) inside it. This note is the entry point into section 3: what to build first, in what order, and how the repository is laid out.

## What design hands over

Do not start until you have, from the technical framing doc:

- the **step decomposition** with each step marked `LLM` / `CODE` / `HUMAN` (see below);
- an **agent spec** per agent — I/O contract, tools, stop conditions, guardrails, numeric success criteria;
- the **tool and data inventory**, with access confirmed by whoever owns each system;
- the **non-functional budgets** — latency p95, cost per run, availability;
- the **eval plan** and the eval set (or a dated task to build it).

A missing item is not a detail to sort out later: an agent spec without stop conditions produces a runaway loop, and a tool inventory without confirmed access produces a demo that cannot ship.

### The step decomposition is a table, not pseudocode

It is a design artefact in the technical framing doc — one row per step of the task, each marked with who executes it — and it is deliberately not code: it is written and argued over before anyone opens an editor, and non-engineers on the project have to be able to read it.

| # | Step | Who | Why |
| --- | --- | --- | --- |
| 1 | Normalise and deduplicate the incoming ticket | `CODE` | Deterministic, testable, cheaper than a model call |
| 2 | Classify into a category with a confidence | `LLM` | Genuine judgement over free text |
| 3 | Fetch account and entitlement | `CODE` | An API call with a fixed shape |
| 4 | Retrieve similar past tickets | `CODE` | Retrieval is a pipeline, not a decision |
| 5 | Draft a reply | `LLM` | Generation |
| 6 | Approve or edit the draft | `HUMAN` | Irreversible, customer-facing |
| 7 | Send and update the ticket | `CODE` | Deterministic write, idempotent |

The marking is the substance of the handover, because it records **what the model is allowed to decide**. Each mark then has a fixed translation into build:

- `CODE` → a plain function, exposed as a tool if the model calls it. It gets validation, a timeout, and a structured error, and it is unit-tested without a model.
- `LLM` → a model call against a versioned prompt file with a schema-validated output. This is what evals measure.
- `HUMAN` → a gate the run checkpoints at and resumes from, not a note in the doc ([state & execution](state-and-execution.md)).

The useful pressure it applies is downward: every step marked `LLM` that could have been `CODE` is latency, cost, and a new failure mode bought for nothing. Reviewing the table is mostly reviewing that column.

## Build the walking skeleton first

The instinct is to build the pieces well one at a time — the retrieval pipeline, then the tools, then the loop. Resist it. Build the **thinnest end-to-end path that produces a real output**, then deepen it:

> real input → real model call → one stubbed tool → validated output → logged trace

A day or two spent on that skeleton surfaces the risks that actually kill agent projects — an API that needs credentials nobody has, a response shape the model refuses to produce, a latency budget blown by one slow system — while they are still cheap to fix. A perfect retrieval pipeline in a system that cannot reach the CRM is worth nothing.

## Order of construction

1. **Contracts first.** Turn the agent spec's I/O into Pydantic models, and the categories and enums into types. Everything downstream refers to these ([RAG & tooling](rag-and-tooling.md)).
2. **Tool stubs.** Implement each tool's signature and return shape with hardcoded data. The agent can now run end to end.
3. **The loop with its stop conditions.** `max_steps`, timeout, and budget from day one — not after the first runaway run ([error handling](../02-design/error-handling.md)).
4. **Tracing.** Wire the trace and per-run cost/token capture now; retrofitting observability is how teams end up debugging blind ([monitoring & observability](../06-operations/monitoring-and-observability.md)).
5. **Real tools, one at a time**, each with validation, timeout, retry, and a structured error ([integrations & auth](integrations-and-auth.md)).
6. **Retrieval**, if the design calls for it — and evaluate it separately from generation ([RAG & tooling](rag-and-tooling.md)).
7. **Guardrails and the human gate** — input/output validation, permission checks, and the approval step, which changes how the run executes ([state & execution](state-and-execution.md), [human-in-the-loop](../02-design/human-in-the-loop.md)).
8. **The eval harness**, run against the real eval set, before tuning anything ([evaluation methods](../04-evaluation/evaluation-methods.md)).

Steps 1–4 are typically days, not weeks. If they are taking weeks, the design is too big for a first version ([scoping the first version](../01-framing/scoping-the-first-version.md)).

## Repository skeleton

Nothing here is agent-specific: it is an ordinary Python package in the standard [src layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/), with the [layered architecture](../02-design/architecture.md) mapped onto modules inside it.

```
pyproject.toml       dependencies with pins, tool config
src/
  ticket_agent/
    config/          settings class: model ids, routes, temperatures, limits, prices
    core/            the control loop, state object, stop conditions
    prompts/         prompt files, versioned and diffable — never inline strings
    tools/           one module per tool: args schema, implementation, errors
    mcp/             MCP server exposing the tool set
    memory/          session store and long-term store adapters
    retrieval/       ingest, chunk, embed, retrieve, rerank
    adapters/        one per external system (CRM, ticketing, storage)
    observability/   tracing, cost accounting, structured logging
    api/             FastAPI app: routes, schemas, streaming
tests/               unit, contract, and loop tests (see testing agent code)
evals/               datasets and the harness (see section 4)
docs/
  specs/             the agent spec(s), versioned with the code implementing them
```

`src/` is there for the usual reason: it forces the tests to run against the *installed* package rather than whatever happens to be in the working directory, so a missing entry in your packaging config fails in CI instead of in production. It costs you an editable install and nothing else.

One agent-specific consequence: `prompts/` lives **inside** the package, not at the repo root. Prompts are code — they ship with the artefact and are selected by version at runtime — so they must be package data (read via `importlib.resources`, and declared in `pyproject.toml`), not files read by relative path. A prompt directory outside the package works in dev and disappears in the wheel.

Two boundaries earn their keep: **adapters** (so the CRM can be swapped or faked in tests without touching the loop) and **prompts as files** (so a prompt change is a reviewable diff with a version id). Both are what make the design's [evolvability](../02-design/architecture.md) real rather than aspirational.

### Why the specs are in the repo

This is a convention, not a standard, and it is worth being explicit about the trade. The agent spec is not background reading — it is the document the loop's stop conditions, the guardrails, and the eval thresholds are all derived from, and tests assert against its numbers. Keeping it in `docs/specs/` means a PR that changes the escalation threshold changes the spec and the test in the same diff, and a reviewer sees both.

The alternative — the spec in Confluence or Notion, where the rest of the business can read it — has the failure mode everyone has met: the page says 0.7, the code says 0.6, and nobody can say which is intentional. If the spec must live in a wiki for the audience, keep the machine-readable part (thresholds, budgets, I/O contract) in the repo and have the wiki link to it, so there is exactly one authoritative copy of every number.

## Configuration and secrets

- **One settings object**, loaded from environment variables, holding model ids, temperatures, limits, retrieval parameters, and prices. No literals scattered through the code — you will change all of them during tuning.
- **Per-environment config** (dev / staging / prod) with the same code path. Differences that live in `if env == "prod"` branches are the ones that break on release day.
- **Pin versions** — model ids, SDKs, and the embedding model. An unpinned model id means the system silently changes underneath you ([regression & drift](../04-evaluation/regression-and-drift.md)).
- **Feature-flag the risky parts** — autonomy level, new tools, prompt versions, model id. Ship both paths in the same build, dark, and choose between them at runtime. Turning one on is then a flag flip rather than a rebuild-and-redeploy: you can enable it for 5% of tickets, and **turn it off in seconds without shipping anything** ([rollout & safety](../05-deployment/rollout-and-safety.md)). That last property is why it matters here more than in ordinary services — the way an agent fails is by behaving plausibly and wrongly at scale, and reverting a deploy is far too slow a lever when you notice.

## Definition of done for the first slice

Build is finished when, for the ticket-triage example carried through [from framing to technical design](../02-design/from-framing-to-technical-design.md):

- every `CODE` step is a tool with validation, a timeout, and a structured error;
- every `HUMAN` step is an actual gate the run pauses at, not a note in the doc;
- the run emits a trace with tokens, cost, model, prompt version, and tool calls;
- the eval harness runs the eval set the spec named — 300 historical tickets in this worked example, a number that came from the available labelled history, not from a rule — and reports the spec's release criteria;
- the failure paths — tool down, malformed output, step limit hit — have been exercised on purpose, not just imagined.

The last one is the one teams skip, and it is the one production tests for you.

## References

- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — build the simplest thing that works, then add.
- [The Twelve-Factor App](https://12factor.net/) — config in the environment, parity between environments, disposability.
