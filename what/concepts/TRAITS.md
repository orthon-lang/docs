# Traits

> **✅ ACCEPTED — [EDR-019](../how/decision_records/architecture/EDR-019-traits.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Identity,
> [`GLOSSARY.md`](../GLOSSARY.md) § Trait, Trait Bound, Orphan Rule,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a language express shared behaviour across different types — polymorphism — without the fragility of class inheritance?

The evolution of behavioural abstraction reveals a clear winner: **traits** (Rust's model) synthesise the best of interfaces (Java), typeclasses (Haskell), and structural typing (Go) while avoiding the pitfalls of each.

The core problem: Orthon needs a mechanism for types to declare that they satisfy a behavioural contract — enabling generic functions, polymorphic dispatch, and standard library abstractions — without introducing class hierarchies, fragile base classes, or ad-hoc polymorphism without coherence.

## Principles

1. **Behaviour separate from data** — Traits define only behaviour (method signatures, associated types), never data fields. Data belongs to types (structs, enums).
2. **Explicit satisfaction** — A type must explicitly declare that it implements a trait. No structural (implicit) satisfaction. Explicitness over convenience.
3. **Static dispatch by default** — Trait bounds on generic parameters use static dispatch (monomorphisation). Dynamic dispatch (`dyn Trait`) is opt-in, syntactically visible.
4. **Coherence** — A trait implementation must be defined in the same module as either the trait or the type. At most one implementation of a trait for any type. No orphan implementations.
5. **Associated types** — Traits can declare associated types, allowing a single trait to model type families (e.g., `Iterator` with `Item`).
6. **No inheritance** — Traits do not extend other traits to form hierarchies. Trait bounds express requirements: `fn sort(items: [T]) where T: Ordered`. This is composition of constraints, not inheritance.
7. **Default methods enable Template Method** — Traits with default implementations express the Template Method pattern without abstract classes, `virtual`, or `override`.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inheritance Policy | Traits replace implementation inheritance entirely. No class inheritance. |
| Dispatch Policy | Determines whether trait dispatch is static (monomorphisation) or dynamic (vtable). Default: static. |
| Coherence Policy | Controls orphan rules — no downstream implementations of foreign traits on foreign types. |
| Subtyping Policy | Traits do not create subtype relationships. A `dyn Trait` is a dynamically-dispatched handle, not a subtype. |
| Interface Reuse Policy | Traits compose via bounds (`where T: A + B`), not via extending parent traits. |

## Model (What)

A **trait** is a behavioural contract that types can implement. Traits define method signatures, associated types, and optionally provide default implementations.

### Trait Declaration and Implementation

```orthon
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

### Static vs. Dynamic Dispatch

Orthon chooses **static dispatch by default** because it eliminates indirect call overhead, enables inlining across trait boundaries, and produces smaller binaries (no vtables).

```orthon
// Static dispatch — monomorphised at compile time (default)
fn process[T: Processor](item: T)
    item.process()

// Dynamic dispatch — vtable at runtime (opt-in)
fn process_dyn(item: dyn Processor)
    item.process()
```

### Associated Types

```orthon
trait Collection
    type Item
    fn get(self, index: Int) -> Option<Self::Item>
    fn len(self) -> Int
```

Associated types allow a trait to model type-level relationships without additional generic parameters.

### Default Implementations

```orthon
trait Stringifiable
    fn to_string(self) -> String
        return "<opaque>"  // default

impl Stringifiable for Int
    // uses default — no override needed
```

### Template Method Pattern

Traits with default implementations express the **Template Method** pattern without abstract classes, `virtual`, or `override`:

```orthon
trait DataImporter
    fn open(self)                   // hook — declared in trait signature
    fn parse(self)                  // hook — declared in trait signature
    fn close(self)                  // hook — declared in trait signature

    // default implementation = template method
    fn import(self)
        self.open()
        self.parse()
        self.close()

impl DataImporter for CsvImporter
    fn open(self)    // file open logic
    fn parse(self)   // CSV parsing logic
    fn close(self)   // resource cleanup
```

The template method `import()` is inherited from the default. `impl` blocks only require providing the methods declared in the trait signature, not default methods. This eliminates accidental override.

### Coherence: The Orphan Rule

An implementation of a trait for a type must be defined in the same compilation unit as either the trait or the type. This prevents:

- Two downstream modules both implementing `ForeignTrait for ForeignType` with conflicting behaviour
- Adding behaviour to types you do not own without the type author's knowledge

```orthon
// Allowed: trait defined in this module
impl Printable for User { ... }

// Allowed: type defined in this module
impl ForeignTrait for User { ... }

// ERROR: orphan — neither trait nor type defined here
// impl ForeignTrait for ForeignType { ... }
```

### Blanket Implementations

Traits support blanket implementations using `where` clauses:

```orthon
impl<T> Printable for T where T: Display
    fn format(self) -> String
        return self.to_display()
```

Blanket implementations are subject to the orphan rule: the blanket must be in the same module as the trait being implemented.

### Named Function Equivalents

Per the Named Before Symbolic principle, trait method calls have named equivalents:

```orthon
item.process()       # method call syntax
process(item)        # free function via trait (UFCS-like)
```

## Default Strategy

Traits with explicit `impl` blocks. Static dispatch by default — generic functions with `where T: Trait` bounds are monomorphised at compile time. Dynamic dispatch (`dyn Trait`) is opt-in and syntactically visible. The orphan rule is enforced (no downstream implementations of foreign traits on foreign types). Associated types are supported. Default method implementations are supported. Blanket implementations via `where` clauses are supported.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| Full class inheritance | Java, C++ | Fragile base class problem, deep hierarchies, tight coupling. Rejected. |
| Structural interfaces | Go | Implicit satisfaction is convenient but can mask accidental interface conformance, making refactoring harder. Rejected: violates Explicitness. |
| Typeclasses with unrestricted orphans | Haskell | Orphan instances enable cross-cutting concerns but create incoherence. Rejected: Coherence principle requires the orphan rule. |
| Protocols with inheritance | Swift | Protocol inheritance creates hierarchy. Rejected: Trait bounds via `where` clauses replace inheritance. |
| Concepts | C++20 | Compile-time constraints on template parameters. Powerful but syntactically heavy and coupled to the template system. |

## Open Questions

1. Should traits support negative constraints (`where T: !FixedSize`)?
2. Should `dyn Trait` be object-safe by default, or should object safety require explicit opt-in?
3. How do traits interact with Orthon's Metadata Protocol (`@`)?
4. Should trait bounds be expressible in the Schema Provider for LLM querying?
5. What is the precise interaction between traits and Orthon's Declaration Kinds (`fun`/`proc`/`new`)? Can a trait declare a `proc` method that requires mutable access?

## Decision History

- **Traits over class inheritance** adopted. Rationale: Behaviour separate from data, static dispatch by default, coherence guarantee. Aligns with Orthogonality and Minimal Core.
- **Explicit `impl` over structural satisfaction** adopted. Rationale: Explicitness principle — a type must explicitly declare trait conformance. No accidental interface satisfaction.
- **Static dispatch by default** adopted. Rationale: Performance, inlining, and binary size. Dynamic dispatch via `dyn` is opt-in and visible.
- **Orphan rule** adopted. Rationale: Coherence — at most one implementation of a trait for any type. Prevents conflicting implementations.
- **No trait inheritance** adopted. Rationale: Trait bounds via `where T: A + B` replace hierarchies with composition of constraints.
- **Accepted via EDR-019** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/SEMANTIC_MODEL.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
