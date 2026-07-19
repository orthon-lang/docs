# Pattern Matching

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

## Issue (Why)

Why do cascading `if-else if` chains produce poor code?

Conditional branching is fundamental to programming, but imperative cascades (`if-else if-else`) degrade readability as conditions grow. Nested conditionals create "arrow code" where the programmer must track multiple predicates simultaneously. Missed `else` branches cause subtle bugs. Exhaustiveness is never checked — the compiler cannot verify that all cases are handled.

The real problem: **conditions mix data structure inspection with control flow**, forcing the programmer to manually decompose values rather than declaring the shape they expect.

## Principles

1. **Exhaustiveness** — The language must verify at compile time that all cases of a given type or value domain are covered.
2. **Decomposability** — Matching must decompose compound data (tuples, records, enums) by structure, not just by value equality.
3. **Irrefutability** — Default/wildcard patterns must be available but discouraged — the compiler should prefer explicit case listing.
4. **Expression-oriented** — Match constructs must be expressions, not statements, enabling assignment from matching.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Policy | Determines which types are matchable (algebraic data types, enums, sealed traits) |
| Exhaustiveness Policy | Governs how strictly the compiler enforces complete coverage |
| Pattern Complexity Policy | Limits nesting depth or guards complexity to prevent pathological cases |

## Model (What)

Pattern matching transforms conditional logic into **declarative structure description**. The programmer describes the shape a value should take; the compiler generates the decision tree.

```
match value:
  case Pattern1 => result1
  case Pattern2 if guard => result2
  case _ => defaultResult
```

Key features:
- **Structural decomposition** — match on type constructors, tuple elements, record fields, and nested patterns.
- **Guards** — additional predicates on bound variables.
- **Exhaustiveness checking** — compiler warns or errors on missing cases.
- **Binding** — matched subvalues are automatically bound to variables.

## Default Strategy

By default, matching follows **pattern order precedence** (first match wins). Guards are evaluated only after the pattern shape matches. The compiler performs **exhaustiveness checking** as a warning unless explicitly configured as an error.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Total match | No wildcard allowed — all cases must be listed explicitly. |
| Fallible match | Matching produces an `Option`/`Result`; unmatched cases return `None`/`Error`. |
| Prioritised guards | Guards are evaluated before structural matching (rare, used in theorem provers). |

## Open Questions

1. Should matched variables be immutable by default?
2. How deep should nested pattern decomposition be allowed? (Performance cliff.)
3. Should `or` patterns (`case A | B =>`) be supported natively?
4. Interaction with ownership: does matching consume or borrow the scrutinee?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
