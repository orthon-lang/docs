# Root Object

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

Does every value in Orthon share a common ancestor type — a universal parent
class, like Java's `Object`, C#'s `object`, or Kotlin's `Any` — that supplies
a baseline set of behaviours (string conversion, equality, hashing, type
identification) to anything that exists in the language?

Class-based languages answer this question in one of two ways:

1. **Concrete root class (Java, C#, Kotlin, Python 3, Scala)** — every class
   implicitly inherits from a single root (`Object`, `object`, `Any`, `object`,
   `Any`). The root defines a small fixed set of methods (`toString()`,
   `equals()`, `hashCode()`, `getClass()`) that every instance inherits and
   may override. This guarantees that *any* value can be printed, compared,
   and hashed without the caller knowing its concrete type.

2. **No universal root (Rust, Go)** — there is no base class. Common
   behaviour is expressed as traits/interfaces (`Display`, `Debug`, `Hash`,
   `Eq` in Rust; `Stringer`, `error` in Go) that a type opts into explicitly.
   A generic function that needs "printable" or "hashable" behaviour
   constrains its type parameter to that trait/interface rather than
   assuming a root type. There is no value that is *not* some concrete type,
   and no way to hold "any value" without an explicit top type (`dyn Any`,
   `interface{}`/`any`) that itself carries no methods.

The root-class model buys convenience — `toString()` always works, `Object`
is a universal existential container, reflection has one entry point — at
the cost of two well-known problems:

- **Every object inherits methods it may not want.** `equals`/`hashCode`
  contracts are notoriously easy to violate (asymmetric equality, hash/equals
  drift) precisely because they are inherited defaults rather than
  deliberate opt-ins. Java's `Object.equals` is reference equality, which is
  almost never what a data type wants — nearly every class ends up
  overriding it, making the "default" actively misleading.
- **A root class implies implementation inheritance**, since the common
  methods are inherited, not composed. This is in direct tension with
  Orthon's committed default: composition-only, no implementation
  inheritance (see [`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md)).

The no-root model avoids both problems but reintroduces a different one:
without a common ancestor, a function cannot accept "a value of any type" in
a data-first language the way `Object param` or `interface{}` allows in
inheritance-based languages — every point of genuine dynamism (heterogeneous
collections, reflection, generic logging, a plugin boundary) needs its own
explicit top type or trait bound.

The core problem: **should Orthon guarantee that every value supports a
fixed set of universal operations, and if so, is that guarantee delivered
through a concrete root class (inherited), a structural interface (composed),
or standalone functions (no method dispatch at all)?**

## Principles

Referencing [`../DESIGN_PRINCIPLES.md`](../DESIGN_PRINCIPLES.md) and
[`../../why/MANIFESTO.md`](../../why/MANIFESTO.md):

1. **Composition over inheritance** — Orthon's default strategy is
   composition-only, structural interface satisfaction, no implementation
   inheritance (see [`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md)).
   A concrete root *class* that every type implicitly extends is
   implementation inheritance by another name — universal, unconditional,
   and impossible to opt out of. Any root-object design must justify why
   this one case earns an exception to the composition-only default.
2. **Data First** — data is a value without imposed semantic meaning
   (see [`DATA_MODEL.md`](DATA_MODEL.md)). A root *object* smuggles an
   object-oriented mental model (identity, inherited behaviour, `getClass()`
   reflection) into a language whose primary abstraction is data, not
   objects.
3. **Explicit Equality** — Orthon defines `===` (value), `==` (semantic),
   and `is` (identity) as three distinct, explicit operators
   (see [`EQUALITY.md`](EQUALITY.md)). A root class's inherited
   `equals()`/`hashCode()` pair — a single overridable method standing in
   for all three semantics — is precisely the ambiguity this design
   principle exists to prevent.
4. **Orthogonality** — each construct should solve one problem. A root class
   conflates at least three orthogonal concerns: (a) universal
   string conversion, (b) universal equality/hashing, (c) a top type for
   heterogeneous storage and reflection. These do not need to be delivered
   by the same mechanism, let alone the same inherited type.
5. **Minimal Core** — if a universal top type is needed at all (for
   heterogeneous collections, dynamic interop, reflection), it should carry
   the minimum surface required for that purpose, not an open-ended,
   growable method set that invites every future cross-cutting concern to
   be bolted onto it (as `Object` has accumulated over decades in Java).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inheritance Policy | Determines whether a universal root type exists and whether it is delivered via implementation inheritance or structural composition |
| Comparison Policy | Determines whether universal equality/hashing is inherited from a root type or supplied per-type via explicit interface satisfaction |
| Representation Policy | Determines whether a top type is a real supertype of every Representation (Value, Tuple, Reference, Sequence, Set, Option, Result) or an existential wrapper around them |
| Reflection Policy | Determines whether runtime type identification (`getClass()`-equivalent) is a root-type method or a standalone reflection facility |

*The specific list is filled in during concept design and passes
verification through `IMPLEMENTATION_INDEPENDENCE_GATE`.*

## Model (What)

Three candidate models, in decreasing order of inheritance commitment:

**1. Concrete root class (Java/C#/Kotlin model).** Every type implicitly
extends a single root (e.g. `Root`), which declares:

```
class Root:
    to_string() -> String
    hash() -> Int
    equals(other: Root) -> Bool
    type_of() -> TypeInfo
```

Any value can be widened to `Root`, printed, compared, and hashed without
knowing its concrete type. This is the angle explored as primary in this
document, per its motivating question — a Java-`Object`-style root.

**2. Structural interface, no inheritance (composition model).** No root
class. Instead, a small set of interfaces capture the same guarantees, and
the compiler auto-derives conformance structurally wherever possible:

```
interface Showable:
    to_string() -> String

interface Hashable:
    hash() -> Int

interface Equatable:
    equals(other: Self) -> Bool
```

A function that needs "any showable value" constrains its parameter to
`Showable`, not to a root type. There is no single type that every value
belongs to; there are only types that satisfy zero or more of these
interfaces (auto-derived by the compiler for structural data, unless
suppressed).

**3. No universal guarantee; standalone functions (Go/Clojure model).**
`to_string`, `hash`, and `equals` are free functions with structural or
generic implementations, not methods on any type. Types opt in by
satisfying a function signature, not by inheriting or implementing anything
nominal. There is no notion of "widening to a common type" at all; a
heterogeneous container is expressed explicitly (e.g. a tagged union or an
explicit top type built only where dynamism is genuinely needed).

## Default Strategy

Given the **Composition over inheritance** and **Data First** principles
already committed to elsewhere in the specification, Model 2 (structural
interfaces, no inheritance) is the strategy most consistent with Orthon's
existing direction: universal guarantees (`Showable`, `Hashable`,
`Equatable`) are expressed as structural interfaces the compiler
auto-derives for data types, not as inherited members of a concrete root
class. There is no `Root`/`Object`/`Any` type that every value extends.

Where a genuine top type is needed (heterogeneous collections, dynamic
interop boundaries, reflection), it is a narrow, explicitly-named construct
— not the same type that supplies `to_string`/`hash`/`equals` — keeping the
three concerns identified under Orthogonality separate.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Concrete root class (Model 1) | A single `Object`-equivalent that every type implicitly inherits, with virtual `to_string`/`hash`/`equals` | If Orthon later admits a class-inheritance strategy (e.g. for an interop-with-JVM/.NET Implementation Strategy) where host-platform objects already have a root and mirroring it minimizes friction |
| No universal guarantee (Model 3) | Free functions instead of methods; no top type at all | Embedded/high-performance strategies where even structural interface dispatch is considered unnecessary overhead, and heterogeneous storage is rare enough to special-case explicitly |
| Marker-only top type | A top type with zero methods, used purely for existential storage (`dyn Any`/`interface{}` equivalent), fully decoupled from `Showable`/`Hashable`/`Equatable` | If reflection or FFI genuinely requires holding "a value of unknown type," without implying any behavioural guarantee |

## Open Questions

1. Is a Java-style concrete root class disqualified outright by the
   Composition over Inheritance principle, or is there a narrow reading
   under which a *single, fixed, non-extensible* root (no further
   inheritance from it beyond the one level) is acceptable because it does
   not create the fragile-base-class or deep-hierarchy problems that
   principle actually targets?
2. If Model 2 is chosen, does the compiler auto-derive `Showable`/
   `Hashable`/`Equatable` for all structural data by default (with opt-out),
   or must types explicitly declare conformance (opt-in)? This mirrors the
   opt-in/opt-out tension already open in
   [`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md).
3. Does Orthon need a top type at all in v0.1, or can heterogeneous storage
   and reflection be deferred to a later milestone as a narrowly-scoped
   addition rather than a foundational commitment?
4. How does a chosen model interact with [`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md)'s
   `object` singleton construct — does a singleton also satisfy
   `Showable`/`Hashable`/`Equatable`, and if there is no root type, what (if
   anything) does `object X` implicitly extend?
5. How does `to_string`/`Showable` relate to serialization — is
   human-readable string conversion the same mechanism as structured
   serialization (JSON, binary), or a deliberately separate, narrower
   concern?
6. If reflection (`type_of()`/`TypeInfo`) is retained, should it live on
   every value (root-class style) or be a capability granted only to types
   that opt in, consistent with Minimal Core?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `COMPOSITION_OVER_INHERITANCE.md`
- [ ] `EQUALITY.md`
- [ ] `OBJECTS_AND_SINGLETONS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
