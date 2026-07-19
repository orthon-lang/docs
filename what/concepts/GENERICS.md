# Generics (Traits & Type Classes)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

## Issue (Why)

How do you write code that works with multiple types without sacrificing type safety, performance, or readability?

Generic programming addresses the tension between **reuse** and **safety**:
- `void*` / `Object` erases type information entirely — runtime casts (`ClassCastException`), no compiler help.
- Java/C# generics use type erasure — erased at runtime, no reification, limited performance.
- Templates (C++) use monomorphisation — full type safety, optimal performance, but code bloat and long compile times.

The real problem: **behavioural constraints on type parameters** must be expressible and checkable. The question is *how* to require that a type supports certain operations.

## Principles

1. **Behavioural constraints** — Generic parameters must be constrained by traits/interfaces/type classes that specify required operations.
2. **Static dispatch by default** — Monomorphisation is the default strategy; dynamic dispatch is opt-in (`dyn Trait` / `Box<dyn Trait>`).
3. **Coherence** — Trait implementations should not conflict at a distance (orphan rule, coherence constraints).
4. **No type erasure** — The type system must preserve generic type information.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dispatch Policy | Determines static (monomorphisation) vs dynamic (vtable) dispatch |
| Coherence Policy | Governs orphan rules, overlap rules, and implementation conflicts |
| Trait Resolution Policy | Controls how the compiler resolves trait bounds and associated types |

## Model (What)

Generics are expressed through **traits** (type classes) that define behavioural contracts. Generic functions and types constrain their parameters with trait bounds.

```
trait Hash:
    func hash(self) -> Int

func lookup<K: Hash, V>(key: K, table: Table<K, V>) -> Option<V>
```

Key features:
- **Trait bounds** on type parameters (`T: Hash`).
- **Associated types** and **associated constants** in traits.
- **Static dispatch** by monomorphisation (like C++ templates/Rust generics).
- **Dynamic dispatch** via trait objects (`dyn Trait`).
- **Blanket implementations** for collections of types.
- **Coherence rules** (orphan rule) to prevent conflicting implementations.

## Default Strategy

Static dispatch via monomorphisation with trait bounds. Dynamic dispatch requires explicit `dyn` keyword. Coherence follows the orphan rule (no implementations of external traits on external types in local crates).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Full monomorphisation | Every generic instantiation produces separate compiled code. |
| Dynamic dispatch only | Generics boxed by default, no monomorphisation — simpler but slower. |
| Hybrid dispatch | Compiler chooses static or dynamic based on number of instantiations. |
| Erased generics | Java-style — type parameters erased at runtime. |

## Open Questions

1. Should higher-kinded types (HKT) be supported? (Makes type system significantly more complex.)
2. Should associated type defaults be allowed?
3. How does monomorphisation interact with compilation units and incremental compilation?
4. Should negative trait bounds (`T: !Clone`) be allowed?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
