# Rollout & production safety

## The staged-rollout ladder

Widen autonomy in stages, earning each step with evidence rather than shipping full autonomy on day one. This is the deployment-mechanics home for the ladder; [scoping the first version](../01-framing/scoping-the-first-version.md) plans for it, and [human-in-the-loop](../02-design/human-in-the-loop.md) designs the gate at each stage.

1. **Shadow mode** — the agent runs alongside the human, outputs compared, nothing shipped.
2. **Human-in-the-loop** — the agent proposes; a human approves before anything takes effect.
3. **Supervised autonomy** — the agent acts; humans audit a sample and handle exceptions.
4. **Autonomous** — only for proven, low-risk paths.

Move up a rung on measured approval rates, not a calendar (see [human-in-the-loop — progressive autonomy](../02-design/human-in-the-loop.md)). This is the production expression of the [autonomy levels](../01-framing/do-you-need-an-agent.md) set in framing.

## Progressive data-masking strategy

Roll out data occultation and masking in tiers rather than all at once:

- **V1** — simple.
- **V2** — intermediate.
- **V3** — complex.

Start simple and increase sophistication as you gain confidence.

## Absolute stop conditions

Shipping an absolute stop condition is mandatory in production — it is the hard backstop against runaway loops and cost. The conditions themselves (timeout, max iterations, max items) and the design rationale live in [error handling](../02-design/error-handling.md).

## Traceability

- Every log is dated.
- The output must describe the data it retrieved and include metadata (JSON).
- This feeds [monitoring & observability](../06-operations/monitoring-and-observability.md).
