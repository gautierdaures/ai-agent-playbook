# RAG & tooling

## RAG and knowledge

- RAG for ontologies.
- Build a graph of relationships, between articles and between researchers.
- Use it for recommendation and personalization by user profile (see the [bibliographic assistant example](../01-framing/example-use-cases.md)).

## Tooling practices

- Put business rules in the tools; keep prompts light.
- Validate inputs with regex directly in the tool, not via an LLM.
- Define the output format with Pydantic.
- Version your prompts; use a prompt framework, and consider XML-structured prompts.
- Use tenacity for retry and timeout on API calls.

Related design notes: [prompting](../02-design/agent-anatomy-and-prompting.md), [error handling](../02-design/error-handling.md).
