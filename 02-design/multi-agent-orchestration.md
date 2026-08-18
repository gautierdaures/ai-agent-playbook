# Multi-agent orchestration

Only go multi-agent when it is justified (see [do you need an agent?](../01-framing/do-you-need-an-agent.md)). Keep prompts fairly light and push business rules into the tools.

## Architecture patterns

A handful of patterns cover almost every production system. The first three are the ones from the training; the rest are common alternatives worth knowing.

### Peer-to-peer (swarm / network)

Peer agents hand off to each other directly, with no central coordinator.

- Needs an explicit termination rule, or it risks infinite loops (for example a newsletter agent bouncing back and forth).

### Central orchestrator (supervisor)

One orchestrator routes work to specialized agents and aggregates their replies. This is the most common default in production.

- **Latency** — plus supervisor overload; these are the costs.
- **Specialization** — agents are specialized.
- **Traceability** — good traceability.

### Multi-level orchestrator (hierarchical)

Supervisors of supervisors: a top-level planner delegates whole sub-tasks to mid-level supervisors that own their own teams. Absorbs the complexity of the lower levels (for example Marketing, Audit, a Data pipeline, or a decision-tree AI). Relevant in sectors like Health and Banking.

### Sequential pipeline

A fixed chain of agents, each passing its output to the next, with no branching. Simple and predictable; a good backbone that other patterns plug into.

### Blackboard

Agents read from and write to a shared structured workspace, each picking up work when its precondition is met. Useful when many specialists contribute to a common evolving state.

### Group chat

Agents exchange messages in a shared conversation and a "chat manager" decides who speaks next (the AutoGen model). Good for open-ended collaboration and debate.

### Handoff

Control is transferred from one agent to another by capability matching: when an agent realizes it is not equipped for the next step, it hands off to a specialist.

> **Note:** In practice most production systems are a sequential backbone with one or two parallel or hierarchical steps, orchestrated by a supervisor — not a fully autonomous swarm.

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

## References

- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) (routing, parallelization, orchestrator-workers).
- [Multi-Agent Orchestration: 5 Patterns That Work](https://www.digitalapplied.com/blog/multi-agent-orchestration-5-patterns-that-work).
