# Slots

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

How does a type declare a fixed, known set of attributes — restricting which fields can exist at runtime — and what benefits does that bring in terms of memory efficiency, safety, and programmer intent?

Languages with dynamic attribute models (Python, JavaScript) allow arbitrary attribute assignment on any object at any time:

```python
class User:
    pass

u = User()
u.name = "Alice"       # OK
u.hat_size = "large"   # OK — no restriction whatsoever
```

This has two costs:

1. **Memory overhead** — Each object carries a dictionary (or similar dynamic storage) to hold arbitrary attributes, even when the set of attributes is known at declaration time. Per-object dictionaries consume 50–200 bytes of overhead per instance.
2. **Accidental attribute surface** — A typo like `user.namme = "Bob"` silently creates a new attribute instead of signalling an error. There is no way for the type author to say "these are the only fields this type has."

Python's `__slots__` mechanism addresses this: a class declaring `__slots__ = ("name", "age")` restricts instances to exactly those attributes. The runtime replaces the per-object dictionary with a fixed-size array of slot descriptors, saving memory and catching accidental attribute creation at runtime.

| Language | Fixed-attribute mechanism | Restriction timing | Memory benefit |
|---|---|---|---|
| **Python** | `__slots__` as class attribute | Runtime — `AttributeError` on unknown attr | Per-object dict eliminated |
| **Rust** | Struct fields are always fixed by type system | Compile-time | Optimal — no indirection |
| **Java** | Fields are always fixed by class definition | Compile-time | Optimal — fixed object layout |
| **JavaScript** | No built-in mechanism; `Object.preventExtensions()` / TypeScript | Runtime / compile-time (TS) | No — still dynamic at runtime |
| **C#** | `struct` value types have fixed layout | Compile-time | Optimal (value type layout) |

The core problem: **should Orthon support a "closed" attribute mode where a type restricts its fields to a declared set, and should that be the default or an opt-in?**

## Principles

1. **Explicitness over implicitness** (from `DESIGN_PRINCIPLES.md`) — If a type's set of fields is known, that knowledge should be expressible in the type declaration. The programmer should not need to infer from usage which fields are legitimate.
2. **Safety first** — Accidental attribute creation (typos, refactoring drift) must be detectable, preferably at compile time.
3. **Memory model transparency** — The programmer should understand the memory cost of a type. A fixed-field type has a predictable, compact layout; a dynamic one has overhead.
4. **Progressive disclosure** — The language should allow starting with loose, dynamic attributes and tightening to fixed slots as the design matures, without rewriting code.

## Examples

### Python `__slots__`

```python
class User:
    __slots__ = ("name", "age")

    def __init__(self, name, age):
        self.name = name
        self.age = age

# u = User("Alice", 30)     # OK
# u.email = "a@b.com"       # AttributeError: 'User' object has no attribute 'email'
```

### Rust (fixed by default)

```rust
struct User {
    name: String,
    age: u32,
}
// Accessing user.nonexistent — compile error
```

### TypeScript `interface`

```typescript
interface User {
    name: string;
    age: number;
}
// const u: User = { name: "Alice", age: 30, extra: true };
// → Type '{ name: string; age: number; extra: boolean; }' is not assignable to type 'User'.
```

## Implications for Orthon

1. **Fixed fields as the default** — Orthon's type system should enforce a fixed set of attributes by default for all declared types. Every field is explicitly listed in the type declaration; accessing an undeclared field is a compile-time error.
2. **Dynamic attribute extension as an opt-in** — If a type genuinely needs runtime-expandable attributes, that should be an explicit opt-in (e.g., a `dynamic` modifier on the type or a dedicated "open record" construct). The presence of dynamic attributes should be visible in the type declaration.
3. **Memory layout predictability** — Fixed-field types enable ABI-stable layout (like C structs), which is essential for FFI, serialization, and embedded targets. Orthon can guarantee a compact representation for fixed types.
4. **Relationship with properties** — In a property-based type system (see [`PROPERTIES.md`](PROPERTIES.md)), every declared property is inherently a slot. The slot concept collapses into "a property with a backing field." No separate `__slots__`-like declaration is needed — declaring a property already reserves the slot.
5. **No inheritance complications** — Python's `__slots__` interacts poorly with multiple inheritance (each class must declare its own slots; slots from parent classes are not inherited). Orthon's slot model should compose cleanly with its trait/typeclass mechanism — traits can declare required fields, and implementing a trait adds those slots to the type.

## Open Questions

- Should Orthon allow anonymous/structural types with dynamic fields (like TypeScript's `Record<string, T>` or `{ [key: string]: any }`)?
- How do slots interact with the library boundary? Can a library type expose additional slots in a controlled way for subclassing?
- Should serialization frameworks be able to bypass the slot restriction via an explicit opt-in mechanism?
- Is there a case where a closed type should still allow a controlled escape hatch (e.g., a `rest` slot like Python's `**kwargs` analogues)?
