# Regression & drift

## Prompt drift and model drift

Behavior shifts when the model changes or when the full context changes. Treat prompt drift the same way you would treat model drift: as something to detect and guard against.

## Regression testing

- Run regression tests before any prompt change.
- Wire this into the release flow so a prompt edit cannot silently degrade behavior.

Related metrics tracked in production: [key metrics](../06-operations/metrics-and-sre.md).
