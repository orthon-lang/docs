# Runtime-Constrained Types

> **✅ ACCEPTED — EDR-080.**
> Language Pattern (Level 2) — syntactic sugar over `struct` + contract on constructor.
>
> **Status:** Accepted 2026-07-30.
> **See also:** [`CONTRACTS.md`](../../how/concepts/research/important/CONTRACTS.md),
> [`STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`](../../how/concepts/research/important/STRUCT_AS_NOMINAL_PRODUCT_TYPE.md),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`CORE_CONCEPTS.md`](../CORE_CONCEPTS.md),
> [`GLOSSARY.md`](../GLOSSARY.md)

---

## Issue (Why)

A function signature describes what types flow in and out, but not what subset of values they contain. The programmer must encode domain constraints either in documentation (ignored by compiler) or in contracts on every consuming function (duplicated, caller-dependent).

For LLMs generating code, this gap is especially costly: an LLM given `fn set_age(a: Int)` has no information that valid ages are 0–150 and may generate nonsensical calls. A type that carries its constraint — `type Age = Int(0..150)` — eliminates this ambiguity at the schema level.

Three concrete problems:

1. **No type-level constraint** — `Int` allows any integer. `Age`, `PositiveInt`, `Percentage` must be documented, not enforced.
2. **Boilerplate repetition** — Every domain primitive requires `struct` + `new` + `requires` + `ensures` (≈6 lines). The pattern is mechanical.
3. **LLM schema gap** — Without machine-readable constraints, LLMs rely on naming conventions and documentation, which are unreliable.

## Principles

1. **Composition over addition** — Constrained types decompose to existing primitives. No new semantics.
2. **Nominal identity** — `Age ≠ Int`. A constrained type is a distinct type, not a subtype.
3. **Immutability by default** — The backing value cannot be mutated after construction (consistent with SEMANTIC_MODEL.md § Mutation).
4. **Schema exposure** — Constraints are visible in machine-readable form (Schema Provider).
5. **Runtime enforcement** — Constraints are checked at construction time following Contract Enforcement Policy. Static analysis for literals is an optimisation, not a guarantee.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| **Contract Enforcement Policy** | Determines when constraint checks run (debug, release, elision) |
| **Allocation Policy** | Constrained types follow the same allocation rules as their desugared `struct` form |
| **Representation Policy** | The backing field's representation follows the base type's representation rules |

## Model (What)

### Syntax

```orthon
// Range constraint
type Age = Int(0..150)

// Named form
type Percentage = Int(min=0, max=100)

// Pattern constraint
type Email = String(matches: email_pattern)

// Length constraint
type NonEmptyString = String(min_length=1)

// Compound expression
type PositiveInt = Int(v > 0)
```

### Desugaring

```
type Age = Int(0..150)

    ↓ desugars to

struct Age
    value: Int

    new(v: Int) -> Age
        requires v >= 0 && v <= 150
        ensures result.value == v
```

### Type Identity

Constrained types are **nominally distinct** from their base type and from each other:

```orthon
type Age = Int(0..150)
type Score = Int(0..100)

let a: Age = Age(42)       // ✅ ok
let s: Score = Score(85)    // ✅ ok
let x: Age = Score(85)      // ❌ type error: Score ≠ Age
let y: Int = Age(42)        // ❌ type error: Age ≠ Int
```

### Construction

Construction follows the base type's literal syntax:

```orthon
let age = Age(25)           // ✅ runtime check passes
let bad = Age(200)          // ⚠️ compile-time warning (literal out of range)
                            //    runtime error if executed
let input = Age(read_int()) // ✅ runtime check on read_int() result
```

### Access

The backing value is accessed via the field name (consistent with struct field access):

```orthon
fn is_adult(age: Age) -> Bool
    return age.value >= 18
```

### Constraint Expressions

Constraint expressions are **pure** — same rules as contract expressions:

- No mutation of captured variables
- No I/O operations
- No non-deterministic functions
- May only reference the implicit value (`v`) and pure functions

### Constraint Forms Reference

| Form | Syntax | Equivalent Contract | Notes |
|------|--------|---------------------|-------|
| Range | `Int(0..150)` | `v >= 0 && v <= 150` | Closed range |
| Min | `Int(min=0)` | `v >= 0` | Half-open |
| Max | `Int(max=100)` | `v <= 100` | Half-open |
| Min+Max | `Int(min=0, max=100)` | `v >= 0 && v <= 100` | Closed range |
| Pattern | `String(matches: p)` | `matches(v, p)` | Regex/pattern |
| Min length | `String(min_length=1)` | `v.length >= 1` | String only |
| Max length | `String(max_length=50)` | `v.length <= 50` | String only |
| Compound | `Int(v > 0 && v % 2 == 0)` | `v > 0 && v % 2 == 0` | Any pure expression |

## Default Strategy

Constraint checks follow the Contract Enforcement Policy:

| Build mode | Constraint checking |
|------------|-------------------|
| Debug | Runtime check on every construction |
| Test | Runtime check on every construction |
| Release | Elided (optimised out) unless `--enable-contracts` specified |

Literal values are checked at compile time regardless of mode.

## Alternative Strategies

| Strategy | Description |
|----------|-------------|
| **Static-only** | All constraints verified at compile time via SMT solver. Requires dependent type system integration. Deferred to future milestone. |
| **Always-check** | Constraints checked even in release builds. Opt-in via `--enable-contracts` or Strategy-level policy override. |
| **Ghost constraints** | Constraints are documentation-only — no runtime checks. Suitable for gradual migration from untyped code. |

## Open Questions

1. Should constrained types support generic base types (e.g., `type Positive<T: Numeric> = T(v > 0)`)? Currently deferred — Phase 5 syntax scope.
2. Should compound constraint expressions use `v` or allow any parameter name? Current design uses `v` for consistency with contract `result` and `old`.
3. Should constraint expressions be referentially transparent within the Schema Provider (i.e., can the LLM see the actual expression)? Yes — the constraint AST should be serialized into the schema.

## Decision History

| Decision | Date | Rationale |
|----------|------|-----------|
| Accepted as Language Pattern (Level 2) | 2026-07-30 | Fully decomposable to struct + contract. No new primitives. |
| Immutable backing field | 2026-07-30 | Consistent with SEMANTIC_MODEL.md § Mutation. Prevents constraint bypass. |
| Nominal identity (no subtyping) | 2026-07-30 | Consistent with struct semantics. `Age ≠ Int` is the source of type safety. |

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/SYNTAX.md` — new syntactic form
- [x] `what/LIBRARY_BOUNDARY.md` — clarify constrained types are language, not library
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [x] `how/concepts/research/important/CONTRACTS.md` — note relationship
- [x] `how/concepts/research/important/STRUCT_AS_NOMINAL_PRODUCT_TYPE.md` — note relationship
- [x] `how/decision_records/INDEX.md`
- [x] `how/gates/DECISION_LOG.md`
- [x] `how/gates/DECISION_VALIDATION.md` — Decision Journal entry
