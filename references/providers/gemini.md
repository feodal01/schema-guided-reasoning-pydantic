# Google Gemini (structured output transports)

Gemini exposes **two** ways to get typed JSON. Behavior differs between **native** Gemini structured output and **OpenAI-compatible** HTTP surfaces—plan SGR transports accordingly.

**Python (`google-genai`):** `GenerateContentConfig` + `model_json_schema()` → [`../api-call-examples.md`](../api-call-examples.md#gemini-python-sdk-json-schema).

---

## Native Gemini structured output (`google-genai` SDK)

Generally more permissive around optional/nullable fields and nested schemas than strict OpenAI Structured Outputs. Still validate exported schema for your SDK/version.

Apply SGR in-schema (route literals, branch containers, reasoning order); do not rely on descriptions alone.

---

## OpenAI-compatible route (`/v1beta/openai/`)

Observed behavior from real calls (Gemini 2.5 Flash, Gemini 2.0 Flash):

### Confirmed working (simple to moderately complex schemas)

- `enum` (including single-value `Literal` route locks) ✅
- `additionalProperties: false` ✅
- `minItems` / `maxItems` on arrays ✅
- `minimum` / `maximum` on numbers ✅
- `pattern` on strings ✅ (see stability note below)
- Nullable required fields (`type: ["string", "null"]`) ✅
- `strict: true` in the request (accepted without error) ✅

### Critical: `max_tokens` must be generous

**Gemini 2.5 Flash is verbose.** When the schema includes open-ended `string` fields (e.g. `reasoning: str`), the model generates longer text than you might expect. If `max_tokens` is too small, Gemini truncates the output mid-JSON, producing unterminated strings or empty responses that fail JSON parsing.

**Observed:** with `max_tokens=200` and a schema containing a `reasoning` field, Gemini 2.5 Flash returned invalid JSON in 4/5 runs. With `max_tokens=800`, it returned valid JSON in 5/5 runs.

**Rule:** when your SGR schema contains reasoning fields or other long strings, set `max_tokens` to at least **800** on this transport. For multi-finding Cycle schemas, use more.

### Pattern stability

`pattern` on strings passes most of the time but showed ~1 in 5 empty responses in testing. Treat it as probabilistically enforced rather than hard-guaranteed. Back it up with service-side `model_validate`.

### SGR implication

- Keep a **canonical** semantic schema for your domain (branch literals, Cascade order, split branch lists).
- For this transport: set generous `max_tokens` whenever schemas contain open-ended string fields.
- Treat string-length bounds and some array bounds as post-validation only unless proven reliable for the exact schema shape you ship.
- Prefer **simple branch containers** when complex nested constraints are rejected.

When in doubt, measure on the exact API version and schema you ship.
