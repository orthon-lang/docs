# EDR-080: Runtime-Constrained Types — Language Pattern Over Struct + Contract

**Status:** Accepted

**Date:** 2026-07-30

**Category:** Architecture

**Scope:** Language Pattern (Level 2)

---

## Context

Orthon has two mechanisms for constraining values:

1. **Contracts** (`requires`/`ensures` on functions) — operational constraints checked at call boundaries. They answer *"what must be true when this function is called?"*
2. **Nominal types** (`struct`, `alias`) — type identity without validation. `struct Email { value: String }` creates a distinct type but requires manual constructor boilerplate for validation.

The gap: there is no ergonomic way to say *"this type can only contain values from a specific subset"* at the type declaration level. The programmer must write a `struct` + hand-written contract on `new` for every domain primitive — `Age`, `Email`, `NonEmptyString`, `PositiveInt` — repeating the same 6-line pattern each time.

For LLM generability, this gap is acute. An LLM consuming `fn register(age: Age)` benefits from knowing that `Age` is constrained to 0–150 at the schema level. Without it, the LLM must infer constraints from documentation or contract names, which is unreliable.

Three concrete pain points:

1. **LLM hallucination surface** — `fn set_age(a: Int)` — LLM may generate `set_age(999)`. `fn set_age(a: Age)` with schema-visible constraint eliminates this.
2. **Boilerplate repetition** — Every domain primitive requires `struct` + `new` + `requires` + `ensures`. The pattern is mechanically identical across types; only the constraint expression changes.
3. **No type-level documentation** — `Int` in a signature communicates no domain meaning. `Age` communicates intent but not bounds without external documentation.

## Decision

Orthon introduces **Runtime-Constrained Types** as a **Language Pattern (Level 2)** — syntactic sugar that desugars to `struct` + contract on constructor.

### Core form

```orthon
type Age = Int(0..150)
```

Desugars to:

```orthon
struct Age
    value: Int

    new(v: Int) -> Age
        requires v >= 0 && v <= 150
        ensures result.value == v
```

### General form

```orthon
type Name = BaseType(constraint_expr)
```

Where:
- `Name` — new nominal type (distinct from `BaseType`)
- `BaseType` — any primitive type (`Int`, `Float`, `String`, `Char`, etc.)
- `constraint_expr` — pure expression referencing the value (implicit `v`), supported forms:
  - Range: `0..150`, `0.0..1.0`
  - Predicate: `matches(email_pattern)`, `min_length(1)`, `max_length(50)`
  - Compound: `v >= 0 && v <= 150`

### Semantics

1. **Type identity** — nominal. `Age` and `Int` are distinct types. `fn(x: Int)` does not accept `Age`.
2. **Construction** — `Age(v)` checks the constraint at runtime (following Contract Enforcement Policy). Violation produces the same error mechanism as contract violation.
3. **Immutability** — the backing field is immutable by default (consistent with SEMANTIC_MODEL.md § Mutation). No mutation bypass.
4. **Static analysis** — literal values are checked at compile time: `let x = Age(200)` → compile-time error.
5. **Schema exposure** — the constraint is visible in the Schema Provider: `{base: Int, constraint: {range: [0, 150]}}`.

### Constraint forms

| Form | Example | Semantics |
|------|---------|-----------|
| Range | `Int(0..150)` | `v >= 0 && v <= 150` |
| Min/Max | `Int(min=0, max=150)` | Same semantics, named form |
| Pattern | `String(matches: email_pattern)` | Regex/pattern match |
| Length | `String(min_length=1)` | `v.length >= 1` |
| Compound | `Int(v > 0 && v < 1000)` | Arbitrary pure expression |

## Consequences

- **Positive:**
  - Eliminates 6 lines of boilerplate per domain primitive → 1 line
  - LLM Schema Provider exposes constraints directly — reduces hallucination
  - Zero new semantics — full decomposition to existing primitives
  - Compatible with all Implementation Strategies (follows Contract Enforcement Policy)
  - Reversible — can be deprecated without ecosystem breakage (users keep the desugared form)

- **Negative:**
  - Adds syntax surface area (`type X = Base(constr)`) despite being sugar
  - Constraint checked at runtime for non-literal values — not compile-time proven
  - Limited to single-field structs wrapping primitives (not general purpose refinement)

## Compliance

Verification that this decision is followed:

1. Every constrained type must desugar to `struct` + contract — no special compiler transformations
2. Constraint expressions must be pure (no side effects, no I/O)
3. Schema Provider must expose constraints in machine-readable format
4. Backing field must be immutable (no `var` on the wrapped value)

## Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| **Full refinement types (SMT)** | Compile-time SMT-proven refinement types. Rejected for v0.1: violates Minimal Core, requires Z3 or equivalent solver dependency. Viable for future milestone as enhanced enforcement. |
| **Pure library (`Bounded<T>`)** | Generic wrapper type without nominal identity — `Bounded<Int>` does not distinguish `Age` from `Score`. No special syntax. |
| **Contracts-only** | No change needed — use `struct + requires` manually. Rejected: boilerplate cost is real, LLM generability benefit is lost without schema exposure. |
| **Property wrappers** | Field-level validation without creating a new type. Rejected: does not solve the type identity problem (`Email` vs `String`). |

## Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Domain primitives with schema-visible constraints solve real LLM+programmer pain. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Flag: mutation guard must be explicitly specified — backing field immutable by default. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Full decomposition to `struct` + `pack` + contract. No new primitives. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Level 2 Language Pattern — does not violate layered architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Constraint enforcement follows Contract Enforcement Policy — strategy-agnostic. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Reversible, zero conceptual debt, clear evolution path to SMT refinement. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Schema-serializable, predictable, self-correctable via Static Analyser. |

**Gates not applied:** None.

**Detailed reasoning:** See `DECISION_LOG.md` entry [`CONSTRAINED_TYPES`](../../how/gates/DECISION_LOG.md#entry-constrained-types-edr-080) for per-gate reasoning trail.
