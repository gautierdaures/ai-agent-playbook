# Testing agent code

Evals measure *judgement* ([section 4](../04-evaluation/README.md)). Tests protect everything deterministic around it: tool code, schemas, the loop, guardrails, adapters. Only evals means debugging a broken tool through the model; only unit tests means shipping an agent nobody measured.

## The test pyramid for an agent system

| Layer | Covers | Model involved | Runs |
| --- | --- | --- | --- |
| **Tool unit tests** | Argument validation, business rules, return shape, error branches | No | Every commit |
| **Schema & definition snapshots** | Output schema, tool names/descriptions/params | No | Every commit |
| **Loop tests** | Step sequence, stop conditions, error recovery, gate placement | Faked | Every commit |
| **Guardrail tests** | Injection strings, PII redaction, permission denial | Faked | Every commit |
| **Adapter contract tests** | The real API still behaves as the adapter assumes | No | Nightly / on adapter change |
| **Smoke tests** | A couple of golden cases end to end, real model | Real | Pre-deploy |
| **Evals** | Answer quality on the eval set | Real | On prompt/model change, pre-release ([section 4](../04-evaluation/evaluation-methods.md)) |

The base is ordinary software testing and should feel ordinary: fast, deterministic, no API keys.

## Fake the model, script the run

The loop is deterministic code driving a non-deterministic component. Replace the component and the loop becomes testable:

```python
class FakeModel:
    """Replays a scripted sequence of responses, in order."""
    def __init__(self, responses): self.responses, self.calls = list(responses), []

    def complete(self, messages, tools=None):
        self.calls.append(messages)
        return self.responses.pop(0)

def test_low_confidence_escalates_without_drafting():
    model = FakeModel([
        tool_call("search_past_tickets", {"query": "invoice double charge"}),
        final({"category": "billing", "confidence": 0.42, "draft_reply": None}),
    ])
    result = run_triage(ticket, model=model, tools={"search_past_tickets": fake_search})

    assert result.escalated is True
    assert result.draft_reply is None          # never draft below the threshold
    assert len(model.calls) == 2               # no extra model calls burned
```

This is where you assert the things the spec promised: that a confidence below 0.7 escalates, that the agent never sends without approval, that `max_steps` actually halts a loop. **Assert on structure — tool calls, sequence, schema, decisions — never on the model's prose.** A test that pins wording fails on the next model release for no reason, and a suite that cries wolf is one the team learns to ignore.

For provider-level tests, record real responses once and replay them as fixtures. Refresh the recordings deliberately when you change model or prompt version — the refresh diff is a useful review artefact in itself.

## Test the failure paths first

Most interesting production behaviour is a failure path, and those are exactly what never gets exercised by hand:

- **Tool times out** → structured error returned → agent continues without that input, or escalates. ([error handling](../02-design/error-handling.md))
- **Tool returns 429 / 5xx** → retry with backoff → succeeds on the second attempt; and a 4xx is *not* retried.
- **Model returns malformed output** → schema validation fails → one repair attempt → escalate rather than loop.
- **Step or budget limit reached** → the run stops cleanly with a partial result, no runaway.
- **Retrieval returns nothing** → the agent says so instead of inventing a citation.
- **A write is retried** → the idempotency key prevents a duplicate side effect ([integrations & auth](integrations-and-auth.md)).

Each of these is a fast, boring unit test — and each is a production incident you do not have.

## Snapshot the things the model reads

A tool's name, description, and parameter docs are prompt ([tool design](../02-design/tool-design.md)) — editing a description changes behaviour as surely as editing code. Snapshot the serialised tool definitions and system prompt, not to freeze them but to make every change a visible diff that triggers a regression run ([regression & drift](../04-evaluation/regression-and-drift.md)).

## In CI

- **Every PR** — unit, schema/snapshot, loop, and guardrail tests. No API keys, no network, under a minute.
- **Nightly** — adapter contract tests against sandbox credentials, so third-party drift is caught by a build rather than by a user.
- **On any prompt, model, or tool-definition change** — the eval set, gated on the release criteria from the agent spec.
- **Pre-deploy** — smoke tests on a couple of golden cases with the real model.

Keep the model out of the fast lane: a suite needing API keys and 40 s per case stops being run.

## References

- [pytest](https://docs.pytest.org/) — fixtures and parametrisation for the deterministic layers.
- [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — tool definitions as prompt, and iterating on them against real tasks.
