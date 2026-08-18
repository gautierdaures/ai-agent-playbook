# Logging

- Keep a separate logging service, for example behind an API call, so writing logs does not couple to the agent. See [architecture](../02-design/architecture.md).
- Be very parsimonious with logs; do not log everything.
- Store logs in cold or archive storage.
- Never store PII; store only what you need.
- Every agent decision is journaled and dated, with no PII in the journal. See [security & compliance](../02-design/security-and-compliance.md).
