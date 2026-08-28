# Integrations & authentication

An agent's usefulness is bounded by the systems it can reach, and connecting to them is usually where the build effort actually goes — the model call is the easy part. This note covers the mechanics: who the agent authenticates as, how credentials are held, and how to call real systems that rate-limit, paginate, and occasionally fail. The reversibility and permission *decisions* are made in [security & compliance](../02-design/security-and-compliance.md) and [feasibility & readiness](../01-framing/feasibility-and-readiness.md); this is how they are implemented.

## Who is the agent acting as?

Decide this per tool, before writing the adapter — it determines both the audit trail and the blast radius.

| Identity | How it works | Use when | Watch out for |
| --- | --- | --- | --- |
| **Service account** | The agent holds its own credential with a fixed scope | Back-office jobs, shared read-only data | It sees everything the account can see, for every user — the classic over-permission trap |
| **On-behalf-of the user** | The user's token is exchanged for a scoped one the agent uses | Anything touching per-user data or permissions | Token lifetime and refresh inside a long run |
| **Human-executed** | The agent prepares the action; an authorised human executes it | Irreversible or regulated writes | The gate must be real, not advisory ([human-in-the-loop](../02-design/human-in-the-loop.md)) |

> **The rule that prevents most incidents:** the agent must never have more access than the person it is acting for. A support agent that can read any customer's record because the service account can is a data-leak vector — and prompt injection turns it into an exploitable one.

## Credentials

Ordinary secret hygiene applies and is not restated here. Three things are specific to agents:

- **The credential must never reach the context window.** Not in the prompt, and not by accident either: a tool that surfaces a raw HTTP error can echo an `Authorization` header into the transcript, and an unhandled exception can carry a connection string. Anything in context is reachable by an injected instruction, and is also written to your traces. Redact at the adapter boundary and assert it in a test ([security & compliance](../02-design/security-and-compliance.md), [logging](../06-operations/logging.md)).
- **A run can outlive its token.** A run that checkpoints at a human gate on Friday resumes on Monday, and the access token it was carrying expired in between. Resolve credentials *at the point of use* rather than at run start, and checkpoint the identity the run acts for — never the bearer token, which would put a live credential in the state store ([state & execution](state-and-execution.md)).
- **Scope is enforced by the token, not by the tool.** The model chooses the arguments, so "read-only" implemented as *this adapter only issues GETs* is one injection or one refactor away from being false. Get a credential that cannot write, and let the API reject the call.

## Rate limits, retries, and budgets

An agent's call volume is non-deterministic: one hard case can trigger ten times the tool calls of a normal one. Assume you will hit limits.

- **Respect `Retry-After`**, then fall back to exponential backoff **with jitter** — without jitter, parallel runs retry in lockstep and re-create the spike.
- **Retry transient failures only** — timeout, 429, 5xx. A 4xx is a bug in the arguments; retrying it just burns the step budget ([RAG & tooling](rag-and-tooling.md) has the `tenacity` pattern).
- **Cap concurrency per dependency**, not just globally, so one slow system cannot consume every worker ([state & execution](state-and-execution.md)).
- **Give each run a tool-call budget** alongside its token budget. It is the cheapest protection against a loop hammering a partner API.
- **Fail soft where the design allows it** — "the archive is unavailable, answering without past tickets" is often better than failing the run, and it is a design decision to make explicitly, not a silent fallback.

## Reading: keep results small

Tool results land in the context window and are paid for on every subsequent turn ([memory & context engineering](../02-design/memory.md)):

- **Filter and paginate server-side.** Never fetch a list to filter it in the tool.
- **Return ids and summaries**, with a second tool to fetch the detail on demand.
- **Cap the page size in the tool schema** (`k: int = Field(5, ge=1, le=20)`) so the model cannot ask for ten thousand rows.

## Writing: assume it will be retried

- **Idempotency keys** on every write, derived from the run id and the logical action, so a retry cannot double-issue a refund.
- **Dry-run mode** on write tools, used by tests and by shadow-mode rollout ([rollout & safety](../05-deployment/rollout-and-safety.md)).
- **Classify each write as reversible or not** — the irreversible ones get a human gate, and that gate makes the run pausable ([state & execution](state-and-execution.md)).
- **Long operations return a handle**, not a blocked call: submit, then poll or receive a webhook.

## Environments

Build against a **sandbox tenant with seeded, realistic data** — never production. If the vendor has no sandbox, that is a feasibility finding worth raising early, not a workaround to improvise: the fallbacks (a recorded fixture layer, a read-only prod mirror) cost real time and belong in the plan. Keep the seeded fixtures in the repo so contract tests can run nightly ([testing agent code](testing-agent-code.md)).

## Checklist for adding an integration

1. Identity decided (service account / on-behalf-of / human-executed) and a scoped credential obtained.
2. Adapter behind an interface, so it can be faked in tests and swapped later.
3. Timeout on every call; retry with jitter on transient failures only.
4. Errors returned as structured, actionable data — never a raw stack trace ([tool design](../02-design/tool-design.md)).
5. Results filtered, paginated, and capped before they reach the context.
6. Writes idempotent, with a dry-run path.
7. Rate limit and quota documented in the adapter, with a per-run cap.
8. PII handling checked: what is sent, what is logged, what is retained ([logging](../06-operations/logging.md)).
9. Sandbox credentials wired into CI; contract test written.
10. Owner and escalation path recorded — when this API changes, who tells you?

## References

- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) — excessive agency is LLM06:2025, broken into excessive functionality, permissions, and autonomy.
- [RFC 8693 — OAuth 2.0 Token Exchange](https://datatracker.ietf.org/doc/html/rfc8693) — the on-behalf-of pattern.
- [AWS — Exponential backoff and jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/) — why jitter, and not backoff alone, stops retries re-creating the spike.
- [Stripe — idempotent requests](https://docs.stripe.com/api/idempotent_requests) — a worked idempotency-key contract: client-generated key, stored result, replayed response.
- [Model Context Protocol](https://modelcontextprotocol.io/) — a common interface for exposing tools to agents.
