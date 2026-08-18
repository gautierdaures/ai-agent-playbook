# Vector databases

A vector database stores embeddings for similarity search — the backbone of [RAG](rag-and-tooling.md) and of [long-term memory](../02-design/memory.md) (user profile, conversation schema).

## When they help — and when they don't

- **Useful** when you need semantic retrieval over a large corpus, hybrid search (keyword + vector), metadata filtering, or memory that spans sessions.
- **Overkill** when the corpus is small (roughly a few thousand chunks): an in-memory index (FAISS) or `pgvector` on an existing Postgres is enough, and sometimes a plain keyword search or a long-context prompt beats retrieval entirely. Don't add a dedicated service before you need it.

## Options

- **Chroma** — very good for prototyping; simple, local-first.
- **pgvector** — the Postgres extension; the default when Postgres is already your data platform. Slower at very large scale, but zero extra infrastructure.
- **Pinecone** — fully managed, zero-ops at scale; built-in embeddings, reranking, and hybrid search.
- **Qdrant** — open source, Rust; strong performance and filtering, low latency.
- **Weaviate** — strong hybrid search (BM25 + dense + metadata filtering) in a single query.
- **Milvus** — built for billion-scale search; powerful but more resource-intensive to operate.
- **FAISS** — a library, not a server; great for in-process, read-heavy indexes.

## Pricing and scaling

- **Scale bands** — under ~10M vectors, almost any option performs adequately; 10M–1B narrows the field (Qdrant, Weaviate, Milvus); 1B+ points to Milvus or Vespa.
- **Cost shape** — `pgvector` on an existing Postgres is roughly $0 incremental under a few million vectors. Managed Pinecone commonly runs a few hundred dollars a month and up. Self-hosting an open-source engine trades a fixed VM cost for operational work, and starts paying off versus managed pricing once managed bills reach the low thousands per month.
- **Rules of thumb** — Postgres already in the stack → `pgvector`. Want managed and hands-off → Pinecone. Need top open-source latency/filtering → Qdrant. Need hybrid search → Weaviate. Billion-scale → Milvus.

## References

- [MarkTechPost — Best vector databases in 2026: pricing, scale, tradeoffs](https://www.marktechpost.com/2026/05/10/best-vector-databases-in-2026-pricing-scale-limits-and-architecture-tradeoffs-across-nine-leading-systems/)
- [Firecrawl — Best vector databases: a complete comparison](https://www.firecrawl.dev/blog/best-vector-databases)
