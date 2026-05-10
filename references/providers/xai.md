# xAI (Grok)

xAI's API is OpenAI-compatible. Use the OpenAI Python SDK with the xAI base URL.

**Python:** same pattern as OpenAI — `chat.completions.create` with `response_format` → [`../api-call-examples.md`](../api-call-examples.md#openai-python-sdk-parse) (substitute xAI base URL and model).

---

## Observed behavior (Grok 3 Mini)

All features tested worked reliably (3/3 runs each):

| Feature | Result |
|---------|--------|
| Basic `response_format` JSON Schema | ✅ |
| `strict: true` in request | ✅ (accepted without error) |
| `additionalProperties: false` | ✅ enforced |
| Single-value `enum` (Literal route lock) | ✅ |
| Nullable required fields (`type: ["string", "null"]`) | ✅ |
| `minItems` / `maxItems` on arrays | ✅ |
| `minimum` / `maximum` on numbers | ✅ |
| `pattern` on strings | ✅ |

## SGR implication

Use the same schema you designed for OpenAI Structured Outputs. No special transport derivative needed for xAI based on current observations. Still validate server-side—empirical coverage here is limited to a basic feature matrix.

## Available models (as of May 2026)

`grok-3`, `grok-3-mini`, `grok-4-*` series. Check `/v1/models` for the current list.
