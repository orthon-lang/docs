# Type-Level Null Safety

> **✅ ACCEPTED — [EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`NULL_SAFETY.md`](NULL_SAFETY.md),
> [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Option Type, Narrowing,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

EDR-018 established that Orthon has no `null` sentinel — absence is tracked via `Option<T>`. But without type-level narrowing, every `match` on `Option` requires manual unboxing even when the programmer knows the value is present in the matching branch.

The core problem: **pattern matching on `Option<T>` should automatically inform the type system that the value is `T` inside the matching arm**, eliminating manual unwrap calls and protecting against accidental `None` dereferences.

Type-level null safety extends EDR-018's null safety model by adding flow-sensitive type narrowing — the compiler tracks whether a value is definitely non-null after a check, without requiring explicit type annotations for the narrowed state.

## Principles

1. **`Option<T>` and `T` are distinct types** — Assignment of `None` to a non-optional `T` is a compile-time error.
2. **Narrowing after pattern match** — After `match value { case Some(x) => ... }`, the compiler knows `value` is `T` in the `Some` arm.
3. **Narrowing after explicit check** — After `if value != None { ... }`, the compiler narrows `value` to `T` in the true branch.
4. **Narrowing is per-variable and flow-sensitive** — Narrowing follows control flow and resets on variable reassignment.
5. **Conservative by default** — If the compiler cannot prove a value is non-null, it remains `Option<T>`. The `!` operator is the explicit escape hatch for the programmer's knowledge.
6. **`?T` syntactic sugar deferred to Phase 5** — The semantic model of type-level null safety is settled here. Concrete `?T` syntax for `Option<T>` shorthand is determined in Phase 5.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Narrowing Policy | Controls how the compiler narrows `Option<T>` to `T` after checks |
| Exhaustiveness Policy | Ensures narrowing is only applied when the check is exhaustive |
| Reassignment Policy | Governs narrowing reset on variable reassignment |

## Model (What)

### Narrowing After Pattern Match

When matching on `Option<T>`, the compiler narrows the type in each arm:

```orthon
match opt_value:
    case Some(x) => process(x)       # x is T — narrowed
    case None => handle_missing()    # opt_value is None — narrowed
```

After the `match` expression, the narrowed state of the original variable is determined by the arms: if all arms return or the `match` is exhaustive, the original variable is no longer accessible in its pre-match state.

### Narrowing After Explicit Check

Control-flow-based narrowing works with `if` expressions:

```orthon
if opt_value != None:
    # opt_value is narrowed to T here
    let result = opt_value.some_method()  # OK: T methods available
else:
    # opt_value is None
    handle_missing()

# After the if-else, opt_value returns to Option<T>
```

### Narrowing Chain

Multiple narrowing checks compose:

```orthon
if opt_a != None and opt_b != None:
    # Both opt_a and opt_b are narrowed to their inner types
    process(opt_a, opt_b)
```

### Narrowing and Reassignment

When a narrowed variable is reassigned, narrowing resets:

```orthon
if opt_value != None:
    # opt_value is T
    opt_value = None      # reassignment — narrowing resets
    # opt_value is Option<T> again
```

### The `!` Escape Hatch

When the programmer knows a value is present but the compiler cannot prove it:

```orthon
let value = maybe_none!    # unwrap with panic if None — type is T
```

This does not narrow the original variable — it produces type `T` at the usage site.

### Interaction with Pattern Matching (EDR-025)

Narrowing extends naturally to pattern matching on `Option`:

```orthon
match result:
    case Ok(data) =>                # data is T — narrowed
        process(data)
    case Error(err) =>              # value is Error — narrowed
        handle_err(err)
```

### Deferred `?T` Syntax

The `?T` shorthand for `Option<T>` is deferred to Phase 5 (Syntax). Until then, `Option<T>` is the only form.

## Default Strategy

Narrowing after `match` on `Option<T>` and after explicit `!= None` checks. Flow-sensitive tracking within function bodies. Conservative narrowing — if unsure, the type stays `Option<T>`. Reset on reassignment. The `!` operator for escape-hatch unwrapping.

## Alternative Strategies

| Strategy | Description |
|---|---|
| No narrowing (always `Option<T>` after match) | Safe but requires `!` after every `Some` check. Defeats the ergonomic benefit of pattern matching |
| Global flow analysis (function-call aware) | Narrowing across function call boundaries. More powerful but fragile — inferred contracts are not visible in source |
| TypeScript-style flow typing | Narrowing on type predicates rather than concrete patterns. Less precise than match-based narrowing |
| Kotlin-style smart casts | Narrowing on `is` checks and safe casts. Well-proven in practice for single-nullable scenarios |

## Open Questions

1. Should narrowing apply to compound conditions — `if opt_a != None and opt_a.is_valid()` — where the second condition depends on the first's narrowed type?
2. How does narrowing interact with the Metadata Protocol (`@`) — should `value@is_some()` trigger narrowing?
3. Should narrowing work with `match` on `Result<T, E>` (narrowing `Ok` to `T`)?
4. Should narrowing cross function call boundaries when the callee is inlined?

## Decision History

- **Narrowing after pattern match on `Option<T>`** adopted over no narrowing. Rationale: Eliminates manual `!` unwrap after every `Some` check — ergonomic null safety.
- **Conservative narrowing** adopted over aggressive flow analysis. Rationale: Safe by construction — never incorrectly narrows. The `!` escape hatch covers cases where the programmer knows more than the compiler.
- **Flow-sensitive, per-variable** adopted over global. Rationale: Local tracking is simpler and more predictable. Reassignment reset prevents stale narrowing.
- **Accepted via EDR-028** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/NULL_SAFETY.md`
- [x] `what/concepts/PATTERN_MATCHING.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
