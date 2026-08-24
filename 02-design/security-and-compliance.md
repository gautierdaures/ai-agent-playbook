# Security & compliance

## Prompt injection

- Never inject raw user input into the system prompt.
- Define the actions the agent is allowed to take.
- Isolate the contexts of the different agents from each other.
- Apply the principle of least privilege.
- Limit the scope of each agent.

## Controlling tool inputs

Validate tool inputs in code (schema / regex) inside the tool, never by asking the model to sanitize its own input — this is both a correctness and an injection-defence measure. See [tool design](tool-design.md) for the full treatment.

## GDPR & AI Act

- **GDPR by design** — no PII in the LLM or in agent calls; do not store PII; store only what you actually need.
- **Journaling** — every agent decision is journaled, with no PII in the journal.
- **Architecture** — bake GDPR and AI Act compliance into the architecture, and integrate the AI Act into the prompts.

## Data touching humans

> **Caution:** Be careful with data visualization when it touches humans. How you surface data about people has real consequences.

Related: [architecture](architecture.md), [logging](../06-operations/logging.md).
