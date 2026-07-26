# Objects and Singletons

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

How does a language model a type that has exactly one instance — a singleton — without relying on a manual design pattern?

Languages before Kotlin addressed singleton instances through one of:

1. **Manual pattern (Java)** — `private` constructor, `static getInstance()` method, `volatile` field, double-checked locking. Every singleton requires ~15 lines of boilerplate that is identical across all singletons. Errors (reflection, serialization, multiple classloaders) are easy to miss.
2. **Global mutable state (C, Python, Go)** — A package-level variable that acts as a singleton. Simple, but no control over initialization order, no laziness guarantees, and no interface implementation.

Kotlin introduced the `object` keyword as a language-level singleton declaration — a class that has exactly one instance, created lazily and safely by the runtime. This eliminated the Singleton design pattern as boilerplate.

A related but distinct problem: **how does a language represent things that are logically tied to a type but are not instance members** — factory methods, constants, shared registry entries? Java uses `static` members; Kotlin uses `companion object` inside a class (a named singleton nested in the class).

The core problem: **should Orthon have a dedicated singleton declaration form, or should singletons be expressed through other mechanisms (top-level values, module-level state, type-level constants)?**

## Examples

| Language | Singleton mechanism | Lazy init | Interface impl | Static members |
|---|---|---|---|---|
| **Java** | Manual pattern (boilerplate) | Manual | Yes (class implements interface) | `static` keyword |
| **Kotlin** | `object` keyword | Built-in | Yes (`object : Interface`) | `companion object` |
| **Scala** | `object` keyword | Built-in | Yes | `object` (replaces static entirely) |
| **Rust** | No built-in singleton; `lazy_static!` or `OnceCell` | External crate | N/A | No static members per se |
| **Go** | Package-level `var` | Manual with `sync.Once` | Via interface satisfaction | No static members |
| **Swift** | `static let shared = Type()` | `lazy` by nature | Yes | `static` keyword |

## Implications for Orthon

1. **Singleton as language construct** — A dedicated `object` declaration creates a type with exactly one lazily created instance. The compiler handles thread-safe initialization:

   ```
   object Config:
       version: String = "1.0"
       debug: Bool = false
   ```

   Equivalent to: a type `Config` with a single global instance `Config` accessed by name.

2. **Companion-like construct** — If Orthon has types with instance members, a companion object provides type-level members without the `static` keyword. Alternatively, if Orthon has no instance methods (pure data model), this concept is unnecessary.

3. **Anonymous objects** — Expressing "an instance of an anonymous type implementing interface X" without a named class declaration:

   ```
   comparator = object : Comparable[String]:
       compare(a, b) = a.length <=> b.length
   ```

4. **No `static` keyword** — Static members are replaced by objects. This eliminates the class/static member split that complicates Java's type system (static members do not participate in inheritance, cannot implement interfaces, cannot be overridden).

5. **Object vs. top-level value** — The distinction: an `object` is a type + instance (can implement interfaces, participate in the type system), while a top-level `let`/`val` is just a value (see `TOP_LEVEL_DECLARATIONS.md`).

## Open Questions

1. Do objects participate in inheritance? Can `object Config : Serializable` implement `Serializable`?
2. Can an object be passed by reference? If it implements an interface, can it be used where that interface is expected?
3. How is an object's initialization order determined relative to other objects (dependency ordering)?
4. If Orthon has no instance methods — only data + functions — is the `object` keyword redundant with top-level values and modules?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `TOP_LEVEL_DECLARATIONS.md`
- [ ] `COMPOSITION_OVER_INHERITANCE.md`
- [ ] `VISIBILITY_AND_ENCAPSULATION.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
