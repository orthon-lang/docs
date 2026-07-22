# Traits

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language express shared behaviour across different types — polymorphism — without the fragility of class inheritance?

The evolution of behavioural abstraction reveals a progression:

1. **Interfaces (Java, C#)** — explicit contracts that types declare via `implements`. Nominal: a type only satisfies an interface if it says so. Interfaces carry only method signatures (pre-Java 8), then default methods (Java 8+). No data, no associated types, no static dispatch.

2. **Typeclasses (Haskell)** — ad-hoc polymorphism where behaviour is defined independently of types. A type can satisfy a typeclass without modifying the type's definition (orphan instances). Associated types and functional dependencies model type-level constraints.

3. **Traits (Rust)** — a synthesis of interfaces and typeclasses: traits define behaviour (like interfaces), support default implementations and associated types (like typeclasses), and can be implemented for any type (like typeclasses), but enforce **coherence** (no orphan implementations in downstream crates). Traits support both static dispatch (`impl Trait`, generics with trait bounds) and dynamic dispatch (`dyn Trait`, vtable-based).

4. **Structural typing (Go, TypeScript)** — a type satisfies an interface implicitly if it has the required methods. No explicit `implements` declaration. Go's interfaces are satisfied structurally; TypeScript's structural type system is more general but can produce complex error messages.

| Concept | Java Interfaces | Rust Traits | Go Interfaces | Haskell Typeclasses |
|---|---|---|---|---|
| **Satisfaction** | Explicit (`implements`) | Explicit (`impl Trait for Type`) | Implicit (structural) | Explicit (`instance TypeClass Type`) |
| **Default methods** | Yes (Java 8+) | Yes | No | Yes (via default) |
| **Associated types** | No | Yes | No | Yes |
| **Static dispatch** | No | Yes (`impl Trait`, generics) | No (except inlining) | Yes (monomorphisation) |
| **Dynamic dispatch** | Yes (vtable) | Yes (`dyn Trait`) | Yes (interface values) | No (existentials via extensions) |
| **Coherence** | By declaration | Orphan rule | Structural — always coherent | Orphan instances allowed |
| **Data** | No | No (data in structs) | No | No (data in data types) |

The core problem: **how does Orthon express polymorphic behaviour** — a contract that multiple types can fulfil — without the fragility of class inheritance (Java) or the complexity of typeclasses (Haskell)?

## Principles

1. **Behaviour separate from data** — Traits define only behaviour (method signatures, associated types), never data fields. Data belongs to types (structs, enums).

2. **Explicit satisfaction** — A type must explicitly declare that it implements a trait. No structural (implicit) satisfaction — explicitness over convenience.

3. **Static dispatch by default** — Trait bounds on generic parameters use static dispatch (monomorphisation). Dynamic dispatch (`dyn Trait`) is opt-in, syntactically visible.

4. **Coherence** — A trait implementation must be defined in the same crate as either the trait or the type. No orphan implementations. At most one implementation of a trait for any type.

5. **Associated types** — Traits can declare associated types, allowing a single trait to model type families (e.g., `Iterator` with `Item`).

6. **No inheritance** — Traits do not extend other traits to form hierarchies. Trait bounds express requirements: `fn sort(items: [T]) where T: Ordered`. This is composition of constraints, not inheritance.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inheritance Policy | Traits replace implementation inheritance entirely. No class inheritance. |
| Dispatch Policy | Determines whether trait dispatch is static (monomorphisation) or dynamic (vtable). Default: static. |
| Coherence Policy | Controls orphan rules — whether traits can be implemented for foreign types in downstream crates. |
| Subtyping Policy | Traits do not create subtype relationships. A `dyn Trait` is a dynamically-dispatched handle, not a subtype. |
| Interface Reuse Policy | Traits compose via bounds (`where T: A + B`), not via extending parent traits. |

## Model (What)

A trait is a behavioural contract that types can implement:

```
// Trait declaration
trait Printable
    fn format(self) -> String

// Implementation for a concrete type
impl Printable for User
    fn format(self) -> String
        return "User({self.name})"

// Generic function with trait bound
fn print_all(items: [T]) where T: Printable
    for item in items
        print(item.format())
```

### Static vs Dynamic Dispatch

```
// Static dispatch — monomorphised at compile time
fn process[T: Processor](item: T)
    item.process()

// Dynamic dispatch — vtable at runtime
fn process_dyn(item: dyn Processor)
    item.process()
```

The static form generates specialised code for each concrete type. The dynamic form uses a single code path with a vtable lookup. Orthon chooses static dispatch by default because it:
- Eliminates indirect call overhead
- Enables inlining and optimisation across trait boundaries
- Produces smaller binaries in embedded contexts (no vtables)

### Associated Types

```
trait Collection
    type Item
    fn get(self, index: Int) -> Option<Self::Item>
    fn len(self) -> Int
```

Associated types allow a trait to model type-level relationships without additional generic parameters.

### Default Implementations

```
trait Stringifiable
    fn to_string(self) -> String
        return "<opaque>"  // default

impl Stringifiable for Int
    // uses default — no override needed unless custom behaviour wanted
```

## Default Strategy

Traits with explicit `impl` blocks, static dispatch by default, dynamic dispatch opt-in via `dyn`, an orphan rule (no downstream implementations of foreign traits on foreign types), associated types, and default method implementations. Generic functions use `where` clauses for trait bounds.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Full class inheritance** | Java, C++ | Fragile base class problem, deep hierarchies, tight coupling. Rejected. |
| **Structural interfaces** | Go | Implicit satisfaction is convenient but can mask accidental interface conformance, making refactoring harder to reason about. |
| **Typeclasses with unrestricted orphans** | Haskell | Orphan instances enable cross-cutting concerns but can create incoherence (two instances of the same typeclass for the same type in scope). |
| **Protocols with inheritance** | Swift | Protocols can inherit from other protocols, creating hierarchy. Associated types with generic constraints add complexity. |
| **Concepts** | C++20 | Compile-time constraints on template parameters. Powerful but syntactically heavy and tightly coupled to the template system. |

## Open Questions

1. Should traits support blanket implementations (`impl<T> Trait for T where T: OtherTrait`)?
2. Should `dyn Trait` be object-safe by default, or should object safety require explicit opt-in?
3. How do traits interact with Orthon's Data and Data Modifiers foundational abstractions?
4. Should traits be expressible in the Schema Provider for LLM querying?
5. Orphan rule: strict (Rust-style) or relaxed (allow downstream impls on foreign types with explicit disambiguation)?
6. Should trait bounds allow negative constraints (`where T: !FixedSize`)?

## References

- [`COMPOSITION_OVER_INHERITANCE.md`](./COMPOSITION_OVER_INHERITANCE.md) — related: interfaces as alternative to inheritance
- [`FOUNDATIONAL_ABSTRACTIONS.md`](./FOUNDATIONAL_ABSTRACTIONS.md) — Data and Data Modifiers as foundational abstractions
- [`ALGEBRAIC_DATA_TYPES.md`](./ALGEBRAIC_DATA_TYPES.md) — ADTs as the data counterpart to traits (behaviour)
- [`FUNCTIONS.md`](./FUNCTIONS.md) — functions as the unit of computation; traits provide polymorphic functions
