# Generics

> **✅ ACCEPTED — [EDR-024](../how/decision_records/architecture/EDR-024-generics.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`TRAITS.md`](TRAITS.md),
> [`TYPE_INFERENCE.md`](TYPE_INFERENCE.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Generics, Trait Bound, Monomorphisation,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How do you write code that works with multiple types without sacrificing type safety, performance, or readability?

Every practical language needs parametric polymorphism: a single function definition that works uniformly across many concrete types. The alternatives are:

- **Duplicated implementations** — Write the same logic for each type. Violates DRY, error-prone.
- **Runtime type erasure** — Accept `Object` / `void*` and cast at runtime. Type-unsafe, performance overhead.
- **Code generation macros** — Write once, expand per type. No type checking at definition time.

The core problem: **behavioural constraints on type parameters** must be expressible and checkable. The language needs a mechanism to say "this function works for any type `T` that supports operations X, Y, Z."

## Principles

1. **Trait-bounded constraints** — Generic parameters are constrained by traits that specify required operations. No duck-typed generics.
2. **Static dispatch by default** — Monomorphisation is the default; dynamic dispatch (`dyn Trait`) is opt-in.
3. **No type erasure** — Generic type information is preserved through compilation. No erasure, no runtime type casts.
4. **Invariant by default** — Generic type parameters are invariant unless variance is declared via trait method signatures.
5. **`where` clauses for complex bounds** — Multiple trait bounds use `where T: TraitA + TraitB`. Simple single bounds may use inline shorthand.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dispatch Policy | Determines static (monomorphisation) vs. dynamic (boxing/vtable) dispatch |
| Variance Policy | Controls default invariance and declared variance for type parameters |
| Monomorphisation Policy | Governs per-instantiation vs. compilation-unit-level code generation |
| Trait Resolution Policy | Controls bound resolution and associated type substitution |

## Model (What)

### Generic Functions

Trait-bounded type parameters constrain the types a generic function accepts.

```orthon
// Single bound — inline syntax
fn first[T: Iterator](items: T) -> Option<T::Item>
    for item in items
        return Some(item)
    return None

// Multiple bounds — where clause
fn process[T](value: T) where T: Hash + Eq
    let hash = value.hash()
    let equal = value == other
```

### Generic Types

Types can also be parameterised:

```orthon
// Generic struct
type Pair[T, U]
    first: T
    second: U

// Generic with trait bounds
type HashMap[K: Hash, V]
    # implementation
```

### Static Dispatch (Default)

By default, each generic instantiation produces separate compiled code through monomorphisation:

```orthon
let a = identity(42)     # identity[Int] — monomorphised
let b = identity("hi")   # identity[String] — separate monomorphised copy
```

### Dynamic Dispatch (Opt-In)

Dynamic dispatch uses `dyn Trait` to erase the concrete type:

```orthon
fn process(items: [dyn Processor])
    for item in items
        item.process()   # vtable dispatch
```

### Variance Rules

- Type parameters are **invariant by default** — `List[Cat]` is not a subtype of `List[Animal]`.
- **Covariance** is declared via return-position associated types in traits.
- **Contravariance** is declared via argument-position parameters in traits.

```orthon
trait Producer
    type Output          # covariant — appears in return position

trait Consumer[T]
    fn accept(self, item: T)   # contravariant — appears in argument position
```

### Associated Type Resolution

Associated types are resolved during monomorphisation. The compiler substitutes the concrete type's associated type for the trait's declaration.

```orthon
trait Collection
    type Item

impl Collection for List[Int]
    type Item = Int

// When monomorphising with List[Int], Collection::Item becomes Int
```

### Cross-Reference: COMPILE_TIME_EXECUTION (Plan 04-03)

Generics interact with compile-time execution in two ways:

1. **Comptime generic evaluation** — If `comptime` is adopted, generic type computations may execute at compile time, enabling type-level programming with generic parameters.
2. **Comptime trait resolution** — Trait bounds on generic parameters may be resolved at compile time for `comptime` contexts.

The precise interaction is specified in Phase 04-03 (COMPILE_TIME_EXECUTION).

## Default Strategy

Static dispatch via monomorphisation with trait bounds. Invariant by default. `where` clauses for complex bounds, inline `[T: Trait]` for simple bounds. Associated types resolved during monomorphisation. No type erasure.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Dynamic dispatch only | Generics boxed by default, no monomorphisation — simpler but slower. Trades performance for binary size and compile time |
| Hybrid dispatch | Compiler chooses static or dynamic dispatch based on instantiation count. Performance heuristic, not semantic change |
| Compilation-unit-level monomorphisation | Monomorphises per compilation unit rather than per instantiation — reduces duplicate code at the cost of inter-procedural optimisation |
| Dictionary passing (typeclass-based) | Passes method dictionaries at runtime instead of monomorphising — used in Haskell. Less code bloat but no inlining across trait boundaries |

## Open Questions

1. Should higher-kinded types (HKT) be supported in v0.2+? (Not in v0.1.)
2. Should associated type defaults be allowed? (E.g., `type Item = T` as fallback.)
3. How does monomorphisation interact with the Execution Program model — can monomorphised code be cached across builds?
4. Should negative trait bounds (`where T: !Hash`) be supported in v0.2+?
5. How does generics interact with the Metadata Protocol (`@`) — should `T@name` expose the concrete type name?

## Decision History

- **Trait-bounded generics** adopted over duck-typed templates (C++) and type erasure (Java). Rationale: Explicit constraints provide type safety and clear error messages while preserving all performance via monomorphisation.
- **Static dispatch by default** adopted. Rationale: Performance, inlining capability, and consistency with the trait system.
- **Invariant by default** adopted. Rationale: Safe, conservative. Covariance/contravariance are declared via trait method signatures.
- **No type erasure** adopted. Rationale: Preserves type information for LLM tooling, schema provision, and reflection.
- **Accepted via EDR-024** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/TRAITS.md`
- [ ] `what/concepts/TYPE_INFERENCE.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
