# Cross-cutting caveats (all providers)

These apply regardless of which structured-output API you call.

---

## Schema constraints without enforcement are just hints

Passing a JSON Schema to the API does not automatically enforce every field in it. **Whether a constraint is actually enforced depends on the provider and how you call the API.**

From adversarial testing (prompt instructs model to violate the schema):

| Constraint | OpenAI (no strict) | OpenAI (strict=True) | Gemini OpenAI-compat | xAI |
|---|---|---|---|---|
| `enum` / Literal route lock | ❌ not enforced | ✅ | ✅ | ✅ |
| Required fields | ❌ | ✅ | ✅ | ✅ |
| `additionalProperties: false` | ❌ | ✅ | ✅ | ✅ |
| `maximum` / `minimum` | ❌ | ✅ | ✅ | ✅ |
| `maxItems` / `minItems` | ❌ | ✅ | ✅ / ⚠️ partial | ✅ |
| `pattern` | ❌ | ✅ | ❌ not enforced | ✅ |

**Consequence:** always add service-side `model_validate` / post-validation after parsing. Treat constraints the provider does not enforce as policy (prompt + validation), not schema guarantees.

---

## OpenAI: `strict: true` is mandatory

Without `strict: true`, OpenAI enforces none of the constraints above—the model follows the prompt, not the schema. See [`providers/openai-structured-outputs.md`](./providers/openai-structured-outputs.md).

---

## Schema descriptions are not a reliable policy channel

Some runtimes treat `Field(description=...)` as hidden prompt context; others ignore it.

- Do not put mandatory behavioral rules only in descriptions.
- Encode routing and admissibility in types (`Literal`, required fields, bounded collections).
- Duplicate critical policy in the prompt when it must hold.

**Smoke test:** ship a tiny schema whose only instruction lives in `description`. If behavior ignores it, descriptions are not dependable in your stack.

---

## Over-constrained schemas hurt accuracy

Constrained decoding can truncate reasoning quality when the schema is tighter than necessary.

**Mitigation via SGR:** keep mandatory checkpoints (Cascade), explicit branches (Routing), bounded repetition (Cycle), and drop decorative constraints that do not change decisions.

---

## Prompt + schema are joint contracts

- Prompt carries task policy and edge handling.
- Schema carries the output protocol, branch locks, and reasoning checkpoints.

Neither layer alone substitutes for the other.
