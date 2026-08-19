# Evaluation methods

Continuous evaluation is a first-class design goal, not an afterthought (see [framing goals](../01-framing/project-framing.md)).

## Deterministic where you can

For rational or deterministic chains, use deterministic evaluation: exact, reproducible checks rather than LLM-judged ones.

## Expert debate

Having several agents argue a point reduces errors, surfaces disagreement, and can break a stuck decision. This is the canonical home for the technique; [error handling](../02-design/error-handling.md) reuses it to break tie/loop situations.
