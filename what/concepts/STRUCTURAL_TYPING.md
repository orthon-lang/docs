# Structural Typing

> **✅ ACCEPTED — [EDR-044](../how/decision_records/architecture/EDR-044-structural-typing.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`TRAITS.md`](TRAITS.md), [`GLOSSARY.md`](../GLOSSARY.md) § Structural Typing,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md),
> [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md)

---

## Issue (Why)

Two approaches to polymorphism dominate: **nominal typing** (explicit `impl Trait for Type`, EDR-019) and **structural typing** (a type satisfies a trait if it has the required methods, regardless of explicit declaration).

Nominal typing tends toward premature abstraction and ceremony — interfaces are designed before they have multiple implementations, and adding a new implementation requires writing `impl` blocks. Structural typing offers "if it quacks like a duck, it is a duck" — no prior arrangement needed.

The core problem: Orthon's TRAITS (EDR-019) are nominally typed. Should Orthon also support structural trait satisfaction, and if so, should it be the default or opt-in?

## Principles

1. **Nominal by default** — Most traits require explicit `impl Trait for Type`. Structural satisfaction is opt-in via the `structural` keyword on the trait declaration.
2. **`structural` keyword makes it explicit** — The choice between nominal and structural mode is visible in the trait declaration.
3. **Explicit `impl` overrides structural matching** — An explicit `impl` block always takes priority over structural satisfaction for a given trait-type pair.
4. **Static dispatch by default** — Structural resolution happens at compile time with static dispatch. Dynamic dispatch (`dyn Trait`) is opt-in.
5. **`@derive` generates explicit `impl` blocks** — Derived implementations take priority over structural matching.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Compatibility Policy | Determines whether trait satisfaction is nominal (explicit `impl` required) or structural (automatic by method shape) |
| Dispatch Policy | Controls static (monomorphisation) vs dynamic (vtable) dispatch for structural traits |
| Trait Resolution Policy | Governs how the compiler resolves which trait a type satisfies — explicit `impl` first, then structural matching |
| Coherence Policy | Prevents conflicting structural/nominal matches — explicit `impl` always wins |

## Model (What)

### Structural trait declaration

A trait may be declared `structural` to enable automatic satisfaction by method shape:

```orthon
# Nominal — explicit impl required (default)
trait Serializable
    fn serialize(self) -> String

# Structural — implicit satisfaction by method signature
structural trait Show
    fn show(self) -> String
```

### Structural satisfaction

Any type with methods matching a structural trait's signature satisfies the trait automatically:

```orthon
structural trait Show
    fn show(self) -> String

type Point(x: Int, y: Int)

# Point satisfies Show structurally if it has a `show` method
Point has show(self) -> String

# No explicit impl needed — structural matching resolves at call site
fun print[T: Show](value: T)
    print(value.show())
```

### Explicit impl override

An explicit `impl` block takes priority over structural matching:

```orthon
impl Show for Point:
    fn show(self) -> String
        "Custom: ($self.x, $self.y)"
```

### Derive generates explicit impls

```orthon
@derive(Show, Eq)
type User(name: String, age: Int)

# Generated impl blocks take priority over structural matching
```

### Conflict resolution

If a type structurally matches two traits with the same method name but incompatible signatures, the compiler reports an ambiguity error:

```orthon
structural trait A:
    fn process(self) -> Int

structural trait B:
    fn process(self) -> String

type Value(x: Int)
    fn process(self) -> Int = self.x
    fn process(self) -> String = "value: $(self.x)"

# Error: Value.process matches both A and B — provide explicit impl
impl A for Value:
    fn process(self) -> Int = self.x
```

## Default Strategy

The compiler resolves trait satisfaction at each call site: first check explicit `impl` blocks, then check structural matching for `structural` traits. Static dispatch is used for both modes. Dynamic dispatch (`dyn Trait`) is available as opt-in.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Fully nominal (Rust) | No structural matching — all traits require explicit `impl`. Rejected for marker traits where `impl` is pure ceremony. |
| Fully structural (Go) | All traits satisfied structurally. Rejected — violates Explicitness principle for semantically meaningful traits. |
| Per-trait opt-in via `structural` keyword | **This decision.** Trait author chooses the mode at declaration time. Balances explicitness with ergonomics. |
| Per-method structural matching | Too fine-grained — adds complexity without proportional benefit. |

## Open Questions

1. Should structural matching support associated type inference? (If a structural trait has an associated type, can the compiler infer it from the type's methods?)
2. Should structural matching work across module boundaries, and if so, what coherence rules apply?
3. Should there be a `structural` block syntax for marking multiple methods as a structural implicit group?

## Decision History

- **EDR-044 (2026-07-27):** Structural typing accepted as Language feature. Opt-in via `structural` keyword on trait declaration. Nominal is default. Explicit `impl` overrides structural.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/concepts/TRAITS.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
