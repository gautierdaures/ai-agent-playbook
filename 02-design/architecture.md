# Architecture

The system-level view of an agentic system: the components every agent has, how to arrange them into layers, and how to keep the whole thing evolvable. For the agent's internal reasoning cycle see [agent anatomy & prompting](agent-anatomy-and-prompting.md); for many-agent topologies see [multi-agent orchestration](multi-agent-orchestration.md). This note assumes you have a [technical framing doc](from-framing-to-technical-design.md) to design against.

## The core components

Whatever the framework, a production agent system is built from the same handful of building blocks:

- **Model core** — the LLM that reasons, plans, and decides the next action.
- **Control loop / orchestration runtime** — drives the perceive → decide → act → observe cycle, holds state, handles retries and hand-offs, and enforces stop conditions.
- **Tools** — the agent's hands: APIs, code execution, search, databases. Discovered and called through an [MCP](#use-mcp-for-tools) surface. See [RAG & tooling](../03-build/rag-and-tooling.md).
- **Memory & context** — short-term (session) and long-term (vector store, knowledge graph). See [memory](memory.md).
- **Guardrails** — input/output validation, permission checks, and human gates on irreversible actions. See [error handling](error-handling.md) and [security & compliance](security-and-compliance.md).
- **Observability** — traces, logs, and evals so you can see and improve what the agent does. See [monitoring & observability](../06-operations/monitoring-and-observability.md).

Design in that order of necessity: you always need a model core and a loop; add tools, memory, and extra guardrails only when the [success criteria](from-framing-to-technical-design.md) demand them.

## The control loop

The heart of the system is a loop, not a straight line:

> **perceive → reason / plan → act (tool call) → observe result → repeat**

It runs until an explicit **stop condition** is met — the goal is reached, a step/budget/time limit is hit, or a guardrail halts it. Without a stop condition an agent loops forever; make it a first-class design decision, not an afterthought ([error handling](error-handling.md)). The loop's working state and memory live in a semi-structured object (for example JSON).

## BDI — Belief, Desire, Intention

A useful way to structure that loop state is the BDI model:

- manages the agent state;
- drives the execution thread;
- handles the task response.

The LLM is the "brain"; it decides what we are trying to achieve. BDI is a classical agent model from multi-agent systems research, formalized by Rao and Georgeff: **beliefs** (the agent's view of the world), **desires/goals** (what it wants to achieve), and **intentions** (goals it has committed to acting on). See the references below.

## Layered architecture (separation of concerns)

Separate the routes so each layer has a single responsibility. Suggested layers:

- **config** — a settings class: LLM tokens, routes, temperature, pricing, and so on. For multi-agent, centralize the config.
- **core** — the control loop and all the sub-agents.
- **prompts** — the prompt layer.
- **tools / mcp** — client and server; tool discovery and invocation.
- **memory** — session and long-term stores.
- **logging** — log tool call, log tool result.
- **monitoring** — see [monitoring & observability](../06-operations/monitoring-and-observability.md).

Keep a separate logging service, for example behind an API call, so the agent can write to a logging system without coupling. See [logging](../06-operations/logging.md).

### Use MCP for tools

Use an MCP server for tool discovery, so the agent finds and calls tools through one consistent interface and you can add or swap tools without touching the core.

## Hexagonal architecture

Separating the sources gives a hexagonal (ports-and-adapters) architecture. With this you can plug the agent into other applications — the core loop stays the same while adapters change. Present it as a global architecture across:

- software architecture;
- data;
- integration.

## Designing one, in practice

Work from the [technical framing doc](from-framing-to-technical-design.md):

1. **Identify the loop** — the single agent and its stop condition. Resist adding more agents until one is proven ([multi-agent orchestration](multi-agent-orchestration.md)).
2. **List the tools** the steps require, and their permission scope.
3. **Decide memory** — does the task need anything beyond the current context? Only add long-term memory if it does ([memory](memory.md)).
4. **Place the guardrails** — validate inputs and outputs, and put a human gate on every irreversible action.
5. **Wire observability from day one** — traces and evals are not a retrofit ([project framing](../01-framing/project-framing.md)).

Start with the simplest arrangement that meets the criteria and let it grow.

## Evolvability

Design so you can swap the LLM or the data source later without a rewrite. The layering and hexagonal boundaries above are what make that swap cheap.

## Cross-cutting best practices

- **Separation of concerns** — each layer owns its responsibility.
- **Logging** — journal every agent decision across the API and LLM boundary; the mechanics (separate service, dating, retention) live in [logging](../06-operations/logging.md).
- **Compliance & privacy** — no PII in the LLM or agent calls; bake GDPR and the AI Act into the architecture. Owned in [security & compliance](security-and-compliance.md).

## References

- Rao & Georgeff — [*BDI Agents: From Theory to Practice*](https://cdn.aaai.org/ICMAS/1995/ICMAS95-042.pdf) (1995), the foundational BDI paper.
- [Belief–Desire–Intention architecture — overview](https://www.sciencedirect.com/topics/computer-science/belief-desire-intention-architecture) (ScienceDirect Topics).
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) — the control loop and when to add components.
- [Choose a design pattern for your agentic AI system](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system) (Google Cloud Architecture Center).
