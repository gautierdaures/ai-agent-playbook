# Agent anatomy & prompting

## Anatomy of an LLM agent

An LLM agent combines perception, reasoning, and action.

- **Reasoning** — covers analyze, plan, reflect.
- **State and memory** — live in a semi-structured object, for example JSON.

## Prompting

Use a prompt framework for structure:

- **Frameworks** — COSTAR, RISEN, CCR.
- **Contents** — include a step-by-step instruction and the target audience.

Prompt engineering means describing something that knows the objective very well. The semantics matter a lot; wording carries real weight.

## Good practices for an agent prompt

- **Role** — who the agent is.
- **Available tools** — what it can call.
- **Output format** — for example a Pydantic schema.
- **Ethical constraints** — state them explicitly.
- **Concise and precise** — keep it tight.
- **AI Act** — integrate it into the prompt (see [security & compliance](security-and-compliance.md)).

## Meta

You can use an agent itself to list frameworks or to refactor your prompts.
