# Google Gemini (structured output transports)

Gemini exposes **more than one** way to get typed JSON. Behavior differs between **native** Gemini structured output and **OpenAI-compatible** HTTP surfaces—plan SGR transports accordingly.

**Python (`google-genai`):** `GenerateContentConfig` + `model_json_schema()` → [`../api-call-examples.md`](../api-call-examples.md#gemini-python-sdk-json-schema).

## Native Gemini structured output

Generally more permissive around optional/nullable fields and nested schemas than strict OpenAI Structured Outputs. Still validate exported schema for your SDK/version.

Still apply SGR in-schema (route literals, branch containers, reasoning order); do not rely on descriptions alone.

## OpenAI-compatible route (`/v1beta/openai/`)

Observed behavior can be **uneven** vs strict OpenAI documentation: some JSON Schema constraints are enforced strongly; others are flaky or rejected depending on schema complexity.

### Reported / observed patterns (treat as empirical, re-verify for your build)

- **`maxItems`:** may return `INVALID_ARGUMENT` on **complex** nested schemas; can still work on **simple** shapes. Treat as **compatibility-sensitive**, not universally safe.
- **`min_length` (string):** not reliable—calls may accept strings shorter than the declared minimum.
- **`max_length` (string):** not a hard guarantee—mix of clipping, over-length strings, and later Pydantic failures depending on prompt and shape.
- **Often reliable on simpler schemas:** `enum`, `pattern`, numeric bounds, `format: date`, `additionalProperties: false`, `minItems`.

### SGR implication

- Keep a **canonical** semantic schema for your domain (branch literals, Cascade order, split branch lists).
- For this transport, assume **string length bounds** and some **array bounds** may need **prompt-level** and **service-side** enforcement unless you have proven them for the exact schema version.
- Prefer **simple branch containers** when nested union-heavy shapes are rejected.

When in doubt, **measure** on the exact API version and schema you ship.
