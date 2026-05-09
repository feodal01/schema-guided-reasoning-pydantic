# Schema-Guided Reasoning for Pydantic

An LLM skill for building strict Pydantic schemas for structured output using **Schema-Guided Reasoning (SGR)**.

## What problem it solves

LLM APIs without a JSON Schema on the structured-output endpoint don't guarantee valid JSON on every call—agents silently fail when parsing breaks. This skill covers two things:

1. How to build a correct Pydantic-derived JSON Schema for **constrained decoding** so the model server enforces the shape.
2. How to use **Schema-Guided Reasoning (SGR)** to go further—encoding the reasoning *path* (ordered checkpoints, branch locks, bounded lists) into the schema itself so the model thinks in the structure you need, not around it.

## Who uses this

Engineers building LLM-as-judge pipelines, evaluators, classifiers, extraction workflows, and agent handoff schemas where downstream code depends on which reasoning branch ran—not just whether the JSON parsed.

## How to use

```
Use the Schema-Guided Reasoning for Pydantic skill to design a strict schema
for an LLM judge that evaluates whether a generated answer is faithful
to source evidence.
```

The skill walks through:

- Building a **route matrix** (branch × codes × required fields × decision impact)
- Mapping branches to SGR patterns: **Cascade**, **Routing**, **Cycle**
- Encoding branch locks with `Literal[...]` fields
- Checking the exported JSON Schema against provider constraints
- Ready-to-copy API call examples for **OpenAI** (`chat.completions.parse`) and **Gemini** (`google-genai` + `model_json_schema()`)

## Contents

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions |
| `references/patterns.md` | SGR pattern playbook |
| `references/templates.md` | Copy-paste Pydantic templates |
| `references/cross-cutting-caveats.md` | Caveats that apply to all providers |
| `references/api-call-examples.md` | OpenAI and Gemini API call examples |
| `references/providers/openai-structured-outputs.md` | OpenAI Structured Outputs rules and limits |
| `references/providers/gemini.md` | Gemini native vs OpenAI-compatible quirks |

## Attribution

Based on Schema-Guided Reasoning patterns by Rinat Abdullin:
- Overview: https://abdullin.com/schema-guided-reasoning/
- Patterns: https://abdullin.com/schema-guided-reasoning/patterns
- Structured output caveats: https://abdullin.com/structured-output/
