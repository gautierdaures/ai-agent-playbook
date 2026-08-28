# Vector databases

A vector database stores embeddings for similarity search — the backbone of [RAG](rag-and-tooling.md) and of [long-term memory](../02-design/memory.md) (user profile, conversation schema). Pick one after the pipeline shape is settled — chunking, hybrid retrieval, and reranking decide more of your quality than the store does.

## When they help — and when they don't

- **Useful** when you need semantic retrieval over a large corpus, hybrid search (keyword + vector), metadata filtering, or memory that spans sessions.
- **Overkill** when the corpus is small (roughly a few thousand chunks): an in-memory index (FAISS) or `pgvector` on an existing Postgres is enough, and sometimes a plain keyword search or a long-context prompt beats retrieval entirely. Don't add a dedicated service before you need it.

## Options

- **Chroma** — very good for prototyping; simple, local-first.
- **pgvector** — the Postgres extension; the default when Postgres is already your data platform. Slower at very large scale, but zero extra infrastructure.
- **Pinecone** — fully managed, zero-ops at scale; built-in embeddings, reranking, and hybrid search.
- **Qdrant** — open source, Rust; strong performance and filtering, low latency.
- **Weaviate** — strong hybrid search (BM25 + dense + metadata filtering) in a single query.
- **Elasticsearch / OpenSearch** — `dense_vector` fields with kNN, ELSER sparse retrieval, and BM25 in one engine, fused with reciprocal rank fusion in a single query; quantisation (BBQ, int8, int4) keeps large indexes affordable in memory. The argument for it is the `pgvector` argument rather than a benchmark: if Elasticsearch is already in the stack for search or logs, adding vectors is a mapping change and a reindex, not a new service to run, secure, and back up.
- **Milvus** — built for billion-scale search; powerful but more resource-intensive to operate.
- **FAISS** — a library, not a server; great for in-process, read-heavy indexes.

## Pricing and scaling

- **Scale bands** — under ~10M vectors, almost any option performs adequately; 10M–1B narrows the field (Qdrant, Weaviate, Milvus); 1B+ points to Milvus or Vespa.
- **Cost shape** — `pgvector` on an existing Postgres is roughly $0 incremental under a few million vectors. Managed Pinecone commonly runs a few hundred dollars a month and up. Self-hosting an open-source engine trades a fixed VM cost for operational work, and starts paying off versus managed pricing once managed bills reach the low thousands per month.
- **Rules of thumb** — Postgres already in the stack → `pgvector`; Elasticsearch already in the stack → use it. Want managed and hands-off → Pinecone. Need top open-source latency/filtering → Qdrant. Need hybrid search as a first-class query → Elasticsearch or Weaviate. Billion-scale → Milvus.

The first two are the same rule, and it is the one that most often wins: the store you already operate starts with a large head start on the one that benchmarks slightly better, because retrieval quality is decided by chunking, hybrid retrieval, and reranking well before it is decided by the engine.

## References

- [MarkTechPost — Best vector databases in 2026: pricing, scale, tradeoffs](https://www.marktechpost.com/2026/05/10/best-vector-databases-in-2026-pricing-scale-limits-and-architecture-tradeoffs-across-nine-leading-systems/)
- [Firecrawl — Best vector databases: a complete comparison](https://www.firecrawl.dev/blog/best-vector-databases)
- [Elastic — Vector search in Elasticsearch](https://www.elastic.co/docs/solutions/search/vector) — dense and sparse vectors, hybrid retrieval with RRF, and quantisation options.
