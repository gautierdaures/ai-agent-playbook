# Evaluation methods

Continuous evaluation is a first-class design goal, not an afterthought (see [framing goals](../01-framing/project-framing.md)).

## Deterministic where you can

For rational or deterministic chains, use deterministic evaluation: exact, reproducible checks rather than LLM-judged ones.

## Expert debate

Expert debate, having several agents argue a point, reduces errors and helps break loops. It doubles as an [error-handling technique](../02-design/error-handling.md).

## Semantic cache

Semantic caching returns the result of a similar previous query instead of recomputing. It cuts latency and avoids [useless LLM calls](../05-deployment/cost-management.md).
