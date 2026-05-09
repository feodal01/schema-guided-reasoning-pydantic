# Cross-cutting caveats (all providers)

These apply regardless of which structured-output API you call.

## Schema descriptions are not a reliable policy channel

Some runtimes treat `Field(description=...)` as hidden prompt context; others ignore it.

- Do not put mandatory behavioral rules only in descriptions.
- Encode routing and admissibility in types (`Literal`, required fields, bounded collections).
- Duplicate critical policy in the prompt when it must hold.

**Smoke test:** ship a tiny schema whose only instruction lives in `description`. If behavior ignores it, descriptions are not dependable in your stack.

## Over-constrained schemas hurt accuracy

Constrained decoding can truncate reasoning quality when the schema is tighter than necessary.

**Mitigation via SGR:** keep mandatory checkpoints (Cascade), explicit branches (Routing), bounded repetition (Cycle), and drop decorative constraints that do not change decisions.

## Prompt + schema are joint contracts

- Prompt carries task policy and edge handling.
- Schema carries the output protocol, branch locks, and reasoning checkpoints.

Neither layer alone should be expected to substitute for the other.
