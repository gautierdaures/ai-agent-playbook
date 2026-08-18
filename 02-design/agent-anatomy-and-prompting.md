# Agent anatomy & prompting

## Anatomy of an LLM agent

An LLM agent combines three parts:

- **Perception** — text and memory (context).
- **Reasoning** — analyze, plan, reflect.
- **Action** — call tools, produce output.

The agent's working state and memory live in a semi-structured object, for example JSON.

## Prompting

Use a prompt framework for structure:

- **Frameworks** — COSTAR, RISEN, CCR.
- **Contents** — include a step-by-step instruction and the target audience.

Prompt engineering means describing something that knows the objective very well. The semantics matter a lot; wording carries real weight.

### Markdown or XML?

Both work. For Claude specifically, structuring a prompt with **XML tags** (e.g. `<instructions>`, `<context>`, `<example>`) helps the model separate the parts cleanly and is the documented best practice. Markdown headings are fine for lighter prompts; the key is to be consistent and unambiguous. See also [prompt versioning](../03-build/rag-and-tooling.md).

## Good practices for an agent prompt

- **Role** — who the agent is.
- **Available tools** — what it can call.
- **Output format** — for example a Pydantic schema.
- **Ethical constraints** — state them explicitly.
- **Concise and precise** — keep it tight.
- **AI Act** — integrate it into the prompt (see [security & compliance](security-and-compliance.md)).

## Meta

You can use an agent itself to list frameworks or to refactor your prompts.

## References

- **COSTAR** (Context, Objective, Style, Tone, Audience, Response) — introduced by Sheila Teo, winner of Singapore's first GPT-4 prompt-engineering competition.
- **RISEN** (Role, Instructions, Steps, End goal, Narrowing) — by Kyle Balmer; see the [RISEN write-up](https://dev.to/gunnargrosch/writing-system-prompts-that-actually-work-the-risen-framework-for-ai-agents-4p94).
- [Anthropic — Use XML tags to structure your prompts](https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
