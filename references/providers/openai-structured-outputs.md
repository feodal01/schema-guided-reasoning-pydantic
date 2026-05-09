# OpenAI Structured Outputs

Rules and nuances for the **OpenAI Structured Outputs** transport (strict JSON Schema subset). Canonical upstream guide: [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs).

**Python:** minimal `chat.completions.parse` + Pydantic example → [`../api-call-examples.md`](../api-call-examples.md#openai-python-sdk-parse).

## Role in an SGR workflow

Treat this API as a **transport profile**: it constrains which JSON Schema shapes are legal at request time and how keys are emitted on success. SGR still owns *what* must be reasoned and in *what order*; OpenAI’s profile defines *how* that must be expressible in schema for this endpoint.

## Contract (strict subset)

- JSON Schema is **not** full draft support—only a documented strict subset.
- **Root** must be an `object`. **No** top-level `anyOf`.
- **Every** object field must appear in `required`. Model “optional” values as a union with `null` (in JSON Schema terms, e.g. `type: [T, "null"]`), not by omitting keys from `required`.
- **Every** object must set `additionalProperties: false`. In Pydantic: `ConfigDict(extra="forbid")` on each exported object model.
- **Key order:** the model tends to emit object keys in **schema declaration order**—treat field order as a first-class SGR control (reasoning and route locks first).
- **`$defs` / recursion:** supported; prefer reuse over duplicating large trees.
- **`strict: true`:** unsupported keywords and shapes **fail at request time**—preflight the exported schema before production calls.

## Hard limits (from OpenAI’s guide)

- At most **5,000** object properties (total).
- At most **10** nesting levels.
- At most **120,000** characters across property names, definition names, enum values, and const values.
- At most **1,000** enum values in the whole schema.
- If a single string enum has **> 250** values, combined enum string length must stay under **15,000** characters.

## Unsupported / risky keywords (strict mode)

**Generally unsupported** composition and conditionals, including: `allOf`, `not`, `dependentRequired`, `dependentSchemas`, `if`, `then`, `else`.

**Fine-tuned models (additional avoid list):** string `minLength`, `maxLength`, `pattern`, `format`; number `minimum`, `maximum`, `multipleOf`; object `patternProperties`; array `minItems`, `maxItems`.

## Response path vs schema

- Schema guarantees apply to **successful** structured generations.
- The API can return **`refusal`** or **`incomplete`** (or other non-success paths). The **caller must branch** on these before assuming the body matches your Pydantic model.

## Pydantic habits

- `ConfigDict(extra="forbid")` on every object that is exported to this API.
- Review **exported** JSON Schema, not only Python types.
- Add a **preflight** step: reject top-level unions, missing `additionalProperties: false`, oversize / over-nested schemas, and known-bad keywords for your model class.

## Preflight checklist (OpenAI)

- [ ] Root is an object; no top-level `anyOf`.
- [ ] Every object has `additionalProperties: false` / `extra="forbid"`.
- [ ] Every field is in `required`; optionality is `T | null` as required by the guide.
- [ ] Counts: properties, depth, enum sizes, total string length within documented limits.
- [ ] No unsupported keywords for your model **and** transport.
- [ ] Caller handles `refusal` / `incomplete` before parsing into the target model.
