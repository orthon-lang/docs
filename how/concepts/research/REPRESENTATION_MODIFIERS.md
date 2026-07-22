# Representation Modifiers

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

Given a user-defined type (a record/class/struct with named fields), what
mechanism does the programmer use to control *how* values of that type are
stored in memory — without changing the *semantics* of the type?

In mainstream languages, the storage strategy of a type is baked into its
definition. A Python class is always a heap-allocated reference type with a
per-object dictionary. A Java `class` is always a heap-allocated reference.
A Rust `struct` is always an inline value. Changing the storage strategy
requires changing the type itself — or writing conversion code between two
structurally identical types:

```rust
// Rust — changing storage means changing the type
struct Point { x: f64, y: f64 }       // value, inline

use std::boxed::Box;
struct BoxedPoint(Box<Point>);        // different type, same fields
```

This creates a tension: the programmer must choose a storage strategy at
type-definition time, even though the optimal strategy depends on usage
context (hot path, embedded, distributed, FFI boundary). Languages that
offer escape hatches — `__slots__`, `ctypes.Structure`, `memoryview`,
`Buffer Protocol` — are each ad-hoc mechanisms with different syntax,
different guarantees, and no unified mental model.

The core problem: **can Orthon provide a family of representation modifiers
that decouple *what a type is* from *how it is stored*, allowing the
programmer (or the compiler) to select a materialization strategy per-use
site without changing the type's semantic identity?**

## Principles

1. **Semantics Before Optimization** (from [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)) —
   The semantic meaning of a type is independent of its storage strategy.
   `struct(T)` and `boxed(T)` represent the *same* data; they differ only
   in *how* that data is laid out in memory.

2. **Explicitness** (from [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)) —
   Changes in representation are syntactically visible. The programmer
   explicitly declares `struct(User)` or `shared(User)`; the compiler does
   not silently change the materialization of a type.

3. **Orthogonality** (from [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)) —
   Representation modifiers compose with any type. There is no special case:
   every type can be `struct`ed, `boxed`, `shared`, `atomic`, `ffi`, or
   `packed` — and these modifiers can combine with Data Modifiers
   (Sequence, Option, Result, etc.).

4. **Data First** (from [`MANIFESTO.md`](../../../why/MANIFESTO.md) and
   [`FOUNDATIONAL_ABSTRACTIONS.md`](FOUNDATIONAL_ABSTRACTIONS.md)) —
   A type's essence is the data it carries. Representation modifiers affect
   only storage, not semantics. This extends the existing "Data + Data
   Modifiers" model into the domain of user-defined types.

5. **Intent Over Implementation** (from [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md)) —
   The programmer states *what* representation they need (`ffi(User)`);
   the compiler determines the low-level details (alignment, padding,
   calling convention).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Representation Policy | Defines how each representation modifier maps to runtime memory layout |
| Allocation Policy | Determines storage location (stack, heap, arena, shared memory) for each modifier |
| Lifetime Policy | Governs when representation memory is reclaimed (scope-bound, reference-counted, GC-traced) |
| Equality Policy | Controls structural vs. identity comparison across different representations of the same type |
| FFI Policy | Specifies C-compatible layout and calling convention for `ffi(T)` |

## Model (What)

### Core Insight

`struct(T)` is not a new type — it is a **structural representation of
type T**. The programmer writes:

```
type User
    let id: Int
    let name: String
    let email: String
```

`User` as declared carries the full semantic meaning: methods, interfaces,
reflection metadata, runtime type information, fields. Applying a
representation modifier selects a specific materialization:

```
struct(User)     // plain data — only fields, no runtime overhead
boxed(User)      // heap-allocated object with full runtime info
shared(User)     // reference-counted, shared ownership
atomic(User)     // thread-safe, atomic access guarantees
ffi(User)        // C-compatible memory layout (POD)
packed(User)     // densely packed with minimal alignment padding
```

All refer to the *same semantic type* `User`. They differ only in memory
layout, ownership model, and runtime metadata. Values can convert between
representations when needed:

```
let u = User(id: 1, name: "Alice", email: "alice@example.com")
let data = struct(u)    // strip to plain data layout
let obj   = boxed(data) // re-inflate to runtime object
```

### Relationship to Existing Data Modifiers

Orthon already defines data-level representation modifiers in
[`DATA_MODEL.md`](DATA_MODEL.md) § Model: Value, Tuple, Reference,
Sequence, Set, Option, Result. These operate on **data values** — they
describe the shape of a single value.

Representation modifiers operate on **types** — they describe the storage
strategy for values of a given type. They are orthogonal to data-level
modifiers:

```
sequence(struct(User))     // a sequence of plain-data Users
option(boxed(User))        // an optional heap-allocated User
result(ffi(User), Error)   // a result whose success value is C-compatible
```

### Type Identity Across Representations

A critical property: `struct(T)` and `boxed(T)` are **the same type** with
different storage. This means:

- Assignment between representations is safe (the compiler inserts the
  conversion):
  ```
  let u: User = ...
  let data: struct(User) = struct(u)    // explicit
  let back: boxed(User) = boxed(data)   // explicit
  ```

- Equality is defined across representations:
  ```
  struct(u) == struct(v)    // structural, fast
  boxed(u) == boxed(v)      // structural (dereferences pointers)
  struct(u) == boxed(v)     // structural (mixed representations)
  ```

- Method dispatch works regardless of representation (the compiler
  materializes the representation as needed).

### Syntax for Type Declarations

A type can be declared with a default representation strategy, but
callers can override it:

```
// Declare default
type User                             // default: boxed (full object)

compact type User                     // default: struct (compact)
shared type User                      // default: reference-counted
```

Or, following the Execution Program philosophy (semantics declared,
execution strategy selected per-target), the type's default
representation can be left unspecified and selected by the
Implementation Strategy:

```
type User                             // representation = policy decision
```

## Examples from Other Languages

| Mechanism | Language | What it does | Limitation |
|---|---|---|---|
| `__slots__` | Python | Removes per-object dict, fixes attribute set | Same type, runtime only, no compile-time guarantee |
| `ctypes.Structure` | Python | C-compatible layout for FFI | Different type system, manual field mapping |
| `struct` / `class` | C# | `struct` = value type, `class` = reference type | Choice baked into type definition; cannot change per-use |
| `Box<T>` / `Rc<T>` | Rust | Heap allocation wrappers | Different types; conversion is explicit but verbose |
| `repr(C)` / `repr(packed)` | Rust | Attribute-driven layout control | Applies to type definition, not per-use site |
| `@Packed` / `@Atomic` | D | UDAs (User-Defined Attributes) on structs | Fixed at declaration; per-use not supported |
| `std::is_trivial<T>` | C++ | Type trait query (read-only) | No way to *select* a representation |
| `flatbuffers::FlatBufferBuilder` | C++ | Packed serialized representation | External library, not language construct |
| `record` | Java (16+) | Data carrier with auto-generated methods | Always a class (reference type); no layout control |
| `data class` | Kotlin | Auto-generates `equals`, `hashCode`, `toString`, `copy` | Always a class; no struct/compact variant |

### Analysis

No mainstream language provides a unified, orthogonal system where:

1. The programmer writes the semantic type once.
2. Representation is selected per-use site via a modifier.
3. All representations of the same type maintain type identity.
4. The compiler handles the conversion mechanics.

The closest approaches are Rust's `Box<T>`/`Rc<T>` wrappers (but these
are distinct types, breaking type identity) and C#'s `struct`/`class`
choice (but the choice is baked into the definition, not per-use).

## Default Strategy

If no representation modifier is specified, the default is determined by
the active Implementation Strategy (see
[`IMPLEMENTATION_STRATEGIES.md`](../../strategies/IMPLEMENTATION_STRATEGIES.md)):

- **Default Strategy:** `boxed(T)` — heap-allocated with full runtime
  metadata. Suitable for general-purpose use where polymorphism,
  reflection, and method dispatch are common.
- **Embedded Strategy:** `struct(T)` — plain data, no runtime overhead.
  Suitable for memory-constrained environments.
- **High-Performance Strategy:** `packed(T)` or `struct(T)` — compact
  layout, cache-line optimised.

The programmer can always override the strategy default at the use site:

```
let local: struct(User) = ...     // always compact, regardless of strategy
```

## Alternative Strategies

| Modifier | Semantics | When to use |
|---|---|---|
| **`struct(T)`** | Plain data; only fields, no runtime metadata, no GC tracking | Data transfer objects, serialization, embedded, hot loops |
| **`boxed(T)`** | Heap-allocated, full runtime info (methods, interfaces, reflection, GC root) | Polymorphic dispatch, dynamic typing, general use |
| **`shared(T)`** | Reference-counted, shared ownership | Cross-thread or cross-component data sharing |
| **`atomic(T)`** | Thread-safe reads/writes, atomicity guarantees | Concurrent data access without explicit locks |
| **`ffi(T)`** | C-compatible ABI (standard layout, predictable alignment, no padding gaps) | FFI boundaries, interop with C libraries, system calls |
| **`packed(T)`** | Dense packing; minimum alignment, no padding | Memory-constrained targets, wire protocols, file formats |

### Composability

Representation modifiers compose with each other when the combination is
meaningful:

```
shared(struct(User))     // reference-counted, internally compact
atomic(boxed(User))      // atomic pointer to heap object
ffi(packed(User))        // C-compatible, densely packed
```

The compiler rejects nonsensical combinations (e.g., `struct(boxed(T))` —
stripping runtime metadata from something that is already plain data — is
a no-op, not an error).

## Open Questions

1. **Type identity across representations.** Is `struct(User)` truly the
   same type as `boxed(User)`, or is it a view/projection? If they are
   the same type, how does method dispatch work when the representation
   doesn't carry a vtable?

2. **Conversion cost.** Converting between representations may require
   allocation (e.g., `struct(T)` → `boxed(T)` needs a heap allocation).
   Should conversion be implicit or explicit? The current proposal favors
   explicit conversion (via modifier syntax) to surface the cost.

3. **User-defined representation modifiers.** Should Orthon allow
   libraries to define custom representation modifiers (e.g.,
   `mmap(T)`, `gpu(T)`, `remote(T)`), or is the set fixed? The
   "Extensibility over built-in magic" principle from
   [`MANIFESTO.md`](../../../why/MANIFESTO.md) suggests the former.

4. **Interaction with mutable fields.** If a `struct(T)` strips runtime
   metadata, but the type has mutable fields, does mutation through
   `struct(T)` still work? It must — but the mechanism differs from
   `boxed(T)` where mutation goes through a heap pointer.

5. **Cross-representation equality.** Should `struct(a) == boxed(b)` be
   defined structurally (comparing field values) or is it a type error?
   The proposal argues for structural equality, but this affects code
   generation (the compiler must materialize both sides to compare).

6. **What about `class(T)`?** Is `class(T)` a synonym for `boxed(T)` in
   the representation modifier system, or does it carry additional
   semantics (inheritance, virtual methods)? The modifier system
   suggests representation is orthogonal to behavior: a type can have
   methods regardless of whether it is `struct` or `boxed`.

7. **LLM Generability.** Can an LLM reliably produce correct code when
   representation modifiers are a single keyword applied to an existing
   type name? The syntax `struct(User)` is simple, but the semantic
   implications (conversion costs, method dispatch, equality) may be
   subtle. Does this pass the LLM Generability Gate from
   [`DECISION_VALIDATION.md`](../../gates/DECISION_VALIDATION.md)?

## Implications for Orthon

### Alignment with Existing Design

This concept fits naturally into Orthon's existing architecture:

- It extends the **Data + Data Modifiers** dichotomy from
  [`FOUNDATIONAL_ABSTRACTIONS.md`](FOUNDATIONAL_ABSTRACTIONS.md) into
  the type domain. At the data level, modifiers change *view* (Value,
  Tuple, Reference, etc.); at the type level, modifiers change
  *storage* (struct, boxed, shared, etc.).

- It reinforces the **Semantics Before Optimization** and **Intent Over
  Implementation** principles from
  [`DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md). The programmer
  declares *what representation* they need; the compiler handles *how*.

- It complements the **Execution Program** model from
  [`EXECUTION_PROGRAM.md`](EXECUTION_PROGRAM.md). Just as execution
  strategy is a deployment-time decision, representation strategy is a
  use-site decision — both are decoupled from the program's semantic
  definition.

### Relationship to Related Concepts

| Concept | Relationship |
|---|---|
| [`DATA_MODEL.md`](DATA_MODEL.md) | Data-level representations (Value, Tuple, etc.) are orthogonal; type-level representation modifiers compose with them |
| [`SLOTS.md`](SLOTS.md) | `struct(T)` subsumes the slots concept — it is a general mechanism that includes fixed-field compact storage |
| [`DATACLASSES.md`](DATACLASSES.md) | Representation modifiers do not generate boilerplate (equals, hash, toString); that remains a separate concern |
| [`VALUE_SEMANTICS.md`](VALUE_SEMANTICS.md) | `struct(T)` is value-like (copied on move), `boxed(T)` is reference-like — the modifier system makes both available for the same type |
| [`ALLOCATION.md`](ALLOCATION.md) | Each modifier implies an allocation strategy; `struct(T)` is stack/arena, `boxed(T)` is heap |
| [`OWNERSHIP.md`](OWNERSHIP.md) | `shared(T)` implies reference-counted ownership; `struct(T)` implies unique/linear ownership |
| [`FOUNDATIONAL_ABSTRACTIONS.md`](FOUNDATIONAL_ABSTRACTIONS.md) | Representation modifiers extend the Data + Data Modifiers model to types |

## Decision History

This document is exploratory research. No decisions have been made.
Formal consideration will occur during Concept Design Review (Milestone 2).

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md` — if concept is accepted
- [ ] `what/GLOSSARY.md` — if new terminology is introduced
- [ ] `what/SEMANTIC_MODEL.md` — representation modifiers affect the semantic model
- [ ] `how/concepts/research/DATA_MODEL.md` — relationship to existing data representations
- [ ] `how/concepts/research/FOUNDATIONAL_ABSTRACTIONS.md` — extension of Data + Data Modifiers
- [ ] `how/DESIGN_PRINCIPLES.md` — may need explicit "Representation over Inheritance" or similar
- [ ] `how/IMPLEMENTATION_POLICIES.md` — new Policy Types may be required
- [ ] Other: `how/strategies/IMPLEMENTATION_STRATEGIES.md` — each strategy defines a default modifier
