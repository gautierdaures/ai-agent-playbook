# Rollout & production safety

## Progressive data-masking strategy

Roll out data occultation and masking in tiers rather than all at once:

- **V1** — simple.
- **V2** — intermediate.
- **V3** — complex.

Start simple and increase sophistication as you gain confidence.

## Absolute stop conditions

Always ship an absolute stop condition. In production that means:

- timeout;
- max iterations;
- max items.

See [error handling](../02-design/error-handling.md) for the design side.

## Traceability

- Every log is dated.
- The output must describe the data it retrieved and include metadata (JSON).
- This feeds [monitoring & observability](../06-operations/monitoring-and-observability.md).
