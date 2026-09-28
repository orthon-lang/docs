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

### Behaviour: traits over data, not methods on data

`type` declares a data shape only. Behaviour attaches externally — through
traits (EDR-019) and separate `impl` blocks (EDR-039 §4). ADT variants never
carry inherent methods. Two canonical shapes follow from this rule.

**Path A — one closed ADT, one implementation, dispatch via `match`.** Use this
when the variants form a closed family and behaviour is defined for the family
as a whole:

```orthon
type Shape = Circle(radius: Float)
           | Rectangle(w: Float, h: Float)

trait Area
    fun area(self) -> Float

impl Area for Shape
    fun area(self) -> Float
        match self:
            Circle(r)        -> pi * r * r
            Rectangle(w, h)  -> w * h
```

Adding a variant (`Triangle`) makes the `match` non-exhaustive — a compile-time
error at every call site that consumes `Shape` by matching.

**Path B — independent named types sharing a behavioural contract.** Use this
when `Circle` and `Rectangle` are genuinely independent types that happen to
share behaviour. Each is a bare product type; each gets its own `impl` block:

```orthon
type Circle(radius: Float)      # bare record — no methods
type Rectangle(w: Float, h: Float)

trait Area
    fun area(self) -> Float

impl Area for Circle            # separate block, outside the declaration
    fun area(self) -> Float
        return pi * self.radius * self.radius

impl Area for Rectangle
    fun area(self) -> Float
        return self.w * self.h
```

Polymorphism over Path B types is trait-based: statically via generics
(`fun total<Area as T>(List<T> shapes)`, monomorphised — the default), or
dynamically via `dyn Area` (opt-in vtable).

In both paths the `impl` block is **external** to the `type` declaration. The
paths answer different questions: Path A models "data is one of several known
forms" (closed, exhaustive); Path B models "independent types share behaviour"
(trait polymorphism). Neither attaches implementation inside the type — that is
the Java/C# class-with-methods model, which Orthon rejects.

#### The Expression Problem

The data/behaviour split is a solution to the **Expression Problem** — the
classic language-design problem of extending a system in two dimensions without
modifying existing code: adding *new types* and adding *new operations* over
them.

- **Path A (ADT + match) optimises adding operations.** A new function (e.g.,
  `perimeter`) is a new `match` over existing variants — no existing code
  changes. Adding a *new variant* (`Triangle`) forces changes at every match
  site; exhaustiveness turns this into a compile-time error.
- **Path B (traits) optimises adding types.** A new type (`Triangle`) is a new
  record plus a new `impl Area for Triangle` — no existing code changes. Adding
  a *new operation* is a new trait plus impls for the types that need it.

Path B therefore inverts the responsibility for defining operations: behaviour
is not owned by the type, so both extension dimensions stay open without
touching existing code.

#### Scientific Grounding

- **The Expression Problem** was formulated by Philip Wadler: a system should
  be extensible along two axes — new data variants and new operations — without
  modifying existing code. Traits / type classes are the accepted elegant
  solution.
- **Traits as composable units of behaviour** were formalised in *"Traits:
  Composable Units of Behaviour"* (Schärli, Ducasse, Nierstrasz, Black, 2002) —
  a formal model of traits as groups of methods that act as building blocks for
  classes, overcoming the flaws of multiple inheritance.
- Orthon inherits this lineage: TRAITS (EDR-019) adopts the Rust-style trait
  model (explicit `impl`, coherence / orphan rule, static dispatch by default),
  which itself descends from Haskell type classes and the traits research of
  the 2000s.

#### Comparison with Other Languages

| Language | Model | How it compares |
|---|---|---|
| Rust | `enum` ADT + separate `impl`/`trait` | Closest match: variants carry data; behaviour lives in external trait and impl blocks. |
| Haskell / OCaml | `data` + typeclasses | Behaviour separate from data; exhaustiveness enforced. Orthon uses explicit `impl` (Rust-style) rather than implicit typeclass instances. |
| Swift | Structs + protocols + extensions | Protocols with extensions attach behaviour to existing types externally; extensions can also add methods without a protocol. |
| Scala | Case classes + traits / type classes | Traits and type classes are the standard solution to the Expression Problem; Scala 3 makes the approach more natural. |
| Go | Structs + structural interfaces | Interfaces are satisfied structurally (implicitly) by method shape — no explicit `impl`; Orthon requires explicit satisfaction. |
| C++ | Types + Concepts (C++20) + templates | Static polymorphism via templates and Concepts; dynamic polymorphism via inheritance and virtual functions. |
| Kotlin | Sealed classes + extension functions | Sealed hierarchy with behaviour outside the class. Orthon makes the closed set the default rather than opt-in. |
| TypeScript | Discriminated unions + functions | Untagged and erased at runtime; no exhaustiveness. Orthon keeps a tag and compile-time exhaustiveness. |
| Java / C# | Classes bind methods to data | The rejected alternative: behaviour lives inside the type, enabling inheritance and the fragile-base-class problem. |
| Python | Dynamic classes + ABC / Protocols (typing) | Dynamic languages can add methods at runtime (monkey patching); static contracts come from ABCs or Protocols rather than a compiler-enforced trait model. |

#### Trade-offs

**Advantages.**

- **Behaviour stays orthogonal to data.** A type is a pure value; contracts can
  be added or combined without inheritance and without editing the type — no
  fragile base classes, no hierarchy-forced method collisions.
- **One sum-type mechanism.** ADT subsumes enums ("One concept, one syntax").
- **Compile-time exhaustiveness** over closed variant sets.
- **LLM-readiness.** External `impl` blocks and sealed variant sets give a code
  generator a single, unambiguous contract per type; `@derive` removes
  boilerplate the generator would otherwise emit.

**Disadvantages.**

- **Verbosity.** Behaviour requires a separate `impl` block; the reader must
  cross-reference the `type` declaration and its `impl` blocks. This is more
  ceremony than a class-with-methods.
- **Unfamiliarity.** Programmers coming from OOP expect methods on the type; the
  data/behaviour split is a mental shift.
- **No inherent methods** on variants — a free function or a trait impl is always
  needed to attach behaviour.

Mitigations: trait default methods (Template Method), blanket impls, and
`@derive` reduce the ceremony; see [TRAITS](TRAITS.md).

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

1. ~~Should ADTs support method-like functions directly on variants (Rust `impl` block syntax), or should all behaviour go through traits?~~ **Resolved by EDR-039 §4** — all behaviour goes through traits; `impl` blocks are separate; variants carry no inherent methods. See [Behaviour: traits over data](#behaviour-traits-over-data-not-methods-on-data).
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
