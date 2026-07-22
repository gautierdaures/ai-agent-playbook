# Multi-agent orchestration

Only go multi-agent when it is justified (see [do you need an agent?](../01-framing/do-you-need-an-agent.md)). Keep prompts fairly light and push business rules into the tools.

## Architecture patterns

### Peer-to-peer

Each agent communicates directly with the others.

- Risk of infinite loops, for example a newsletter agent bouncing back and forth.

### Central orchestrator

One orchestrator coordinates specialized agents.

- **Latency** — plus supervisor overload; these are the costs.
- **Specialization** — agents are specialized.
- **Traceability** — good traceability.

### Multi-level orchestrator

Orchestrators of orchestrators, to absorb the complexity of the lower levels, for example Marketing, Audit, a Data pipeline, or a decision-tree AI. Relevant in sectors like Health and Banking.

## Who calls what

- The orchestrator calls tools.
- The orchestrator calls sub-agents.
- Sub-agents modify the state and memory.

## Planning doesn't always need an LLM

- You are not obliged to use LLMs for every node.
- In the graph you can add a non-LLM node, but it cannot reflect.
- Each agent should be expert in a single domain.

## Building blocks

- **Hybrid layer** — sub-agents equipped with specific tools.
- **Data loading** — best as a sub-agent, because it is non-deterministic.
