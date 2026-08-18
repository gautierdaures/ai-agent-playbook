# Memory

## Two horizons

- **Short-term memory** — the session.
- **Long-term memory** — a vector store, spanning across sessions.

## What to keep

- The user profile.
- A vector database holding the schema of the conversation.

## You don't need the whole history

You are not obliged to keep the entire conversation history. Instead, make the relevant history accessible to the agent, for example via an API or history lookup, rather than stuffing everything into context.

Related: [error handling](error-handling.md) — context overflow (summarize and prune).
