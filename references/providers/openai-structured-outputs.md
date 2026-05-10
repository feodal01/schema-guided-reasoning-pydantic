# OpenAI Structured Outputs

Rules and nuances for the **OpenAI Structured Outputs** transport. Canonical upstream guide: [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs).

**Python:** minimal `chat.completions.parse` + Pydantic example → [`../api-call-examples.md`](../api-call-examples.md#openai-python-sdk-parse).

---

## Critical: `strict: true` is mandatory

**Without `strict: true`, OpenAI enforces nothing.** Adversarial tests (prompt instructs the model to violate the schema) on `gpt-4o-mini` without strict confirmed: all constraints—`enum`, required fields, `additionalProperties`, numeric bounds, `pattern`, `maxItems`—were ignored 5/5 times. The model followed the prompt, not the schema.

**With `strict: true`, all tested constraints were enforced (adversarial runs):**

| Constraint | Result |
|---|---|
| `enum` / Literal route lock | ✅ |
| Required fields | ✅ |
| `additionalProperties: false` | ✅ |
| `maximum` / `minimum` | ✅ |
| `exclusiveMaximum` / `exclusiveMinimum` (`lt` / `gt`) | ✅ |
| `maxLength` / `minLength` (string) | ✅ |
| `multipleOf` | ✅ |
| `maxItems` / `minItems` | ✅ |
| `pattern` | ✅ |
| `anyOf` (nullable `T \| None`, `Union[A, B]`) | ✅ |
| `$defs` / `$ref` reuse | ✅ |
| `oneOf` (discriminated union) | 💥 400 API error |
| `allOf` (inheritance) | 💥 400 API error |

Always set `strict: true`. Without it, your schema is a hint, not a contract.

```python
response_format={
    "type": "json_schema",
    "json_schema": {"name": "my_schema", "schema": MyModel.model_json_schema(), "strict": True}
}
```

---

## Role in an SGR workflow

Treat this as a **transport profile**: field order, branch locks, and reasoning checkpoints are SGR's job; strict mode is what makes the API server actually honour them.

---

## Contract (strict subset)

- Root must be an `object`. No top-level `anyOf`.
- Every field must appear in `required`. Model optional values as `T | null` (required, nullable), not as omitted fields.
- Every object must set `additionalProperties: false`. In Pydantic: `ConfigDict(extra="forbid")`.
- Key ordering matters: OpenAI tends to emit keys in schema declaration order—use this as an SGR control surface (reasoning and route locks first).
- `$defs` / recursion: supported; reuse instead of duplicating large trees.

---

## Hard limits (from OpenAI guide)

- At most **5,000** object properties total.
- At most **10** nesting levels.
- At most **120,000** characters across property names, definition names, enum values, and const values.
- At most **1,000** enum values in the whole schema.
- Single string enum with **> 250** values: combined length must stay under 15,000 characters.

---

## Keywords that are "unsupported" but work in practice

OpenAI documents these as unsupported for fine-tuned models, yet adversarial tests on base models (`gpt-4o-mini`) show they are enforced with `strict: true`:

- `maximum` / `minimum`, `exclusiveMaximum` / `exclusiveMinimum`
- `maxItems` / `minItems`, `maxLength` / `minLength`
- `multipleOf`, `pattern`

Use them, but know they may fail on fine-tuned model variants. Back up with server-side `model_validate` anyway.

**Cause 400 API errors** (rejected at request time with `strict=True`): `oneOf`, `allOf`, `not`, `dependentRequired`, `dependentSchemas`, `if`, `then`, `else`.

---

## Caller responsibilities

Schema guarantees apply to **successful** generations. The API can return `refusal` or the response may be `incomplete`. **Branch on these before parsing into your Pydantic model.**

---

## Preflight checklist

- [ ] `strict: true` is set in the request.
- [ ] Root is an object; no top-level `anyOf`.
- [ ] Every object has `additionalProperties: false`.
- [ ] Every field is in `required`; optional semantics use `T | null`.
- [ ] Property / nesting / enum / string length counts within documented limits.
- [ ] No unsupported composition keywords (`oneOf`, `allOf`, `not`, `if/then/else`) — these cause 400 errors.
- [ ] Caller handles `refusal` / `incomplete` before parsing.
