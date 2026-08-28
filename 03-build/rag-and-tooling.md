# RAG & tooling

The build-side mechanics: how retrieval is actually assembled, and how a tool is actually written. The *why* behind each decision lives in section 2 — [tool design](../02-design/tool-design.md) for the contract, [memory & context engineering](../02-design/memory.md) for what belongs in the window. This note is the wiring: pipeline stages, default parameters worth starting from, and the code patterns that keep a tool safe to hand to a non-deterministic caller.

## Do you need retrieval at all?

Cheapest first — retrieval is infrastructure, and infrastructure is a running cost ([cost management](../06-operations/cost-management.md)):

- **A direct lookup** (SQL query, API call by id) beats RAG whenever the answer is addressable. Don't embed what you can `SELECT`.
- **The whole document in the prompt** beats RAG when the corpus is small and stable — a 30-page handbook fits in context, and with prompt caching it is cheap to re-send.
- **RAG** earns its place when the corpus is large, changes often, or must be cited.
- **A knowledge graph** on top when the questions are about *relationships* ("which suppliers are affected if this plant stops?") rather than passages.

## The RAG pipeline

Six stages. Each is a place where quality is won or lost, so instrument each one.

**1. Ingest & normalise.** Convert to text with structure intact — headings, tables, lists. PDF-to-text quality is the single most common cause of bad retrieval; check a sample by eye before blaming the model. Carry metadata from the source: `source_id`, `title`, `section`, `updated_at`, `access_level`.

**2. Chunk.** Start with **300–800 tokens per chunk and ~10–15% overlap**, split on structural boundaries (heading, section, then paragraph) rather than a fixed character count. Keep a chunk semantically self-contained: prepend the document title and heading path to each chunk so it still makes sense out of context.

**3. Embed & index.** One embedding model for both indexing and querying — mixing them silently destroys relevance. Store the metadata alongside the vector so you can filter (`access_level`, `updated_at`) at query time; filtering is what makes multi-tenant retrieval safe. See [vector databases](vector-databases.md) for choosing the store.

**4. Retrieve.** Default to **hybrid search** — BM25 (exact terms, product codes, error numbers) plus dense vectors (paraphrase, synonym) — and fetch generously: `top_k` around 20 candidates.

**5. Rerank.** Pass those candidates through a cross-encoder reranker and keep the **top 3–5**. This is usually the highest-return single change to a mediocre RAG system, because it lets you retrieve broadly without flooding the context window ([context rot](../02-design/memory.md)).

**6. Assemble.** Give the model each chunk with its `source_id` and ask for the ids it used in the output schema. That makes answers auditable, makes hallucinated citations detectable, and gives the eval set something objective to score.

> **Caution:** Re-index on a schedule and on source change, and delete chunks when the source is deleted — a stale index is an agent confidently citing a policy that was withdrawn last quarter.

## Knowledge graphs & ontologies

Where relationships carry the meaning, complement vectors with a graph:

- **RAG for ontologies** — the graph holds the entity types and their relations; retrieval walks it instead of guessing from prose.
- **Build the graph** by extracting entities and relations from the corpus (an LLM pass with a strict output schema works well), then storing them as typed edges.
- **Hybrid at query time** — vector search finds the entry-point entities, the graph expands to their neighbourhood, and the model sees both the passages and the relations.
- **Recommendation and personalization** by user profile ride on the same graph — the profile is another node with edges to what the user touched.

## Evaluate retrieval separately from generation

A wrong answer has two possible causes; test them apart ([evaluation methods](../04-evaluation/evaluation-methods.md)):

- **Retrieval** — build a small set of question → known-relevant-chunk pairs and track **recall@k** and **hit rate**. If the right chunk is never fetched, no amount of prompting fixes the answer.
- **Generation** — given the correct chunks, is the answer faithful to them? Score groundedness and citation correctness.
- Log the retrieved `source_id`s on every production run so failures can be replayed ([monitoring & observability](../06-operations/monitoring-and-observability.md)).

## Writing a tool

The design rules — granularity, descriptions the model reads, actionable errors — are in [tool design](../02-design/tool-design.md). The implementation pattern:

```python
from pydantic import BaseModel, Field, field_validator
from tenacity import retry, stop_after_attempt, wait_exponential

class SearchTicketsArgs(BaseModel):
    """Search past support tickets for similar cases."""   # the model reads this
    query: str = Field(..., description="Natural-language description of the issue")
    k: int = Field(5, ge=1, le=20, description="How many tickets to return")

    @field_validator("query")
    @classmethod
    def not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("query must not be empty")
        return v

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, max=8))
def _call_archive(query: str, k: int) -> list[dict]:
    return archive_client.search(query, k=k, timeout=5)     # always a timeout

def search_past_tickets(args: SearchTicketsArgs) -> dict:
    try:
        hits = _call_archive(args.query, args.k)
    except TimeoutError:
        return {"error": "archive_timeout", "retryable": True,
                "message": "Ticket archive did not respond. Answer without past tickets."}
    # return identifiers and summaries, not full payloads
    return {"tickets": [{"id": h["id"], "summary": h["summary"][:300]} for h in hits]}
```

What the pattern buys you:

- **Validation in code, never in the prompt** — the Pydantic model rejects bad arguments before they reach the system, and doubles as the schema the model is given ([security & compliance](../02-design/security-and-compliance.md)).
- **Retry with exponential backoff** via `tenacity`, wrapped around the network call only — never around the whole tool, or a partial write gets replayed. Retry transient errors (timeout, 429, 5xx); never retry a 4xx.
- **Timeouts on every call**, so a hanging dependency cannot consume the agent's step budget.
- **Errors returned as data**, in a shape the agent can act on — including what to do instead. An exception that escapes the tool is a dead run; a structured error is a recoverable one ([error handling](../02-design/error-handling.md)).
- **Token-efficient returns** — ids and summaries, with a second tool to fetch detail on demand.
- **Idempotency on writes** — pass a caller-supplied key so a retried `create_refund` does not issue two refunds.

Business rules belong in the tool, not the prompt: a tool that refuses out-of-policy arguments is enforced, a prompt that asks the model to respect policy is a suggestion ([multi-agent orchestration](../02-design/multi-agent-orchestration.md)).

When the tool talks to a real system, the identity it authenticates as, its rate-limit handling, and idempotency on writes are covered in [integrations & auth](integrations-and-auth.md); testing the validation and error branches without a model is in [testing agent code](testing-agent-code.md).

## Serving tools over MCP

Expose the tool set through an MCP server so the agent discovers and calls tools through one interface, and tools can be added or swapped without touching the core loop. Keep the server's tool list scoped per agent — discovery is not a reason to expose every tool to everyone. Wiring and framework fit are in [frameworks & infrastructure](frameworks-and-infra.md); where MCP sits in the layers is in [architecture](../02-design/architecture.md).

## Versioning prompts and tool definitions

Prompts and tool descriptions change model behaviour as much as code does, so treat them as code:

- Keep them in **files under version control**, not inline strings scattered through the codebase, and give each a stable id plus a version (`ticket_triage.classify.v3`).
- **Log the prompt and tool-definition versions with every run**, so a quality shift in production can be traced to the change that caused it.
- **Regression-test before promoting** a new version: run the eval set against old and new, and compare ([regression & drift](../04-evaluation/regression-and-drift.md)). A prompt edit that improves one case and breaks four is normal, and invisible without this.
- Roll out prompt changes like code — behind a flag, on a slice of traffic first ([rollout & safety](../05-deployment/rollout-and-safety.md)).

## References

- [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — token-efficient returns and actionable errors.
- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — retrieving on demand rather than pre-loading.
- [tenacity](https://tenacity.readthedocs.io/) — retry, backoff, and stop conditions in Python.
- [Pydantic](https://docs.pydantic.dev/) — schema validation and structured outputs.
