# Vector databases

Choice depends on stage and scale:

- **Chroma** — very good for prototyping.
- **Pinecone** — for large volumes.
- **pgvector** — the slowest of the three, but keeps everything in Postgres.

Used for [long-term memory](../02-design/memory.md): stores the user profile and the schema of the conversation, and backs [RAG](rag-and-tooling.md).
