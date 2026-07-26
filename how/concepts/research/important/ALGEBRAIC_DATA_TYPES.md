# Algebraic Data Types

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

How does a language model data that can be "this OR that" (sum types) alongside "this AND that" (product types) in a way that is type-safe and composable?

Most real-world data is naturally described as alternatives:
- A shape is a circle **or** a rectangle **or** a triangle.
- A payment is cash **or** credit card **or** bank transfer.
- A tree node is a leaf **or** a branch with children.

Without language support, programmers encode alternatives using:
- **Enums with casts** (C, Java) — runtime cast errors, no compiler checking.
- **Inheritance** (OOP) — open for extension, no exhaustiveness, accidental complexity.
- **Tagged unions** (manual) — error-prone tag management.

The core problem: **data that takes one of several known forms needs language-level support** so the compiler can verify that all forms are handled, every time.

## Principles

1. **Sum types are first-class** — A type is defined as a choice between named variants. Each variant carries its own fields.
2. **Exhaustiveness** — Pattern matching on an ADT must cover all variants. The compiler enforces this.
3. **No null variants** — ADTs use `Option` for optional fields, not nullable fields.
4. **Product types as records** — A variant with multiple fields is a product type (tuples / records). No separate struct syntax needed.
5. **Recursive types** — ADTs support recursion (trees, lists) with compiler-enforced termination checks.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Definition Policy | Governs how ADTs are declared (syntax, variant naming, field types) |
| Pattern Exhaustiveness Policy | Determines how strictly the compiler enforces exhaustive matching |
| Memory Layout Policy | Controls how ADTs are laid out in memory (tagged union, nested, flat) |
| Recursion Policy | Governs recursive type definitions (termination checking, size bounds) |

## Model (What)

Algebraic Data Types combine **product types** ("and") and **sum types** ("or") into a single declaration. A `type` keyword introduces a named ADT with variants separated by `|`.

```orthon
# Sum type — Shape is Circle OR Rectangle OR Triangle
type Shape = Circle(radius: Float)
           | Rectangle(width: Float, height: Float)
           | Triangle(a: Float, b: Float, c: Float)

# Product type — a simple record
type Point(x: Int, y: Int)

# Recursive ADT — binary tree
type Tree<T> = Empty
             | Node(value: T, left: Tree<T>, right: Tree<T>)

# Generic ADT
type Option<T> = Some(value: T)
               | None

type Result<T, E> = Ok(value: T)
                  | Error(err: E)
```

Key features:
- **`type Name = Variant(fields) | Variant(fields)`** — ADT declaration.
- **Named variants** — each variant is a constructor with optional named fields.
- **Pattern matching** — `match` exhaustively decomposes ADTs.
- **Generics** — ADTs can be generic (`Tree<T>`).

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

### Nesting and composition

ADTs compose naturally — a variant field can itself be an ADT:

```orthon
type Widget = Button(label: String, onClick: Action)
            | TextInput(placeholder: String, value: String)
            | Panel(children: List<Widget>)        # recursive composition
```

## Default Strategy

ADTs use a **tagged union** memory layout: a discriminant (tag) followed by the variant's fields. The compiler optimises layout by packing the tag into padding bytes where possible (niche optimisation, like Rust's `NonZero`). Pattern matching compiles to a jump table on the tag.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Sealed classes (Kotlin) | Class hierarchy with restricted inheritance. Exhaustiveness checked by compiler. |
| Enum variants (Rust) | ADTs as `enum` with named or positional fields. Same semantics. |
| Tagged unions (C/C++) | Manual — programmer manages tag and union. No compiler exhaustiveness. |
| Visitor pattern (OOP) | Double dispatch to simulate sum types. Boilerplate-heavy, open to new operations but closed to new variants. |
| Dynamic dispatch (Python) | `isinstance` chains. No exhaustiveness, runtime errors. |

## Open Questions

1. Should ADTs support methods directly, or should functions be separate (like Rust `impl`)?
2. Should variant fields be positional, named, or both?
3. How do ADTs interact with the trait system? Can variants implement traits individually?
4. Should the compiler support automatic derivation of `Show`, `Eq`, `Clone` for ADTs? (See `@derive` in `METAOBJECTS.md`.)

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] Other: `how/concepts/research/DATA_MODEL.md`, `how/concepts/research/PATTERN_MATCHING.md`, `how/concepts/research/GENERICS.md`
