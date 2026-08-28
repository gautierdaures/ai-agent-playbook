# Evaluating company-specific guardrails

Generic safety — toxicity, self-harm, CSAM, obvious jailbreaks — is largely handled by the provider and by off-the-shelf classifiers. It is not what breaks your product. What breaks your product is the clause in your own policy: *never quote a price without the validity disclaimer*, *never confirm a refund above €500*, *never give tax advice*, *always name the regulator in a complaint response*, *never mention a competitor*. Nobody else's benchmark covers those, so this is the part you must evaluate yourself.

## Turn the policy into a register

Policy that lives in a Confluence page cannot be tested. Extract each clause into a row with an ID:

| Field | Example |
| --- | --- |
| `id` | `POL-014` |
| `statement` | Never state a refund amount above €500 without human approval |
| `owner` | Finance / Compliance — a named person, not a team inbox |
| `enforcement` | Deterministic (amount cap in tool code) |
| `severity` | Blocking — release fails on any miss |
| `cases` | 6 positive, 4 negative, 5 adversarial |

The register is the contract between compliance and engineering, and it is what makes the eval suite auditable. When a regulator or an internal auditor asks "how do you know it doesn't do X", the answer is a policy ID, its cases, and its pass rate per release.

## Enforce at the cheapest layer that works

| Layer | Good for | Note |
| --- | --- | --- |
| **Deterministic code** | Numeric caps, allow-lists, forbidden entities, required fields, permission checks | An amount cap is an `if`, not a prompt. Anything expressible here should never be a prompt instruction |
| **Prompt / system instruction** | Tone, framing, house style, disclosure wording | Cheapest to write, weakest under adversarial pressure — never the only layer for a blocking policy |
| **Classifier / validator** | Topic bans, PII, injection detection, output format | Latency in the tens to low hundreds of milliseconds; run in parallel with the model call where possible |
| **Post-hoc judge** | Nuanced clauses ("did it give advice or information?") | Slow and non-deterministic; use for measurement, and for blocking only when nothing cheaper works |

**Eval each layer at its own layer, then eval the composition.** A validator with 99% recall in isolation tells you nothing about the system if the agent phrases things the validator never sees.

## Test both directions

The failure everyone tests for is the rail not firing. The failure that kills adoption is the rail firing when it shouldn't.

- **Violation recall** — of cases that genuinely breach the policy, how many are caught? For blocking policies this must be 100% on the regression set.
- **Over-refusal / false-block rate** — of legitimate cases, how many get blocked, hedged into uselessness, or padded with an irrelevant disclaimer? Track this per policy with an explicit budget. A guardrail suite that only measures recall converges on an agent that refuses everything and scores perfectly.

Every policy therefore needs **negative cases**: realistic requests that come close to the line and must be answered normally.

## Adversarial cases, and where the attacker actually is

Red-team each policy specifically: rephrasings, roleplay framing, hypotheticals, incremental escalation across turns, a second language, and — for anything multi-turn — pressure ("my manager already approved this"). Keep every successful bypass as a permanent regression case.

The critical point for agents: **the untrusted input is not only the user's message.** Instructions arrive through retrieved documents, tool responses, file contents, and web pages. A guardrail suite that only fuzzes the user turn misses the entire indirect-injection class, which is where the real exposure is for a tool-using agent. Build cases where the malicious instruction is buried in a returned document or an API payload ([tool design](../02-design/tool-design.md), [RAG & tooling](../03-build/rag-and-tooling.md)).

Use the OWASP LLM Top 10 as the generic checklist — prompt injection, sensitive-information disclosure, excessive agency, system-prompt leakage, unbounded consumption — and your policy register as the specific one.

## Gate on severity

- **Blocking policies** — any failure fails the build, no discussion, no percentage threshold. These run on every change that touches prompts, tools, or the model ([CI & triggers](eval-in-ci.md)).
- **Advisory policies** — tracked as a rate with a target; a regression opens a ticket rather than stopping a release.

Keep guardrail results out of the aggregate quality score. A 97% overall pass rate that includes a breached blocking policy is a number that hides the only thing that mattered.

## In production

Offline suites prove the rails work on cases you thought of. Live traffic finds the ones you didn't — so the same checks run as [online evaluation](online-evaluation.md), and every production breach becomes an offline case the same week. Enforcement design and the decision to fail open or closed live in [error handling](../02-design/error-handling.md) and [human-in-the-loop](../02-design/human-in-the-loop.md).

## References

- [OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/) — the generic risk checklist: LLM01 prompt injection through LLM10 unbounded consumption.
- [NVIDIA NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) — programmable dialogue rails; Apache 2.0.
- [Guardrails AI](https://github.com/guardrails-ai/guardrails) — validator library for input/output checks; Apache 2.0.
- [promptfoo](https://github.com/promptfoo/promptfoo) — declarative eval configs plus a red-teaming/vulnerability-scanning mode that generates adversarial cases.
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — the governance frame policies get tiered against.
