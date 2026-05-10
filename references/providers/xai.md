# xAI (Grok)

xAI's API is OpenAI-compatible. Use the OpenAI Python SDK with the xAI base URL.

**Python:** same pattern as OpenAI — `chat.completions.create` with `response_format` → [`../api-call-examples.md`](../api-call-examples.md#openai-python-sdk-parse) (substitute xAI base URL and model).

---

## Adversarial enforcement results (grok-3-mini, 5/5 runs each)

All tested constraints were fully enforced even when the prompt explicitly instructed the model to violate the schema:

| Constraint | Result |
|---|---|
| `maxItems` / `minItems` on arrays | ✅ enforced |
| `maximum` / `minimum` on numbers | ✅ enforced |
| `pattern` on strings | ✅ enforced |
| Single-value `enum` / Literal route lock | ✅ enforced |
| Required fields | ✅ enforced |
| `additionalProperties: false` | ✅ enforced |

Unlike OpenAI, xAI does not require `strict: true` for constraint enforcement to apply.

---

## SGR implication

Use the same schema you designed for OpenAI (with `strict: true`). No special transport derivative needed for xAI based on current observations. Still validate server-side—empirical coverage here is one model and one model version.

---

## Available models (as of May 2026)

`grok-3`, `grok-3-mini`, `grok-4-*` series. Check `/v1/models` for the current list.
