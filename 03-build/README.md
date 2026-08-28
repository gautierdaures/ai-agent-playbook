# 3. Build

Turning the design into working software.

_Scope: implementation, framework/SDK choices, integrations, tooling, development workflow._

## Notes

- [from-design-to-code.md](from-design-to-code.md) — the entry point: what design hands over, the walking skeleton, the order of construction, repo skeleton, config & secrets, definition of done.
- [frameworks-and-infra.md](frameworks-and-infra.md) — whether you need a framework at all; LangGraph vs. LangChain vs. CrewAI and the rest; FastAPI and what sits around it; MCP.
- [rag-and-tooling.md](rag-and-tooling.md) — the RAG pipeline end to end (chunking, hybrid retrieval, reranking), knowledge graphs, writing a tool in code (Pydantic, tenacity, error shape), MCP, prompt versioning.
- [vector-databases.md](vector-databases.md) — Chroma, pgvector, Pinecone, Qdrant, Weaviate, Milvus, FAISS; scale bands and cost shape.
- [integrations-and-auth.md](integrations-and-auth.md) — who the agent acts as; credentials; rate limits and backoff; small reads, idempotent writes; sandboxes; the per-integration checklist.
- [state-and-execution.md](state-and-execution.md) — where run state lives; checkpointing; why a human gate is a pause; sync vs. streaming vs. background; timeouts, concurrency, cancellation.
- [testing-agent-code.md](testing-agent-code.md) — the test pyramid around the model; faking the model to test the loop; failure-path tests; what runs in CI.
