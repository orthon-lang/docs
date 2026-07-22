# Nested Classes

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

When a type is defined inside another type, does it automatically have access to the outer type's instance — or is that opt-in?

In Java, **every inner class** (a class defined inside another class) implicitly holds a reference to the enclosing instance unless it is declared `static`:

```java
class Outer {
    class Inner {
        // has implicit reference to Outer.this
    }
    static class Nested {
        // no reference to Outer instance
    }
}
```

This default (inner = has enclosing reference) causes problems:

- **Memory leaks** — Non-static inner classes hold a reference to the outer instance. If the inner class outlives the outer (e.g., an anonymous `Runnable` posted to a global thread pool), the outer instance cannot be garbage collected.
- **Serialization surprises** — Non-static inner classes cannot be serialized reliably because the compiler-generated reference to the outer instance is implicit.
- **Accidental coupling** — The inner class has unrestricted access to the outer's private members, breaking encapsulation boundaries.

Kotlin reversed the default: **nested classes are static by default** (no enclosing reference). The `inner` keyword is required to create a class that holds an enclosing reference.

The core problem: **should Orthon's default be nested-without-enclosing-reference (like Kotlin, Rust) or nested-with-enclosing-reference (like Java)?** Or should Orthon not support nesting at all?

## Examples

| Language | Default | Inner syntax | Enclosing ref | Use case for inner |
|---|---|---|---|---|
| **Java** | Non-static (inner) | `inner` is default | Automatic | Callback adapters (pre-lambda) |
| **Kotlin** | Static (nested) | `inner class` | Explicit `this@Outer` | Rare — mostly for scoping |
| **Rust** | No nesting in type system | Modules are separate | N/A | Modules, not inner classes |
| **Swift** | Nested type (static) | No `inner` concept | N/A | Namespace-like nesting |
| **Python** | Inner class has no auto `self` | Not idiomatic | Manual `self.outer` | Rare |

## Implications for Orthon

1. **Nested = static by default** — A type declared inside another type is a nested type: it exists in the outer type's namespace but has no reference to any instance of the outer type:

   ```
   struct Outer:
       data: Int

       struct Nested:           // no reference to Outer instance
           value: String
   ```

2. **`inner` for enclosing reference** — If a nested type needs access to an enclosing instance, it must be declared `inner`:

   ```
   struct Outer:
       data: Int

       inner struct Inner:      // has implicit reference to Outer instance
           fun get_data() -> Int: return outer.data
   ```

3. **Explicit outer reference** — Within an `inner` type, the reference to the enclosing instance is accessed via `outer` (or `outer.Outer`), never implicitly through name resolution.

4. **No memory leak risk** — Because `inner` is opt-in and explicit, no type accidentally captures an enclosing instance reference. The default is safe for memory management.

5. **Anonymous inner classes are unnecessary** — If Orthon has first-class functions (see `FUNCTIONS.md`) and lambdas, anonymous inner classes (Java's `new Interface() { ... }`) have no purpose. Lambdas capture only what they need, not the entire enclosing instance.

## Open Questions

1. Does Orthon need nested types at all, or should all types be top-level + modules (see `TOP_LEVEL_DECLARATIONS.md`)?
2. Should `inner` types be restricted to certain visibility levels (e.g., always `private` to the enclosing type)?
3. How does nesting interact with generics? Can a nested type access the outer type's type parameters?
4. If Orthon has no class/instance distinction (pure data model), does `inner` even have meaning?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `TOP_LEVEL_DECLARATIONS.md`
- [ ] `VISIBILITY_AND_ENCAPSULATION.md`
- [ ] `FUNCTIONS.md`
- [ ] `what/GLOSSARY.md`
