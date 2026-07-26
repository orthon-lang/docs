# Composition Over Inheritance

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language enable code reuse and subtype polymorphism without the fragility of class inheritance hierarchies?

Two fundamentally different approaches dominate:

1. **Class inheritance (Java, C++, C#)** — a class extends another class via `extends`. Subtypes inherit behaviour *and* structure from their parent. Reuse comes from the hierarchy, but at a cost: the fragile base class problem (changes to a parent class can break all descendants), the diamond problem (multiple inheritance ambiguity), and deep hierarchies that make code hard to reason about. Inheritance creates a tight coupling between parent and child that grows tighter over time.

2. **Composition with interfaces (Go, Rust)** — there are no classes; only structs (or similar product types) and interfaces/traits. Reuse happens through embedding one struct in another (composition) or by satisfying interface contracts. There is no hierarchy — only relationships. Go's approach is the most radical: no `extends`, no `implements` keyword, no abstract base classes — just structs, embedding, and interface satisfaction.

The core problem: **inheritance captures an "is-a" relationship that often doesn't exist in the domain**, forcing programmers to choose between artificial hierarchy or code duplication. Composition captures "has-a" or "behaves-like" relationships, which are more stable as requirements evolve.

## Principles

1. **Favor composition over inheritance** — Code reuse through embedding/aggregation is preferred over class hierarchy. Inheritance, if provided, should be opt-in and limited (e.g., interface inheritance only, no implementation inheritance).

2. **Interfaces as behavioural contracts** — Polymorphism is achieved through interface/trait satisfaction, not through class hierarchy. A type satisfies an interface if it provides the required methods, regardless of any explicit declaration.

3. **No fragile base class** — Changes to one type should not silently break subtypes. Composition avoids this because there is no parent-child coupling.

4. **Flat over deep** — Type relationships should be flat (a type satisfies zero or more interfaces) rather than deep (a type inherits from a grandparent). Deep hierarchies are hard for both humans and LLMs to reason about.

5. **Explicit embedding** — If a language supports embedding (struct-in-struct), the embedded type's methods should be promoted to the outer type only through explicit opt-in, not automatic forwarding.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inheritance Policy | Determines whether implementation inheritance is allowed (full classes), restricted (interface-only), or absent (composition-only) |
| Embedding Policy | Governs whether struct/types can embed other types and how method promotion works |
| Subtyping Policy | Determines whether subtyping is nominal (explicit `implements`) or structural (duck typing) |
| Interface Reuse Policy | Controls whether interfaces can embed/compose other interfaces |

## Model (What)

The composition-over-inheritance model replaces class hierarchies with:

- **Structs (or product types)** — Named collections of fields. No methods inherited from a parent; methods are attached directly or promoted from embedded types.
- **Interfaces** — Sets of method signatures. A type satisfies an interface implicitly (structural) if it has the required methods, or explicitly (nominal) if it declares satisfaction.
- **Embedding** — A type can embed another type, gaining access to its fields and methods. Method promotion is explicit and does not create subtyping.
- **No abstract base classes** — Code reuse through embedding + interface defaults, not through abstract class hierarchy.

This is the Go model. Rust uses traits instead of interfaces but follows the same principle: composition, not inheritance.

## Default Strategy

The default strategy uses **composition-only** (no implementation inheritance) with **structural interface satisfaction**. Types are structs; polymorphism is via interfaces; embedding provides method promotion without subtyping.

## Alternative Strategies

1. **Limited inheritance** — Allow single implementation inheritance (like Kotlin's `open`/`final` by default) but discourage deep hierarchies through linting or code review.

2. **Trait-based reuse** — Allow default method implementations on interfaces/traits (like Rust or Java 8+), providing code reuse without class inheritance.

3. **Mixins** — Allow composing behaviour through mixin classes (like Scala traits before Java 8), with linearization rules for conflict resolution.

## Open Questions

1. Should Orthon allow any form of implementation inheritance, or commit to composition-only?
2. If embedding is allowed, should promoted methods be automatic or require explicit delegation?
3. Should interfaces support default method implementations (like Java 8+), or remain pure contracts?
4. How does embedding interact with visibility rules — do embedded private fields become accessible?

## Cross-References

- See `STRUCTURAL_TYPING.md` for the interface satisfaction mechanism
- See `VISIBILITY_AND_ENCAPSULATION.md` for how visibility interacts with embedding
- See `DATA_MODEL.md` for how structs fit into the data model
- See [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Minimal Core for the principle that guides composition over hierarchy
