# Struct as Nominal Product Type

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> Explores the gap between `alias` (type synonym, no new type) and ADT (sum type)
> — a **named product type** with nominal identity.
>
> **Last updated:** 2026-07-29
>
> **See also:**
> - [`STRUCT_AS_VALUE_TYPE.md`](../essential/STRUCT_AS_VALUE_TYPE.md) — Value semantics of structs (essential tier, DRAFT)
> - [`CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`](../reject/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md) — REJECTED (EDR-078)
> - [`ALGEBRAIC_DATA_TYPES.md`](ALGEBRAIC_DATA_TYPES.md) — ADT model (product + sum via `type ... = A | B`)
> - [`../essential/FOUNDATIONAL_ABSTRACTIONS.md`](../essential/FOUNDATIONAL_ABSTRACTIONS.md) — Data / Data Modifiers hypothesis
> - [`../../what/PRIMITIVE_BLOCKS.md`](../../what/PRIMITIVE_BLOCKS.md) — `pack`/`unpack` as composition primitive
> - [`../../what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) — Six semantic dimensions
> - [`../../what/GLOSSARY.md`](../../what/GLOSSARY.md) — Terminology reference

---

## Issue (Why)

Orthon currently has these mechanisms for type-level abstraction:

| Mechanism | Creates new type? | Fields | Use case |
|---|---|---|---|
| **`alias`** (type synonym) | ❌ — same type, new name | N/A | Documentation, shorthand |
| **`pack`/`unpack`** | ❌ — structural composition | ✅ positional | Ad-hoc tuples, destructuring |
| **ADT** (`type ... = A \| B`) | ✅ — distinct sum type | ✅ per variant | Choice between variants |

**Gap:** There is no way to say *"this is a new, self-standing type with named fields, distinct from all other types, without being a sum of variants."*

This produces three concrete pain points:

1. **No type safety for domain primitives** — `Email = String` via alias means a function expecting `Email` silently accepts `Name`, `Phone`, or any other `String`. The compiler does not distinguish them.

2. **No home for "just data with named fields"** — DTOs, API request/response schemas, configuration records, and database row types have no natural construct. ADT is overkill (no variants); `pack` gives no type identity; `alias` gives no type safety.

3. **Domain-Driven Design impedance** — Value Objects (`OrderId`, `CustomerId`, `Money`) require nominal distinctness in the type system, not just in documentation. Without it, the compiler cannot enforce domain invariants.

---

## Hypothesis

Orthon introduces `struct` as a **named product type with nominal identity**: a composite of named fields that constitutes a new, distinct type unrelated to any other type by structure alone.

### Core properties

| Property | Value |
|---|---|
| **Type identity** | Nominal — `struct Point` and `struct Vector` with identical fields are distinct types |
| **Value semantics** | Assignment copies structurally (consistent with SEMANTIC_MODEL.md § Identity) |
| **Field mutability** | Immutable by default; mutation via `var binding` (consistent with SEMANTIC_MODEL.md § Mutation) |
| **Equality** | Structural by default (`===`), overridable via `impl Eq` (`==`) |
| **Methods** | `fun` (pure) always allowed; `proc` (mutating) requires explicit `mut self` |

### Relationship to `alias`

```orthon
alias UserId = Int         # UserId ≡ Int — fully interchangeable
                           # let id: UserId = 42 — valid, no guard

struct OrderId              # OrderId ≠ Int — distinct type
    value: Int
                            # let id = OrderId{42} — explicit construction
                            # takes(42) — COMPILE ERROR: Int ≠ OrderId
```

`alias` is for *documentation intent only*; `struct` is for *compiler-enforced type safety*.

### Relationship to ADT

ADT (`type Shape = Circle(radius) | Rectangle(w, h)`) is a **sum of products** — each variant is an anonymous product within a choice. `struct` is a **standalone product without a choice** — a named type that is not a variant of anything.

```orthon
type Shape = Circle(radius: Float) | Rectangle(w: Float, h: Float)
#           ^^^ each variant is an anonymous product
#               cannot be used outside the sum

struct Point
    x: Float
    y: Float
#   ^^^ independent named product, not a variant
```

They are orthogonal and complementary: ADT for choices, `struct` for standalone composites.

### Relationship to `pack`/`unpack`

`struct` is a **named `pack`** — a derived construct over the existing `pack`/`unpack` primitive (per PRIMITIVE_BLOCKS.md § 3.1.3). The compiler desugars `struct Point{x, y}` to an internal `pack` representation, adds nominal type identity, and generates field accessors as syntactic sugar over `unpack` + `attribute access`.

```
struct Point{x: Int, y: Int}
    → internal: named pack with nominal tag
    → point.x → (unpack point).x
```

No new primitive is required — `struct` is a **derived feature** over `pack` + `identifier` + `attribute access`.

---

## Semantic Model Impact

### Identity (new sub-dimension: Type Identity)

The Semantic Model currently defines:
- **Binding identity** — do two names share the same storage?
- **Value identity** — are two values the same entity across time? (only for shared/reference types)

**Type identity** is a third, distinct notion: *are two values of the same declared type?* Two `struct` values with identical fields but different type names are **not** the same type, even when they have the same structure and the same value equality.

This is a narrowing of the Semantic Model's Identity dimension: the existing rule "identity is not universal" extends to "type identity is not structural — it is nominal." The first two semantic invariants are unaffected:

1. ✅ Every value has exactly one owner — unchanged.
2. ✅ Mutation requires exclusive access — unchanged.
3. ✅ Every value has a well-defined lifetime — unchanged.
4. ✅ All control flow produces a value — unchanged.
5. ✅ Visibility is compile-time guarantee — unchanged.
6. ✅ Ownership transfer is semantically explicit — unchanged.

**New cross-cutting invariant:** *Type identity is nominal, declared at the definition site, and independent of field structure.* This is a compiler-enforced rule, not a runtime property.

### Mutation

No change. `struct` fields follow the existing `val`/`var` model:

```orthon
let p = Point{1, 2}
p.x = 3        # COMPILE ERROR: immutable binding

var q = Point{1, 2}
q.x = 3        # OK
```

### Evaluation

No change. `struct` construction is an expression: `Point{1, 2}`.

### Visibility

No change. Fields have the same three levels (`priv`, default module, `pub`).

A `pub struct` makes the type visible but does not automatically expose its fields:

```orthon
pub struct Counter
    priv count: Int = 0    # type is public, field is not
    pub fun peek() -> Int
        self.count
```

### Ownership / Lifetime

No change. `struct` is a value type: ownership transfers on move, copies on assignment, lifetime is scoped.

---

## Building Blocks Impact

**No new primitives.** `struct` is fully decomposable onto the existing primitive set:

| Primitive | Role |
|---|---|
| `pack` | Constructor — combines fields into a composite |
| `unpack` | Destructuring — recovers fields from a composite |
| `identifier` | Type name, field names, method names |
| `attribute access` | Field read: `point.x` |
| `scope` | Body of struct declaration |
| `function` | Methods attached to the struct |
| `assignment` | Binding struct instances to names |

**The only non-primitive addition** is a **compiler-level rule**: nominal type identity must be checked at every assignment, parameter passing, and return — `struct Point` values cannot be used where `struct Vector` is expected, even if their fields match.

This mirrors the compiler cost already paid for ADT variant checking (variant tags, exhaustiveness) — it is a type-system extension, not a runtime change.

---

## Level Decision

| Level | Decision | Rationale |
|---|---|---|
| **Core Language** | ✅ | Nominal type identity is a type-system rule, not a library concern. The compiler must enforce it. |
| **Standard Library** | ❌ | StdLib cannot introduce new compile-time type identity rules. |
| **Framework** | ❌ | Even less suitable — framework-level struct would be syntactic sugar only, without compiler enforcement. |

**Classification:** Language (D-03 per DECISION_PIPELINE.md) — derived feature over core primitives, requiring compiler-level name-based type checking.

---

## Trade-offs

### Advantages

| Benefit | Explanation |
|---|---|
| **Type safety** | Catches errors that structural typing misses: `send_email(name)` where `name: Name` but `to: Email`. |
| **Domain modelling** | Direct DDD Value Object support — `OrderId`, `CustomerId`, `Money` as distinct, non-interchangeable types. |
| **Intuition** | `struct` is the most intuitive way to declare "I have data with named fields" — lower LLM cognitive load than choosing between `alias`, `pack`, and ADT. |
| **Non-breaking** | Does not change existing `pack`/ADT/alias semantics — adds an option, does not remove any. |
| **Primitive decomposable** | No new primitives — only a new compiler rule over existing ones. |

### Disadvantages

| Risk | Mitigation |
|---|---|
| **Conflicts with EDR-078** (rejected "class/structure as primary composition") | EDR-078 rejected *privileged* composition that crowds out orthogonal primitives. `struct` as *one of several* coexisting mechanisms (alongside ADT, `pack`, traits) does not violate this — but requires an explicit EDR to draw the boundary. |
| **Adds syntactic surface area** | Another keyword, another declaration form. Mitigation: `struct` is exactly as complex as ADT's single-variant case — no new parser complexity beyond what `type ... = A \| B` already requires. |
| **Overlaps with single-variant ADT** | `type Point = Point{x, y}` already works as a product type via ADT. However, this forces an unnecessary variant wrapper — the variant name duplicates the type name. `struct` eliminates this boilerplate and signals "no variants, ever." |
| **Nominal × structural tension** | Orthon uses structural equality for values (`==`) — does nominal typing for types contradict this? No: type checking is compile-time (nominal), value comparison is runtime (structural). They answer different questions at different stages. This is the Swift/Kotlin model. |

---

## Related Concepts and Alternatives

### Within Orthon

| Concept | Relationship |
|---|---|
| **`alias`** | Closest alternative for "just a wrapper." If only documentation intent matters, `alias` + trait bounds may suffice — but provides zero type safety. |
| **ADT** (`ALGEBRAIC_DATA_TYPES.md`) | `struct` = product without sum. ADT = sum of products. Orthogonal and complementary. |
| **`pack`/`unpack`** (PRIMITIVE_BLOCKS.md) | `struct` is named `pack` with nominal identity. All `struct` instances decompose to `pack` at IR level. |
| **TRAITS** | `struct` + `impl Trait for Struct` adds behaviour to data without inheritance — consistent with Orthon's composition model. |
| **`STRUCT_AS_VALUE_TYPE.md`** | Existing DRAFT in essential/ covering value semantics. This document extends it with nominal typing and the `alias`/ADT gap analysis. |
| **`CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`** (REJECTED, EDR-078) | Rejected because it proposed structure as the *primary/privileged* composition unit. `struct` as *a* composition unit (coexisting with ADT, `pack`, traits) is a narrower, compatible proposal — but needs an EDR distinguishing the two cases. |

### External Alternatives

| Approach | Example | Key Difference |
|---|---|---|
| **Pure structural typing** | TypeScript, Go interfaces | No nominal guard — structurally compatible types are interchangeable. Orthon uses structural *value* equality but could use nominal *type* identity — consistent split. |
| **Newtype pattern** | Haskell `newtype`, Rust `struct X(T)` | Zero-cost wrapper. Orthogonal to `struct` — `newtype` could be a special case of `struct` with exactly one field and no runtime overhead. |
| **Opaque type aliases** | Scala 3 `opaque type`, OCaml private types | Type alias that does not export its representation. A middle ground between `alias` and `struct`: same type at runtime, distinct at compile time. Worth considering as an alternative or complement. |
| **No struct — only ADT** | Status quo | Does not solve the DTO/Value Object problem. Single-variant ADT (`type Point = Point{x, y}`) works but adds boilerplate (variant name duplicates type name) and signals intent poorly ("this could have variants" vs. "this is a standalone product"). |

---

## Open Questions

1. **Should `newtype` be a separate concept or syntactic sugar over single-field `struct`?** Haskell's `newtype` guarantees zero-cost abstraction; a `struct` with one field might not. If Orthon wants the guarantee, it needs either a dedicated `newtype` keyword or a compiler optimization guarantee for single-field `struct`.

2. **Does `struct` need a `repr` attribute for C-compatible layout?** If Orthon targets FFI, yes — but this is an Implementation Strategy concern, not a semantic one. Deferrable.

3. **Can `struct` fields have default values?** E.g. `struct Config{host: String = "localhost", port: Int = 8080}`. This is ergonomics — important for DTOs but not semantically essential. Deferrable to syntax design.

4. **`pub struct` with `priv` fields — should the constructor enforce field visibility?** If a `pub struct` has `priv` fields, can external code construct it? If yes, the `priv` modifier on fields is purely read-side; if no, construction requires a builder or constructor function. This mirrors the Semantic Model's open question about `pub` on types.

---

## Decision History

| Date | Decision |
|---|---|
| 2026-07-29 | Initial hypothesis — identified the `alias` ↔ ADT gap and proposed `struct` as nominal product type filling it. |
