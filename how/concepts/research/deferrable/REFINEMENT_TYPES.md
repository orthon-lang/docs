# Refinement Types

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-08-04
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

Orthon's "Make Illegal States Unrepresentable" pattern
([`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md))
works well for *discrete* invalid states: wrong spellings (literal
types), missing variants (ADTs), null (Option). It does **not** cover
*value-range* constraints: "this integer is a valid TCP port (1..65535)",
"this float is non-negative", "this list is sorted".

**Orthon already addresses this in its pragmatic form.** Constrained
Types (EDR-080, [`CONSTRAINED_TYPES.md`](../../../what/concepts/CONSTRAINED_TYPES.md))
provide a nominal type with a type-level constraint predicate, checked at
every entry boundary:

```orthon
type Port = Int requires v in 1..65535
type Age = Int requires v >= 0 && v <= 150
```

The hypothesis of *this* document is **static refinement** — moving the
constraint check from runtime (EDR-080's enforcement model) to compile
time where the compiler can prove it. Under EDR-080, `Age(200)` is only
a *compile-time warning* plus a runtime error if executed. The open
question is: for a decidable subset of predicates (ranges, positivity,
non-empty), can the compiler reject the invalid literal at compile time
— turning the warning into an error?

```orthon
let bad = Age(200)   # EDR-080: ⚠️ warning + runtime error
                     # static refinement (hypothesis): ❌ compile error
```

## Prior Art

| Language | Mechanism | Enforcement | Strength |
|----------|-----------|-------------|----------|
| **Orthon (EDR-080)** | Constrained types (`requires` predicate) | Runtime at entry boundaries | Pragmatic; nominal, not static |
| **Liquid Haskell** | Refinement types via SMT | Static (SMT solver) | Full predicate refinement |
| **F\*** | Refinement types + effects | Static (Z3) | Full; used for verified crypto |
| **Scala 3** | Opaque types + smart constructors | Runtime at boundary | Manual, no auto-refinement |
| **Refined (Scala/Haskell lib)** | `Refined[Int, Positive]` | Runtime predicate | Library-level |
| **Rust** | No native refinement; `NonZeroU32`, `typenum` | Type-level wrappers | Narrow set |

The lesson: full dependent-style refinement (Liquid Haskell, F\*)
requires SMT solvers and heavy machinery — a poor fit for Orthon's
minimal core. The pragmatic forms are **library/type-level wrappers**
(scalarized, like `NonZeroU32`) or **opaque types + smart constructors**
(Scala 3, and Orthon's own `Option`-returning constructors).

## Core Inclusion Filter

Before formal validation, run through the Core Inclusion Filter
procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 1–2 (Primitive / Derived)

Library test:   Can refinement types be a StdLib construct?

                A library can provide `Refined[A, P]` as a wrapper with
                runtime predicate checks (like the `refined` Scala lib).
                But it cannot make `where _ in 1..65535` a part of the
                type itself, nor statically reject invalid literals at
                the type level. `type Port = Int where _ in 1..65535`
                requires compiler support in type checking.

                → BORDERLINE — library wrappers approximate it.

Pattern test:   Can refinement be expressed as a composition of existing
                Primitives and Data Model constructs?

                Smart constructors (`parsePort(s) -> Option<Port>`) +
                opaque types approximate refinement for *construction*
                paths. But they cannot statically reject `let p: Port =
                99999` at a literal — the compiler has no notion of the
                refinement predicate on `Int`.

                → PARTIAL — composition covers the common case.

Primitive test: Is refinement an atomic operation?

                A `where` predicate on a base type is a new type-forming
                operation, not decomposable into existing primitives.

                → PASS for the type-forming semantics; but the value is
                incremental over smart constructors.

Conclusion: BORDERLINE — defer to v0.2; smart constructors + contracts
cover v0.1 needs.
```

## Model (What)

### Predicate-Refined Type (full form, deferred)

```orthon
type Port = Int where _ in 1..65535
type NonEmpty = String where len(_) > 0
```

- Literals violating the predicate are rejected at compile time.
- Conversion from a wider type requires a checked function returning
  `Option<Port>` or `Result<Port, _>`.
- All operations on the refined type preserve the predicate.

### Smart Constructor (v0.1 form, recommended)

```orthon
type Port   # opaque — fields private, no direct literal coercion
    from Int where _ in 1..65535  # checked constructor
```

```orthon
fun parsePort(raw: Int) -> Option<Port>
    if raw in 1..65535 then Some(Port(raw)) else None
```

This is the "Parse, don't validate" pattern
([`notes/parse-dont-validate-idiom.md`](../../../notes/parse-dont-validate-idiom.md))
applied to value ranges: the invariant lives in the constructor, and the
`Port` type cannot represent an out-of-range value.

## Default Strategy (v0.1)

Adopt **Constrained Types (EDR-080)** as the accepted pragmatic form —
no new language work needed for v0.1. This document's *static refinement*
hypothesis is deferred: investigate a decidable predicate subset in
v0.2 pending Type System design
([`TYPE_SYSTEM.md`](../../architecture/TYPE_SYSTEM.md)).

## Alternative Strategies

| Strategy | Description | Assessment |
|----------|-------------|-----------|
| **Full predicate refinement (SMT)** | `Port = Int where _ in 1..65535` proven by a solver | Deferred v0.2+; solver dependency contradicts minimal core |
| **Decidable-subset static refinement** | Compile-time rejection of literals for ranges/positivity/non-empty | The actual hypothesis of this document; no solver needed |
| **Constrained types (EDR-080)** | Runtime-enforced `requires` predicate at boundaries | **Accepted** — the v0.1 form |
| **Smart constructors only** | Opaque type + checked constructor | Partially subsumed by EDR-080's `Callable` construction |
| **Contracts only** | `requires p in 1..65535` on every use | Weaker; duplicated on every consuming function |

## Open Questions

1. For which predicate subset can static refinement reject invalid
   literals at compile time without an SMT solver? (ranges, positivity,
   non-empty strings are candidates)
2. How does static refinement interact with EDR-080's warning-then-
runtime-error model? Is the warning upgradable to an error per module?
3. Does a statically refined type remain nominally distinct from its
   base type (as EDR-080 requires)?
4. What is the migration path from EDR-080 runtime constraints to
   static refinement? Is it purely an Implementation Strategy choice
   (Semantics Before Optimization)?

## Decision History

- **2026-08-04:** Initial research. Positioned against the existing
   Constrained Types (EDR-080) — the hypothesis is specifically the
   *static* (compile-time) extension, not the constrained-type concept
   itself, which is already accepted.

## See also

- [`CONSTRAINED_TYPES.md`](../../../what/concepts/CONSTRAINED_TYPES.md) — the accepted pragmatic form (EDR-080)
- [`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md) — the pattern this extends to value ranges
- [`CONTRACTS.md`](../../../what/concepts/CONTRACTS.md) — Tier-2 invariants, the current mechanism for ranges
- [`CORRECTNESS_BY_CONSTRUCTION.md`](../important/CORRECTNESS_BY_CONSTRUCTION.md) — Tier 3 → Tier 1 migration strengthens CbC
- [`notes/parse-dont-validate-idiom.md`](../../../notes/parse-dont-validate-idiom.md) — the idiom constrained types implement
- [`STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`](../important/STRUCT_AS_NOMINAL_PRODUCT_TYPE.md) — nominal type identity (EDR-080 requirement)
