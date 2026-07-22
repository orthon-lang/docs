# Structural Typing

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

How does a language support polymorphism — writing code that works with multiple types — without forcing the programmer to declare explicit relationships between types?

Two approaches dominate:
- **Nominal typing** (Java interfaces, Rust traits) — a type must explicitly declare `implements Interface`. This makes intent clear but creates ceremony: every type that satisfies an interface must say so, even if the relationship is obvious.
- **Structural typing** (Go interfaces, TypeScript, OCaml objects) — a type satisfies an interface if it has the required methods, regardless of whether it explicitly declares the relationship.

Nominal typing tends toward:
- **Premature abstraction** — interfaces are designed before they have multiple implementations.
- **Ceremony** — adding a new implementation requires editing a separate file.
- **Rigidity** — types from different libraries cannot satisfy the same interface without wrapper types.

Structural typing offers: "if it quacks like a duck, it is a duck" — no prior arrangement needed.

## Principles

1. **Structure over declaration** — A type satisfies a trait/interface if it has the required methods and fields, regardless of explicit `impl` declarations.
2. **Explicit `impl` for opt-in** — Explicit `impl Trait for Type` is available for cases where structural matching is ambiguous or intentional disambiguation is needed.
3. **Static dispatch by default** — Structural resolution happens at compile time. Dynamic dispatch (`dyn Trait`) is opt-in.
4. **Derivation shorthand** — `@derive(Trait)` auto-generates trait implementations for well-known traits (`Show`, `Eq`, `Clone`).
5. **Compatibility with nominal generics** — Traits declared nominally (with explicit method signatures) coexist with structural matching. A type satisfies a trait structurally if no explicit `impl` exists and the structure matches.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Compatibility Policy | Determines whether type compatibility is nominal, structural, or hybrid |
| Dispatch Policy | Controls static (compile-time) vs dynamic (runtime vtable) dispatch |
| Trait Resolution Policy | Governs how the compiler resolves which trait a type satisfies |
| Derivation Policy | Controls automatic trait implementation generation (`@derive`) |

## Model (What)

The trait system uses **structural typing** as the default: a type satisfies a trait automatically if its structure (methods, fields, signatures) matches the trait definition.

```orthon
# Define a trait
trait Show:
    fun show(self) -> String

# No explicit impl needed — if a type has `show`, it satisfies Show
type Point(x: Int, y: Int)

# Auto-satisfaction: Point has show(self) structurally
fun format[T: Show](value: T) -> String
    value.show()

# Explicit impl (when needed, e.g., custom behaviour)
impl Show for Point:
    fun show(self) -> String
        "Point({self.x}, {self.y})"

# Derivation shorthand — compiler generates the impl
@derive(Show, Eq, Clone)
type User(name: String, age: Int)
```

Key features:
- **`trait Name { fun sig(self) -> Type }`** — trait definition.
- **Structural satisfaction** — if a type has a method matching the trait's signature, it satisfies the trait. No `impl` needed.
- **`impl Trait for Type`** — explicit implementation for custom behaviour or disambiguation.
- **`@derive(Trait1, Trait2)`** — compile-time code generation for well-known traits.
- **`[T: Trait]`** — generic constraint: T must satisfy Trait.
- **`dyn Trait`** — opt-in dynamic dispatch (vtable).

### Structural matching rules

The compiler checks that the type has:
1. All required methods with matching names.
2. Each method's signature is compatible (parameter types and return type).
3. All associated types (if any) are satisfiable.

```orthon
trait Comparable:
    fun compare(self, other: Self) -> Int

# Point satisfies Comparable structurally:
type Point(x: Int, y: Int)
# if Point has: fun compare(self, other: Point) -> Int

# But explicit impl is safer for disambiguation:
impl Comparable for Point:
    fun compare(self, other: Point) -> Int
        self.x == other.x && self.y == other.y
```

## Default Strategy

Structural typing with static dispatch. The compiler resolves trait satisfaction at the call site. Explicit `impl` overrides structural matching. `@derive` generates for `Show`, `Eq`, `Clone`, `Hash`, `Default` by default.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Nominal traits (Rust) | Explicit `impl Trait for Type` required. No structural matching. Clearer boundaries. |
| Go interfaces | Structural by default, but no generics / trait bounds. Less expressive constraints. |
| TypeScript structural types | Structural for object types. No trait definition — shapes are anonymous. |
| Haskell type classes | Nominal with instance declarations. Coherence enforced by orphan rules. |
| Python protocols | Structural with `typing.Protocol`. No compiler enforcement. |

## Open Questions

1. Conflict resolution: what if a type structurally matches two traits with the same method name but different signatures?
2. Should structural matching be opt-in per trait? (e.g., `trait Show { ... }` — structurally matched; `trait Serializable` — nominal only.)
3. How does structural typing interact with gradual typing? Can unannotated values satisfy traits?
4. Should `@derive` be extensible (custom derive macros) or fixed to built-in traits?

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
- [ ] Other: `how/concepts/research/GENERICS.md`, `how/concepts/research/METAOBJECTS.md`
