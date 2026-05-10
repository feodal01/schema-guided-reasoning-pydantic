# Google Gemini (structured output transports)

Gemini exposes **two** ways to get typed JSON. Behavior differs between **native** Gemini structured output and **OpenAI-compatible** HTTP surfaces—plan SGR transports accordingly.

**Python (`google-genai`):** `GenerateContentConfig` + `model_json_schema()` → [`../api-call-examples.md`](../api-call-examples.md#gemini-python-sdk-json-schema).

---

## Native Gemini structured output (`google-genai` SDK)

More permissive around optional/nullable fields and nested schemas than strict OpenAI mode. Still validate exported schema for your SDK/version. Apply SGR in-schema (route literals, branch containers, reasoning order); do not rely on descriptions alone.

---

## OpenAI-compatible route (`/v1beta/openai/`)

Unlike OpenAI, Gemini enforces constraints **without** needing `strict: true` in the request. Tested on `gemini-2.0-flash` using adversarial prompts (prompt instructs the model to violate the schema):

### Enforced (adversarial, 5/5 runs)

| Constraint | Result |
|---|---|
| `maxItems` on arrays | ✅ enforced |
| `maximum` / `minimum` on numbers | ✅ enforced |
| Single-value `enum` / Literal route lock | ✅ enforced |
| Required fields | ✅ enforced |
| `additionalProperties: false` | ✅ enforced |

### Not enforced

| Constraint | Result | Note |
|---|---|---|
| `pattern` on strings | ❌ 0/5 | Model follows prompt, ignores pattern |
| `minItems` on arrays | ⚠️ 4/5 | Partial: 1 run returned API error |

**`pattern` is not enforced on Gemini OpenAI-compatible.** Do not rely on it as a hard guarantee. Validate server-side.

### Critical: `max_tokens` must be generous

Gemini 2.5 Flash is verbose. When the schema contains open-ended `string` fields (e.g. `reasoning: str`), the model generates more text than a small token budget allows. With `max_tokens=200` and a reasoning field, responses were truncated mid-JSON in 4/5 runs. With `max_tokens=800`, all 5/5 passed.

**Rule:** set `max_tokens ≥ 800` for any SGR schema containing reasoning fields or other long strings. For multi-finding Cycle schemas, use more.

### SGR implication

- Keep a **canonical** semantic schema.
- For this transport: set generous `max_tokens`; back `pattern` with server-side validation; treat `minItems` as probabilistic.
- Prefer **simple branch containers** when complex nested constraints behave unexpectedly.
