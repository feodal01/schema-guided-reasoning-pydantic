# Cross-cutting caveats (all providers)

These apply regardless of which structured-output API you call.

---

## Schema constraints without enforcement are just hints

Passing a JSON Schema to the API does not automatically enforce every field in it. **Whether a constraint is actually enforced depends on the provider and how you call the API.**

> ⚠️ **This data was collected at a point in time.** Provider SO implementations change. Treat the table below as a starting reference, not a permanent guarantee. [Test your schema against the actual provider before relying on it.](#empirical-verification-workflow)

From adversarial testing (prompt instructs model to violate the schema, 3–5 runs per cell):

### Scalar / value constraints

| Constraint | Pydantic field | OpenAI (no strict) | OpenAI (strict=True) | Gemini OpenAI-compat | xAI |
|---|---|---|---|---|---|
| `enum` / Literal route lock | `Literal[...]` | ❌ | ✅ | ✅ | ✅ |
| `maximum` / `minimum` | `Field(le=, ge=)` | ❌ | ✅ | ✅ | ✅ |
| `exclusiveMaximum` / `exclusiveMinimum` | `Field(lt=, gt=)` | ❌ | ✅ | ❌ | ✅ |
| `maxLength` / `minLength` (string) | `Field(max_length=, min_length=)` | ❌ | ✅ | ❌ | ✅ |
| `multipleOf` | `Field(multiple_of=)` | ❌ | ✅ | ❌ | ✅ |
| `pattern` | `Field(pattern=)` | ❌ | ✅ | ❌ | ✅ |

### Collection / structure constraints

| Constraint | Pydantic field | OpenAI (no strict) | OpenAI (strict=True) | Gemini OpenAI-compat | xAI |
|---|---|---|---|---|---|
| Required fields | default BaseModel | ❌ | ✅ | ✅ | ✅ |
| `additionalProperties: false` | `ConfigDict(extra="forbid")` | ❌ | ✅ | ✅ | ✅ |
| `maxItems` / `minItems` | `Field(max_length=, min_length=)` on list | ❌ | ✅ | ✅ / ⚠️ partial | ✅ |
| Nested object (2 levels) | nested `BaseModel` | ❌ | ✅ | ✅ | ✅ |
| `$defs` / `$ref` reuse | shared sub-models | ❌ | ✅ | ⚠️ partial¹ | ✅ |

### Composition keywords

| Keyword | OpenAI (strict=True) | Gemini OpenAI-compat | xAI |
|---|---|---|---|
| `anyOf` (nullable `T \| None`) | ✅ | ✅ | ✅ |
| `anyOf` (`Union[A, B]`) | ✅ | ✅ | ✅ |
| `oneOf` (discriminated union) | 💥 400 API error | ✅ | ✅ |
| `allOf` (schema inheritance) | 💥 400 API error | ❌ returns `{}` | ❌ returns `{}` |

¹ Gemini resolves `$defs` for structure, but `const` values inside `$defs` refs may not be enforced (model followed the prompt instead). Use `enum` in direct schemas rather than `const` inside `$defs` on Gemini.

**Consequence:** always add service-side `model_validate` / post-validation after parsing. Treat constraints the provider does not enforce as policy (prompt + validation), not schema guarantees.

---

## Empirical verification workflow

Before relying on any constraint as a hard guarantee in your stack:

1. Write a **minimal schema** with only that constraint.
2. Send a prompt that explicitly instructs the model to violate it (adversarial).
3. Run ≥ 3 times (provider output can be non-deterministic).
4. If ≥ 1 violation is observed → constraint is not reliably enforced → validate server-side.

Provider SO implementations can change without notice (new model version, API update). Re-verify after provider upgrades if the constraint is safety/correctness critical.

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
