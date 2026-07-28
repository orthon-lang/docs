# Algebraic Data Types

> **✅ ACCEPTED — [EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`TRAITS.md`](TRAITS.md), [`PATTERN_MATCHING.md`](PATTERN_MATCHING.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Algebraic Data Type, Sum Type, Product Type,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a language model data that can be "this OR that" (sum types) alongside "this AND that" (product types) in a way that is type-safe and composable?

Most real-world data is naturally described as alternatives: a shape is a circle **or** a rectangle; a payment is cash **or** credit card; a tree node is a leaf **or** a branch with children. Without language support, programmers encode alternatives using fragile mechanisms — runtime cast errors, inheritance hierarchies, or manual tag management.

With TRAITS (EDR-019) providing sealed trait hierarchies and PATTERN_MATCHING (EDR-025) providing exhaustive structural matching, the foundation for ADTs exists. The core problem: **data that takes one of several known forms needs a unified declaration mechanism** that is more concise than manual sealed trait + variant type declarations, with automatic discriminant generation and compiler-enforced exhaustiveness.

Additionally, ADTs subsume the need for a dedicated enum construct — payload-free variants (`type Color = Red | Green | Blue`) serve the simple enum use case with compiler-enforced exhaustiveness.

## Principles

1. **One sum-type mechanism** — Orthon has exactly one mechanism for modelling "one of several" types: Algebraic Data Types. No separate enum construct.
2. **Exhaustiveness** — Pattern matching on an ADT must cover all variants. The compiler enforces this.
3. **Variant fields are named** — Fields within a variant are named by default (readable, enables copy-with-modify). Positional shorthand is available for single-field variants.
4. **Sealed by default** — The variant set is closed. Adding a variant is a type declaration change that produces compile-time errors at match sites.
5. **Recursive types** — ADTs support recursion (trees, lists) with compiler-enforced termination checks (size bounds, indirection via reference).
6. **`@derive` compatibility** — Structural derives (`Show`, `Eq`, `Clone`, `Hash`) apply to ADT declarations via the existing derive mechanism (EDR-029).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Definition Policy | Governs how ADTs are declared (syntax, variant naming, field types) |
| Exhaustiveness Policy | Determines how strictly the compiler enforces exhaustive matching on ADT variants |
| Memory Layout Policy | Controls how ADTs are laid out in memory (tagged union, niche optimisation, flat layout) |
| Recursion Policy | Governs recursive type definitions (termination checking, size bounds) |
| Derivation Policy | Controls automatic trait implementation generation for ADT variants |

## Model (What)

Algebraic Data Types combine **product types** ("and") and **sum types** ("or") into a single declaration. The `type` keyword introduces a named ADT with variants separated by `|`.

```orthon
# Sum type — Shape is Circle OR Rectangle OR Triangle
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)
           | Triangle(a: Float, b: Float, c: Float)

# Simple enum-style ADT (payload-free variants)
type Color = Red | Green | Blue

# Product type — a simple record with named fields
type Point(x: Int, y: Int)

# Recursive ADT — binary tree
type Tree<T> = Empty
             | Node(value: T, left: Tree<T>, right: Tree<T>)

# Generic ADT
type Option<T> = Some(value: T) | None
type Result<T, E> = Ok(value: T) | Error(err: E)
```

### Pattern matching with ADTs

Each ADT variant can be destructured in a `match` expression:

```orthon
area = fun (s: Shape) -> Float
    match s:
        Circle(r)          -> pi * r * r
        Rectangle(w, h)    -> w * h
        Triangle(a, b, c)  ->
            p = (a + b + c) / 2
            sqrt(p * (p - a) * (p - b) * (p - c))
```

The compiler guarantees all variants are covered. Unreachable patterns are flagged.

### ADT composition

ADTs compose naturally — a variant field can itself be an ADT:

```orthon
type Widget = Button(label: String, onClick: Action)
            | TextInput(placeholder: String, value: String)
            | Panel(children: List<Widget>)
```

## Default Strategy

ADTs use a **tagged union** memory layout: a discriminant (tag) followed by the variant's fields. The compiler optimises layout by packing the tag into padding bytes where possible (niche optimisation, like Rust's NonNull for `Option<&T>`). Pattern matching compiles to a jump table on the tag.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Sealed trait hierarchy | ADT declaration desugars to a sealed trait + variant types per EDR-019. Explicit but more verbose. |
| Flat layout (no tag) | For ADTs where variants are distinguishable by field types (e.g., `type ID = StringId(String) | IntId(Int)`), the tag may be elided. |
| Niche optimisation | Tag packed into unused bit patterns of a variant field (e.g., `None` stored as null pointer in `Option<&T>`). |
| Boxed recursive variants | Recursive variants use heap-allocated indirection (boxed) to break the size-recursion cycle. |

## Open Questions

1. Should ADTs support method-like functions directly on variants (Rust `impl` block syntax), or should all behaviour go through traits?
2. Should the compiler support automatic `@derive` for all structural traits when an ADT is declared (opt-out rather than opt-in)?
3. How do recursive ADTs interact with the Allocation Policy — minimum size bounds for arena allocation?

## Decision History

- **EDR-039 (2026-07-27):** ADTs accepted as Language feature. ENUM_ALTERNATIVES folded into this decision — ADTs subsume dedicated enums. Variant fields named by default.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] Other: `how/concepts/research/DATA_MODEL.md`, `what/concepts/TRAITS.md`, `what/concepts/PATTERN_MATCHING.md`
