# From design to code

Design ([section 2](../02-design/README.md)) decides *what to build*; build turns it into a running system. The handover artefact is the [technical framing doc](../02-design/from-framing-to-technical-design.md) and the agent spec(s) inside it. This note is the entry point into section 3: what to build first, in what order, and how the repository is laid out.

## What design hands over

Do not start until you have, from the technical framing doc:

- the **step decomposition** with each step marked `LLM` / `CODE` / `HUMAN`;
- an **agent spec** per agent — I/O contract, tools, stop conditions, guardrails, numeric success criteria;
- the **tool and data inventory**, with access confirmed by whoever owns each system;
- the **non-functional budgets** — latency p95, cost per run, availability;
- the **eval plan** and the eval set (or a dated task to build it).

A missing item is not a detail to sort out later: an agent spec without stop conditions produces a runaway loop, and a tool inventory without confirmed access produces a demo that cannot ship.

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

The [layered architecture](../02-design/architecture.md) maps directly onto directories. A layout that holds up:

```
agent/
  config/        settings class: model ids, routes, temperatures, limits, prices
  core/          the control loop, state object, stop conditions
  prompts/       prompt files, versioned and diffable — never inline strings
  tools/         one module per tool: args schema, implementation, errors
  mcp/           MCP server exposing the tool set
  memory/        session store and long-term store adapters
  retrieval/     ingest, chunk, embed, retrieve, rerank
  adapters/      one per external system (CRM, ticketing, storage)
  observability/ tracing, cost accounting, structured logging
specs/           the agent spec(s) — kept next to the code, reviewed with it
evals/           datasets and the harness (see section 4)
tests/           unit, contract, and loop tests (see testing agent code)
```

Two boundaries earn their keep: **adapters** (so the CRM can be swapped or faked in tests without touching the loop) and **prompts as files** (so a prompt change is a reviewable diff with a version id). Both are what make the design's [evolvability](../02-design/architecture.md) real rather than aspirational.

## Configuration and secrets

- **One settings object**, loaded from environment variables, holding model ids, temperatures, limits, retrieval parameters, and prices. No literals scattered through the code — you will change all of them during tuning.
- **Per-environment config** (dev / staging / prod) with the same code path. Differences that live in `if env == "prod"` branches are the ones that break on release day.
- **Secrets from a secret manager**, never in code, config files, or prompts ([integrations & auth](integrations-and-auth.md)).
- **Pin versions** — model ids, SDKs, and the embedding model. An unpinned model id means the system silently changes underneath you ([regression & drift](../04-evaluation/regression-and-drift.md)).
- **Feature-flag the risky parts** — autonomy level, new tools, prompt versions — so rollout is a config change, not a deploy ([rollout & safety](../05-deployment/rollout-and-safety.md)).

## Definition of done for the first slice

Build is finished when, for the ticket-triage example carried through [from framing to technical design](../02-design/from-framing-to-technical-design.md):

- every `CODE` step is a tool with validation, a timeout, and a structured error;
- every `HUMAN` step is an actual gate the run pauses at, not a note in the doc;
- the run emits a trace with tokens, cost, model, prompt version, and tool calls;
- the eval harness runs the 300-case set and reports the spec's numbers;
- the failure paths — tool down, malformed output, step limit hit — have been exercised on purpose, not just imagined.

The last one is the one teams skip, and it is the one production tests for you.

## References

- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — build the simplest thing that works, then add.
- [The Twelve-Factor App](https://12factor.net/) — config in the environment, parity between environments, disposability.
