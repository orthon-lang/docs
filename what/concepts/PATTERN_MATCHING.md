# Pattern Matching

> **✅ ACCEPTED — [EDR-025](../how/decision_records/architecture/EDR-025-pattern-matching.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`UNPACKING.md`](UNPACKING.md),
> [EDR-096](../../how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md)
> (destructuring pattern syntax: rest `*x`, `tuple(x)`, irrefutable-only outside `match`),
> [`TRAITS.md`](TRAITS.md),
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
6. **Refutable patterns only in `match`** — Literal, variant and `Some`/`Ok` patterns can fail, so they are legal only in `match`. Outside `match` (declarations, `for`, parameters) only irrefutable patterns are allowed ([UNPACKING.md](UNPACKING.md), EDR-096 D-13).

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
    case 0, 0 => "origin"
    case x, 0 => "on x-axis at ${x}"
    case 0, y => "on y-axis at ${y}"
    case x, y => "at (${x}, ${y})"

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

### Rest Pattern `*x`

The prefix star collects the remaining elements of a tuple pattern; `*_` discards them. A pattern contains exactly one star, at any position, and only in positional patterns (EDR-096 D-02, D-07):

```orthon
match xs:
    case first, *rest => first + sum(rest)    # rest collects the remaining elements
    case first, *_ => first                   # remainder discarded
    case first, *middle, last => first + last # star in the middle
    # case *a, *b => ...                      # error: more than one star
```

### Exactly One Element and the Empty Tuple

The 1-tuple pattern is the constructor form `tuple(x)`; the empty tuple is `()` (EDR-096 D-08):

```orthon
match xs:
    case () => 0                  # empty tuple
    case tuple(one) => one        # exactly one element
    case first, *rest => first + sum(rest)
```

### Pattern Form

Patterns in `case` are written as a comma list without parentheses; parentheses are needed only for nested patterns and for the empty tuple `()` (EDR-096 D-05). `var` before the pattern makes all of its names mutable (D-10):

```orthon
match nested:
    case (a, b), c => a + b + c
    case var first, *rest => consume(first, rest)
```

### Refutable and Irrefutable Patterns

A pattern is irrefutable when it matches every value of the scrutinee's type (a tuple pattern of names, a rest pattern, a constructor pattern of a single-constructor nominal type with name bindings). A pattern is refutable when it can fail: literal patterns, variant patterns, `Some`/`Ok` and similar. Refutable patterns are legal only in `match`. Declarations, `for` headers and parameters accept irrefutable patterns only (EDR-096 D-13), and a tuple pattern applied to an `Option<T>` is a type error (use `match`, `??` or flow-sensitive narrowing).

```orthon
Some(v) = maybe_value     # error outside match: variant pattern is refutable
x, y = point              # fine: irrefutable
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

> **Note:** the ownership wording in this section is subject to a tracked discrepancy with the copy-on-write value semantics ([COPY_ON_WRITE.md](COPY_ON_WRITE.md)); move versus borrow is to be decided jointly for all constructs (EDR-096 open item DEF-b).

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
4. Should `match` support irrefutable patterns (single-arm match for destructuring only)? (Tracked as open item DEF-c in EDR-096, together with let-else.)

## Decision History

- **Exhaustiveness required (compile-time error)** adopted over warnings-only. Rationale: Safety guarantee is the primary value — warnings are frequently ignored.
- **Expression-oriented** adopted over statement-oriented. Rationale: Composition with assignment, return, and other expression contexts.
- **First-match precedence** adopted over specificity rules. Rationale: Simpler mental model — the programmer controls ordering.
- **Accepted via EDR-025** on 2026-07-27.
- **2026-10-17** — Destructuring pattern syntax extended by EDR-096: rest pattern `*x`, `tuple(x)`, the one-star rule, and the rule that only irrefutable patterns are allowed outside `match`.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/TRAITS.md`
- [x] `what/concepts/UNPACKING.md`
- [ ] `what/concepts/PATTERN_MATCHING_DISPATCH.md`
- [ ] `what/concepts/NULL_SAFETY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
