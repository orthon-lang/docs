# Struct as Value Type

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It captures the working model under discussion for Orthon's value type
> mechanism. Not yet validated through Concept Design Review or accepted via EDR.
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

What is the default type for data in Orthon — and what guarantees does it provide?

Most mainstream languages distinguish value types from reference types, but the
boundary varies:

| Language | Value type | Reference type | Default |
|---|---|---|---|
| **Java** | `int`, `boolean` (primitives) | `class`, arrays | Reference (everything is `new`) |
| **C#** | `struct` | `class` | Reference |
| **Swift** | `struct` | `class`, `actor` | **Value** (prefer struct) |
| **Kotlin** | Primitive (optimised) | `class`, `object` | Reference |
| **Rust** | Everything by default | `Box`, `Rc`, `Arc` | **Value** (ownership) |

Languages with value semantics by default (Swift, Rust) report fewer
aliasing-related bugs and simpler reasoning about state changes. The core
problem: **should Orthon default to value semantics, making reference types the
explicit opt-in?**

## Hypothesis

Orthon uses `struct` as the **default type for data**, with value semantics,
copy on assignment, and immutable-by-default fields. Reference types (`class`)
are opt-in, used only when identity or shared mutable state is required.

### Declaration

```
struct Point
    x: Int
    y: Int
```

### Value semantics

- Assignment copies the value: `let a = Point{x: 1, y: 2}; let b = a; b.x = 3`
  does not affect `a`.
- Equality is structural: `a == b` compares field-by-field.
- No identity: two structs with equal fields are indistinguishable.
- Passed by value to functions: `fn distance(p: Point)` receives a copy.

### Immutable by default

```
struct Point
    x: Int
    y: Int

let p = Point{x: 1, y: 2}
p.x = 3                    // COMPILE ERROR: immutable field

// Mutation requires explicit 'var' declaration
var q = Point{x: 1, y: 2}
q.x = 3                    // OK: mutable binding
```

### No methods beyond `func` (pure)

Structs can have `func` methods (pure, no side effects) but **not** `proc` methods
by default. This enforces the principle that structs are data, not behaviour:

```
struct Point
    x: Int
    y: Int

    func magnitude() -> Float
        return sqrt(x*x + y*y)
```

**Open question:** Should structs allow `proc` methods that mutate `self`? This
would require explicit `var self` or `mut self` parameter.

### Memory layout

- Stack allocation by default (escape analysis may promote to heap).
- No heap header, no reference count, no vtable.
- Layout is fixed and predictable (C-compatible if `#[repr(C)]`).
- No identity overhead: a struct is exactly its bytes.

### When to use `struct` vs `class`

| Situation | Use |
|---|---|
| Pure data without identity | `struct` |
| Configuration, DTOs, value objects | `struct` |
| Mathematical types (Point, Vector, Matrix) | `struct` |
| Shared mutable state | `class` |
| Identity semantics (graph nodes, entities) | `class` |
| Large data structures that are frequently copied | **Measure** — may need `class` |
| Concurrently accessed state | `class` with `act` |

### Interaction with traits

Structs can implement traits:

```
trait Printable
    fn format(self) -> String

struct Point
    implements Printable

    x: Int
    y: Int

    fn format(self) -> String
        return "Point({self.x}, {self.y})"
```

Since structs are value types, trait methods on structs receive `self` by value
(copy) or by reference (`&self` depending on borrowing model).

## Principles

1. **Value semantics by default** — `struct` is the default choice for data.
   Reference types (`class`) are explicit opt-in.

2. **Data without behaviour** — structs carry data; behaviour lives in functions
   or traits. No `proc` methods by default (open question).

3. **Immutability by default** — fields are read-only unless explicitly declared
   mutable. See [`FINAL_BY_DEFAULT.md`](FINAL_BY_DEFAULT.md).

4. **Predictable layout** — no hidden overhead, no identity, no GC header.
   Stack allocation when possible.

5. **Copy safety** — passing a `struct` to a concurrent context (`act` method)
   is naturally safe: the receiver gets a private copy.

## Comparison with alternatives

| Approach | Languages | Pros | Cons |
|---|---|---|---|
| **Reference by default** | Java, C# | Familiar, flexible | Aliasing bugs, heap pressure |
| **Value by default, opt-in ref** | Swift, Rust | Safe, predictable | Learning curve for ref users |
| **Everything is a value** | C, Zig | Simple, fast | No reference semantics |
| **Orthon (hypothesis)** | — | Value safety + explicit ref where needed | Two type constructors to learn |

## See also

- [`CLASS_WITH_ACT.md`](CLASS_WITH_ACT.md) — class reference type hypothesis
- [`ACTORS.md`](ACTORS.md) — concurrency isolation research
- [`VALUE_SEMANTICS.md`](VALUE_SEMANTICS.md) — value semantics research
- [`FINAL_BY_DEFAULT.md`](FINAL_BY_DEFAULT.md) — immutability by default
- [`DATA_MODEL.md`](DATA_MODEL.md) — Orthon's data model
- [`CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`](CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md)
  — composition patterns
