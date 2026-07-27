# Equality

> **✅ ACCEPTED — [EDR-017](../how/decision_records/architecture/EDR-017-equality.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Identity,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Value Equality, Semantic Equality, Identity Equality

---

## Issue (Why)

Equality is one of the most subtle and error-prone concepts in programming languages. The core problem: **reference equality and value equality are fundamentally different operations**, yet most languages conflate them or force the programmer to remember which operator does which.

In Python, `==` compares value equality for most built-in types but reference equality for custom objects (unless `__eq__` is defined). In Java, `==` is reference equality for objects and value equality for primitives — a distinction that causes bugs daily. In JavaScript, `==` performs type coercion while `===` does not, creating a three-way confusion between reference, value, and coerced equality.

Orthon's solution: **three distinct operators with three distinct semantics**, chosen by the programmer at each comparison site.

## Principles

1. **Explicit Equality** — Different kinds of equality (value, semantic, identity) use different syntax. The programmer never guesses which comparison is being made.
2. **Structural by Default** — Value comparison for compound data is structural by default (field-by-field, recursively), not identity-based. This aligns with the Data-first philosophy: data is compared by its structure, not its memory location.
3. **Consistency** — The same equality operator has the same semantics for all types. No special cases for primitives vs. objects vs. custom types.
4. **Transitivity** — Equality must be transitive: if `a == b` and `b == c`, then `a == c`. Non-transitive equality (e.g., floating-point NaN) must be explicitly documented and have a distinct operator.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Comparison Policy | Defines the default comparison algorithm (structural vs. identity) and custom comparison hooks |
| Allocation Policy | Affects whether identity comparison (`is`) is meaningful — arena-allocated values may have unstable addresses |
| Representation Policy | Different representations (Value, Reference, Tuple) may define equality differently |

## Model (What)

Orthon defines three distinct equality operators, each with a clear semantic:

- `===` — **Value equality** (structural). Two values are equal if their data content is structurally equivalent. Recursive field-by-field comparison for compound types. This is the default comparison for all data.
- `==` — **Semantic equality** (user-defined). Custom types can define `==` to mean domain-specific equivalence (e.g., two `Person` objects with the same ID are equal even if other fields differ). Falls back to `===` if not overridden.
- `is` — **Identity equality**. Two references point to the same object in memory. Only meaningful for mutable or reference-counted data. Always returns `false` for value types without identity.

```orthon
"hello" === "hello"      # true — same characters
"hello" == "hello"       # true — same as === by default (no override)
"hello" is "hello"       # false — may be different allocations (implementation-dependent)

a = (1, 2, 3)
b = (1, 2, 3)
a === b                  # true — structural equality
a is b                   # false — different tuple objects

# User-defined semantic equality
impl Eq for Person
    fn ==(self, other: Person) -> Bool
        self.id === other.id

alice = Person(id=42, name="Alice")
bob = Person(id=42, name="Bob")
alice == bob             # true — same ID despite different names
alice === bob            # false — different structural content
alice is bob             # false — different allocations
```

### Named Function Equivalents

Per the Named Before Symbolic principle, each operator has an equivalent named function:

```orthon
a === b          # operator form
eq(a, b)         # named function form

a == b           # operator form
equal(a, b)      # named function form (delegates to trait)

a is b           # operator form
same(a, b)       # named function form
```

### Transitivity Invariant

Orthon enforces the **Transitivity Invariant**: for any types `T` where `==` or `===` is defined, if `a == b` and `b == c` then `a == c` must hold. The compiler may insert runtime checks in debug mode to detect violations.

`is` (identity equality) is transitive by construction: if two references point to the same object, they are transitively identical.

### NaN

Floating-point NaN equality is **deferred** to the Standard Library. IEEE 754 specifies `NaN != NaN`, but this violates the Transitivity Invariant. Orthon requires an explicit comparison method (e.g., `Float.isNaN()`) rather than an operator. A future Standard Library addition may provide `==~` (approximately equal) for numeric computation.

### Interaction with Mutability

Mutable objects support identity equality (`is`) only. Value equality (`===`) on a mutable type compares the current structural content. The compiler may warn when `===` is used on a mutable type whose content changes between the comparison and the use of the result.

## Default Strategy

All data uses **structural value equality** (`===`) by default. The compiler generates a field-by-field comparison for compound types (`pack`/`unpack` structure). Custom types may override `==` for semantic equality by implementing the `Eq` trait. Identity comparison (`is`) is always available as a built-in operation.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Pointer equality | Compare by memory address only (`is`) | Performance-critical identity checks; token/ID objects |
| Custom equality | User-defined `==` that bypasses structural comparison | Domain objects where business-logic equivalence differs from structural equality |
| Approximate equality | Floating-point comparison with epsilon | Numeric computation where exact equality is undesirable |

## Open Questions

1. **NaN semantics:** Should Orthon provide a `===` that respects IEEE 754 (violating transitivity), or require explicit `Float.isNaN()` for NaN checks?
2. **Function equality:** Two functions with the same body are semantically equal, but detecting this is undecidable. Should function comparison be limited to identity (`is`)?
3. **Collection equality ordering:** Should `Set` equality use `===` element-wise regardless of iteration order?

## Decision History

- **Three distinct operators (`===`, `==`, `is`)** chosen over a single `==` with implicit semantics (Python/Java model). Rationale: Explicitness principle requires the programmer's intent to be syntactically visible.
- **Structural by default** chosen over reference-by-default. Rationale: Data-first philosophy; most comparisons are about data content, not memory addresses.
- **Transitivity Invariant** adopted as a hard rule. Rationale: Prevents the NaN paradox (IEEE 754 `NaN != NaN`) from leaking into the language core.
- **NaN deferred** to Standard Library. Rationale: The Transitivity Invariant is more important than IEEE 754 compatibility in the core language.
- **Accepted via EDR-017** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/SEMANTIC_MODEL.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
