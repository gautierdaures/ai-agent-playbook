# Architecture

## BDI — Belief, Desire, Intention

A BDI architecture for state management:

- manages the agent state;
- drives the execution thread;
- handles the task response.

The LLM is the "brain"; it decides what we are trying to achieve. Reference seen in the training: Microsoft Learn material, well designed.

## Layered architecture (separation of concerns)

Separate the routes so each layer has a single responsibility. Suggested layers:

- **config** — a settings class: LLM tokens, routes, temperature, pricing, and so on. For multi-agent, centralize the config.
- **core** — all the sub-agents.
- **prompts** — the prompt layer.
- **logging** — log tool call, log tool result.
- **monitoring** — see [monitoring & observability](../06-operations/monitoring-and-observability.md).
- **mcp** — client and server.

Keep a separate logging service, for example behind an API call, so the agent can write to a logging system without coupling. See [logging](../06-operations/logging.md).

Use an MCP server for tool discovery.

## Hexagonal architecture

Separating the sources gives a hexagonal architecture. With this you can plug the agent into other applications. Present it as a global architecture across:

- software architecture;
- data;
- integration.

## Evolvability

Design so you can swap the LLM or the data source later without a rewrite.

## Cross-cutting best practices

- **Separation of concerns** — each layer owns its responsibility.
- **Logging** — log across the system, through the API and LLM boundary; every agent decision is journaled, with no PII in the journal.
- **GDPR by design** — no PII in the LLM or in agent calls.
- **Compliance** — bake GDPR and the AI Act into the architecture (see [security & compliance](security-and-compliance.md)).
