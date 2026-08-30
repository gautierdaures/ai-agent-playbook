# 5. Deployment

Getting it into users' hands.

_Scope: rollout strategy, infrastructure, release gating, staging vs. production, rollback._

## Notes

- [rollout-and-safety.md](rollout-and-safety.md) — the staged-rollout ladder, tiered data-masking (V1/V2/V3), absolute stop conditions, traceability.
- [adoption-and-change-management.md](adoption-and-change-management.md) — driving real usage: acculturation, ADKAR, adoption signals.

### Deploying on GCP

Cloud-specific mechanics, on the Gemini Enterprise Agent Platform (formerly Vertex AI):

- [gcp-runtimes.md](gcp-runtimes.md) — Agent Runtime vs. Cloud Run vs. GKE, the timeout wall, revisions and traffic splits, networking and cost shape.
- [gcp-state-sessions-and-memory.md](gcp-state-sessions-and-memory.md) — Sessions, Memory Bank, checkpoints and knowledge stores; residency, deletion and lock-in.
- [gcp-evaluation-and-release.md](gcp-evaluation-and-release.md) — Agent Evaluation, simulation, online monitors, Cloud Trace, the Cloud Build gate, and the Govern pillar.
