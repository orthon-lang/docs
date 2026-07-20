# Equality

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

Equality is one of the most subtle and error-prone concepts in programming
languages. The core problem: **reference equality and value equality are
fundamentally different operations**, yet most languages conflate them or
force the programmer to remember which operator does which.

In Python, `==` compares value equality for most built-in types but
reference equality for custom objects (unless `__eq__` is defined).
In Java, `==` is reference equality for objects and value equality for
primitives — a distinction that causes bugs daily. In JavaScript,
`==` performs type coercion while `===` does not, creating a three-way
confusion between reference, value, and coerced equality.

A language with an explicit semantics principle must make the equality
model predictable, unambiguous, and syntactically visible.

## Principles

1. **Explicit Equality** — Different kinds of equality (value, reference,
   structural) must use different syntax. The programmer should never have
   to guess which comparison is being made.
2. **Structural by Default** — Value comparison for compound data should
   be structural by default (field-by-field, recursively), not identity-based.
   This aligns with the Data-first philosophy: data is compared by its
   structure, not by its memory location.
3. **Consistency** — The same equality operator has the same semantics
   for all types. No special cases for primitives vs. objects.
4. **Transitivity** — Equality must be transitive: if `a == b` and `b == c`,
   then `a == c`. Non-transitive equality (e.g., floating-point NaN) must
   be explicitly documented and have a distinct operator.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Comparison Policy | Defines the default comparison algorithm (structural vs. identity) and custom comparison hooks |
| Allocation Policy | Affects whether identity comparison (`is`) is meaningful — arena-allocated values may have unstable addresses |
| Representation Policy | Different representations (Value, Reference, Tuple) may define equality differently |

## Model (What)

Equality in Orthon has three distinct operators, each with a clear semantic:

- `===` — **Value equality**: two values are equal if their data content is
  structurally equivalent. Recursive field-by-field comparison for compound
  types. This is the default comparison for data.
- `==` — **Semantic equality**: user-defined equality that may override
  structural comparison. Custom types can define `==` to mean domain-specific
  equivalence (e.g., two `Person` objects with the same ID are equal even if
  other fields differ).
- `is` — **Identity equality**: two references point to the same object in
  memory. Only meaningful for mutable or reference-counted data.

```orthon
"hello" === "hello"   # true — same characters
"hello" is "hello"    # false — may be different allocations (implementation-dependent)
a = (1, 2, 3)
b = (1, 2, 3)
a === b               # true — structural equality
a is b                # false — different tuple objects
```

## Default Strategy

By default, all data uses **structural value equality** (`===`). The
compiler generates a field-by-field comparison for compound types.
Custom types may override `==` for semantic equality. Identity
comparison (`is`) is always available as a built-in operation.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Pointer equality | Compare by memory address only | Performance-critical identity checks; token/ID objects |
| Custom equality | User-defined `==` that bypasses structural comparison | Domain objects where business-logic equivalence differs from structural equality |
| Approximate equality | Floating-point comparison with epsilon | Numeric computation where exact equality is undesirable |

## Open Questions

1. How does equality interact with floating-point NaN? IEEE 754 specifies
   `NaN != NaN`, which violates transitivity. Should Orthon provide an
   `===` that respects IEEE 754, or should it use a total-order comparison?
2. Should mutable objects support value equality, or should they only
   support identity equality (`is`)? Immutable-by-default design suggests
   value equality for all immutable data, identity only for mutable.
3. How does equality work for function types and closures? Two functions
   with the same body are semantically equal, but detecting this is
   undecidable.

## Decision History

- **Equality operators (`===`, `==`, `is`):** Three distinct operators with
  explicit semantics was chosen over a single `==` with implicit semantics
  (Python/Java model). Rationale: explicitness principle requires that the
  programmer's intent be syntactically visible. Alternatives considered:
  single `==` with type-dependent semantics (rejected: violates Consistency
  principle), Python model with `__eq__` overloading (rejected: violates
  Explicitness).
- **Deferred to Phase 3:** Full design of `==` overloading mechanism,
  comparison trait/generics integration, and NaN semantics.
