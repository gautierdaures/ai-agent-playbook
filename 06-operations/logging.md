# Logging

Traces are the primary object for an agent ([monitoring & observability](monitoring-and-observability.md)); logs exist for the two things a trace does not hold — the operational events around the run, and the **audit journal** that outlives it.

| Signal | Answers | Cardinality | Retention |
| --- | --- | --- | --- |
| **Metrics** | Is it healthy, is it changing | Low | Months–years |
| **Traces** | What did *this run* do, step by step | High | Days–weeks |
| **Logs** | What happened around it; what was decided | High | Days (debug) / years (audit) |

## Two streams that get confused

| | Debug log | Audit journal |
| --- | --- | --- |
| **For** | Engineers, during and after an incident | Compliance, disputes, "why did it do that?" six months later |
| **Content** | Errors, retries, fallbacks, config loads, cache misses | One record per agent **decision**: what, on whose behalf, under which version, with what result |
| **Volume** | Sampled, verbose | Small, one row per consequential action |
| **Mutability** | Best-effort, droppable | Append-only, immutable |
| **Retention** | Days to weeks | Whatever the regulation says |
| **PII** | None | Pseudonymous ids only |

Every agent decision is journaled and dated, with no PII in the journal ([security & compliance](../02-design/security-and-compliance.md)). Mixing the two streams means either paying archive prices for debug noise or losing the audit trail to a log-level change.

## Storing prompts and completions is a decision, not a default

You cannot debug an agent without seeing what it said — and the payload is simultaneously the most sensitive data you hold and the largest storage line. The policy that resolves it:

- **Redact at emit, never at query.** A redaction that lives in the query layer is one dashboard away from being bypassed, and the raw data is already on disk.
- **Sample the happy path, keep 100% of failures**, escalations, guardrail trips, overrides and runs on a new version — those are the ones that become [eval cases](../04-evaluation/eval-datasets.md).
- **Short TTL on payloads, long TTL on structure.** Statuses, timings and version stamps are cheap and are what trends are built from.
- **Reference, don't duplicate.** The audit journal stores a payload id, not the payload.

## Keep it parsimonious

- Write through a **separate logging service** — behind an API call — so logging does not couple to the agent ([architecture](../02-design/architecture.md)).
- Be parsimonious: do not log everything. Log the decision, not the deliberation.
- **Structured JSON, one event per line**, always carrying `run_id`, `session_id`, provider `request-id` and the version stamps — a log you cannot join to a trace is an orphan.
- Never log secrets, tool credentials, full retrieved documents, or raw third-party payloads ([integrations & auth](../03-build/integrations-and-auth.md)).
- Tier storage: hot for the on-call window, then **cold or archive storage**.
- Never store PII; store only what you need.

The retrieval side has its own quiet requirement: log **ingest completion and index age**, because a stale index never raises an error ([alerting](alerting-and-anomaly-detection.md)).
