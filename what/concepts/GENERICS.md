# Generics

> **✅ ACCEPTED — [EDR-024](../how/decision_records/architecture/EDR-024-generics.md),
> syntax revised by [EDR-086](../how/decision_records/architecture/EDR-086-generics-syntax-revision.md).**
>
> **Status:** Accepted 2026-07-27; syntax revision 2026-08-07.
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
5. **`where` clauses for complex bounds** — Multiple trait bounds use `where T as TraitA + TraitB`. `+` combines bounds on one parameter (conjunction of requirements); `,` separates constraints on different parameters. Simple single bounds may use the inline shorthand `<Iterator as T>`.
6. **No method-level shadowing** — A method must not re-declare a type parameter of its enclosing type; a duplicate name is a compile error.

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
// Single bound — inline shorthand (bound-first: "T is (at least) an Iterator")
fun first<Iterator as T>(T items) -> Option<T::Item>
    for item in items
        return Some(item)
    return None

// Multiple bounds — where clause (T must satisfy Hash AND Eq)
fun process<T>(T value, T other) where T as Hash + Eq
    let hash = value.hash()      // justified by T as Hash
    let equal = value == other   // justified by T as Eq
    return hash
```

### Generic Types

Types can also be parameterised:

```orthon
// Generic type — two unbounded type parameters
type Pair<T, U>
    T first
    U second

// Generic with trait bounds — inline shorthand on K
type HashMap<K as Hash, V>
    # implementation
```

### Static Dispatch (Default)

By default, each generic instantiation produces separate compiled code through monomorphisation:

```orthon
let a = identity(42)     # identity<Int> — monomorphised
let b = identity("hi")   # identity<String> — separate monomorphised copy
```

### Dynamic Dispatch (Opt-In)

Dynamic dispatch uses `dyn Trait` to erase the concrete type:

```orthon
fun process([dyn Processor] items)
    for item in items
        item.process()   # vtable dispatch
```

### Variance Rules

Variance describes how subtyping propagates through generic type
parameters. For types `A <: B` (A is a subtype of B), a generic type
`F<T>` relates `F<A>` and `F<B>` in one of three ways:

| Variance | Relationship | Meaning |
|---|---|---|
| **Covariant** | `F<A> <: F<B>` | subtyping is preserved |
| **Contravariant** | `F<B> <: F<A>` | subtyping is reversed |
| **Invariant** | no relationship | parameter can be neither widened nor narrowed |

**Where subtyping comes from.** Orthon has no class inheritance and traits
do not create subtype relationships (see `TRAITS.md`). Subtyping arises
only from structural sources:

| Source | Example |
|---|---|
| Union types (EDR-045) | `Cat <: Cat | Animal` — a member is assignable to its union |
| Literal types (EDR-043) | `"GET" <: String` — a literal is a subtype of its base type |
| Widening | `Int` widens to `Float` where the context demands it |

Variance is therefore exercised on these real relationships, never on
invented class hierarchies.

**Position-based inference.** A type parameter's variance is computed
deterministically from the positions where it appears across the method
signatures of the type (EDR-024, Compliance #4):

- `T` in **output positions only** (return types, associated-type
  definitions) → **covariant** (`+`);
- `T` in **input positions only** (parameter types) → **contravariant** (`−`);
- `T` in **both** input and output positions → **invariant** (0).

The compiler walks every method signature and folds each position's
contribution; any mix of `+` and `−` for the same parameter yields
invariance.

```orthon
// COVARIANT — T appears only in return positions
type Producer<T>
    fun T produce()
    fun T produce_named(String name)

// CONTRAVARIANT — T appears only in argument positions
type Consumer<T>
    fun String consume(T item)

// INVARIANT — T appears in both positions (sound default for read/write containers)
type Box<T>
    fun T get()
    fun set(T value)
```

**Invariant by default.** Type parameters are invariant unless position
analysis derives otherwise. Invariance is the only sound default for
containers with both read and write access: a covariant read/write
container would permit unsound writes (the classic `String[]` →
`Object[]` store problem). Covariance and contravariance are therefore
opt-in — earned by a type whose method signatures place `T` in a single
position family.

**Practical consequences** (on Orthon's real subtyping sources):

```orthon
// Covariant: Producer<Cat> is usable where Producer<Cat | Animal> is expected
//   (Cat <: Cat | Animal; Producer is covariant)
fun register(Producer<Cat | Animal> source) ...
let cats = Producer<Cat>()
register(cats)            # OK — covariant

// Contravariant: Consumer<Cat | Animal> is usable where Consumer<Cat> is expected
//   (a sink accepting more is usable where a sink accepting fewer is needed)
fun hookup(Consumer<Cat> sink) ...
let any = Consumer<Cat | Animal>()
hookup(any)               # OK — contravariant
```

**Invalid: re-declaring a class type parameter in a method.** A method
must not re-declare a type parameter of its enclosing type. Shadowing is
a compile error — explicitness requires distinct names.

```orthon
// INVALID — <T> shadows the class-level T
type Producer<T>
    fun T produce<T>(T item)   # ✗ ERROR: duplicate type parameter T
```

### Associated Type Resolution

Associated types are resolved during monomorphisation. The compiler substitutes the concrete type's associated type for the trait's declaration.

```orthon
trait Collection
    type Item

impl Collection for List<Int>
    type Item = Int

// When monomorphising with List<Int>, Collection::Item becomes Int
```

### Cross-Reference: COMPILE_TIME_EXECUTION (Plan 04-03)

Generics interact with compile-time execution in two ways:

1. **Comptime generic evaluation** — If `comptime` is adopted, generic type computations may execute at compile time, enabling type-level programming with generic parameters.
2. **Comptime trait resolution** — Trait bounds on generic parameters may be resolved at compile time for `comptime` contexts.

The precise interaction is specified in Phase 04-03 (COMPILE_TIME_EXECUTION).

## Default Strategy

Static dispatch via monomorphisation with trait bounds. Invariant by default. `where` clauses for complex bounds, inline `<Iterator as T>` for simple bounds. `+` combines bounds on one parameter; `,` separates parameters. Associated types resolved during monomorphisation. No type erasure.

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
4. Should negative trait bounds (`where T as !Hash`) be supported in v0.2+?
5. How does generics interact with the Metadata Protocol (`@`) — should `T@name` expose the concrete type name?

## Decision History

- **Trait-bounded generics** adopted over duck-typed templates (C++) and type erasure (Java). Rationale: Explicit constraints provide type safety and clear error messages while preserving all performance via monomorphisation.
- **Static dispatch by default** adopted. Rationale: Performance, inlining capability, and consistency with the trait system.
- **Invariant by default** adopted. Rationale: Safe, conservative. Covariance/contravariance are declared via trait method signatures.
- **No type erasure** adopted. Rationale: Preserves type information for LLM tooling, schema provision, and reflection.
- **Accepted via EDR-024** on 2026-07-27.
- **Syntax revision via EDR-086** on 2026-08-07: angle-bracket parameters `<T>` (replacing `[T]`), bound-first inline shorthand `<Iterator as T>`, `where` clauses via `as` (`where T as Hash + Eq`) with `+` for bounds and `,` for parameters, `fun`/`proc`/`new` declaration kinds, type-first parameter syntax (`T item`). Variance section rewritten around position-based inference and Orthon's real subtyping sources (unions, literals, widening); the class-hierarchy example was removed. Method-level re-declaration of a class type parameter is a compile error.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/TRAITS.md`
- [x] `how/decision_records/INDEX.md` (EDR-086)
- [ ] `what/concepts/TYPE_INFERENCE.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
