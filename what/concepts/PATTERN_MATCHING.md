# Pattern Matching

> **✅ ACCEPTED — [EDR-025](../how/decision_records/architecture/EDR-025-pattern-matching.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`TRAITS.md`](TRAITS.md),
> [`PATTERN_MATCHING_DISPATCH.md`](PATTERN_MATCHING_DISPATCH.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Pattern Matching, Exhaustiveness,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Cascading `if-else if-else` chains degrade readability as conditions grow. Nested conditionals create "arrow code" where the programmer must track multiple predicates simultaneously. Missed `else` branches cause subtle bugs. The compiler cannot verify that all cases are handled.

The core problem: **conditions mix data structure inspection with control flow**, forcing the programmer to manually decompose values rather than declaring the shape they expect.

Pattern matching transforms conditional logic into **declarative structure description**: the programmer describes the shape a value should take; the compiler generates the decision tree.

## Principles

1. **Exhaustiveness required** — The compiler verifies that all cases of a given type or value domain are covered. Missing cases are a compile-time error.
2. **Expression-oriented** — `match` produces a value, enabling assignment and composition.
3. **Decomposability** — Patterns decompose compound values by structure (tuples, records, variants), not just by value equality.
4. **First-match precedence** — The first matching arm wins. The compiler may warn about unreachable arms.
5. **Guards are predicates, not patterns** — Guards add conditional checks after the pattern shape matches. They are not part of the pattern itself.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Exhaustiveness Policy | Determines strictness — compile-time error for missing cases |
| Pattern Complexity Policy | Limits nesting depth and guard complexity |
| Ownership Policy | Governs consumption vs. borrowing of the scrutinee in match |

## Model (What)

### Match Expression

```orthon
let result = match value:
    case Pattern1 => expression1
    case Pattern2 if guard => expression2
    case _ => default_expression
```

All arms must produce the same type (or a type coercible to it).

### Destructuring

Patterns decompose compound values by structure:

```orthon
// Tuple destructuring
match point:
    case (0, 0) => "origin"
    case (x, 0) => "on x-axis at ${x}"
    case (0, y) => "on y-axis at ${y}"
    case (x, y) => "at (${x}, ${y})"

// Record destructuring
match user:
    case User(name, role: Admin) => "admin ${name}"
    case User(name, _) => "user ${name}"

// Variant destructuring
match result:
    case Ok(data) => process(data)
    case Error(err) => handle(err)
```

### Wildcard `_`

The wildcard pattern matches any value without binding it:

```orthon
match value:
    case Some(x) => process(x)
    case _ => default
```

### Guards

Guards add conditional predicates after a pattern matches:

```orthon
match number:
    case n if n > 0 => "positive"
    case n if n < 0 => "negative"
    case _ => "zero"
```

### Or Patterns

Multiple patterns with the same body:

```orthon
match value:
    case A | B => handle_ab()
    case C => handle_c()
    case _ => default()
```

### Exhaustiveness

The compiler verifies all cases are covered. For sum types, every variant must appear (explicitly or via wildcard):

```orthon
// Compile error: missing cases
match option:
    case Some(x) => process(x)
    // Error: `None` not handled

// OK: wildcard covers remaining cases
match option:
    case Some(x) => process(x)
    case _ => default
```

### Binding and Ownership

Matching consumes the scrutinee by default. Borrowing is explicit:

```orthon
match value:           # consumes value
    case Some(x) => process(x)

match &value:          # borrows value
    case Some(x) => process(x)    # x is a reference
```

## Default Strategy

Exhaustive matching with `match` as expression. First-match precedence. Guards evaluated after structural matching. Wildcard `_` as catch-all. Consumption by default, borrowing via explicit `&`. Or patterns supported natively.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Total match (no wildcard) | All cases must be listed explicitly — no catch-all. Used for closed type hierarchies where completeness can be proven |
| Fallible match | Matching produces `Option<T>`; unmatched cases return `None`. Useful for partial matching where inexhaustiveness is acceptable |
| Prioritised guards | Guards evaluated before structural matching (rare, used in theorem provers). Not adopted for Orthon |
| Non-exhaustive (warnings only) | Missing cases produce warnings, not errors. Loses the primary safety guarantee |

## Open Questions

1. Should match on non-exhaustive types produce a warning or an error when the wildcard is omitted?
2. How deep should nested pattern decomposition be allowed before the compiler flags it? (Performance cliff prevention.)
3. Interaction with ownership: should match on a reference produce reference bindings in destructured subpatterns?
4. Should `match` support irrefutable patterns (single-arm match for destructuring only)?

## Decision History

- **Exhaustiveness required (compile-time error)** adopted over warnings-only. Rationale: Safety guarantee is the primary value — warnings are frequently ignored.
- **Expression-oriented** adopted over statement-oriented. Rationale: Composition with assignment, return, and other expression contexts.
- **First-match precedence** adopted over specificity rules. Rationale: Simpler mental model — the programmer controls ordering.
- **Accepted via EDR-025** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/TRAITS.md`
- [ ] `what/concepts/PATTERN_MATCHING_DISPATCH.md`
- [ ] `what/concepts/NULL_SAFETY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
