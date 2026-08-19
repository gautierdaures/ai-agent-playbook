# RAG & tooling

## RAG and knowledge

- RAG for ontologies.
- Build a knowledge graph of relationships between entities.
- Use it for recommendation and personalization by user profile.

## Tooling practices

These are the build-side mechanics; the design rationale for each lives in section 2.

- **Tool design** — granularity, descriptions the model reads, and error shape are covered in [tool design](../02-design/tool-design.md). Keep prompts light and push business rules into the tools ([multi-agent orchestration](../02-design/multi-agent-orchestration.md)).
- **Input validation** — enforce it in code (schema / regex) inside the tool, never via the LLM. See [tool design](../02-design/tool-design.md) and [security & compliance](../02-design/security-and-compliance.md).
- **Output format** — define it with Pydantic ([agent anatomy & prompting](../02-design/agent-anatomy-and-prompting.md)).
- **Prompt versioning** — version prompts so a change can be regression-tested; frameworks and XML structure are in [prompting](../02-design/agent-anatomy-and-prompting.md).
- **Retry & timeout** — use tenacity on API calls; the retry/fallback pattern is designed in [error handling](../02-design/error-handling.md).
