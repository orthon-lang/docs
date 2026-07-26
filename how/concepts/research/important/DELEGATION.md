# Delegation

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

How does a type reuse behaviour from another type without inheritance — and how does a property get custom behaviour (lazy, observable, validated) without boilerplate?

These are two distinct but related problems:

1. **Class delegation** — A type implements an interface by forwarding all calls to a contained instance. In Java, this requires hand-writing every forwarding method. For interfaces with many methods (`List`, `Map`, `Iterator`), this is hundreds of lines of boilerplate.
2. **Property delegation** — A property's getter/setter behaviour is delegated to a helper object (lazy initialization, observable change notification, validated assignment, stored in a map). In Java, each property requires its own boilerplate implementation of these patterns.

Kotlin solves both with the `by` keyword:
- **Class delegation**: `class MyList<T>(inner: List<T>) : List<T> by inner` — all `List` methods forward to `inner`.
- **Property delegation**: `val name: String by lazy { computeName() }` — the property getter delegates to `lazy`.

The core problem: **should Orthon have built-in delegation as a language construct, or should these patterns remain in userspace (manual forwarding, library helpers)?**

## Examples

### Class Delegation

| Language | Mechanism | Boilerplate per method |
|---|---|---|
| **Java** | Manual forwarding | ~3 lines per method |
| **Kotlin** | `by` keyword | 0 lines — compiler generates |
| **Scala** | `trait` composition | Varies |
| **Rust** | `delegate` crate (external) | 0 lines with macro |
| **Go** | Struct embedding (implicit promotion) | 0 lines — but implicit, not explicit |

### Property Delegation

| Language | Lazy | Observable | Validation | Map-backed |
|---|---|---|---|---|
| **Kotlin** | `by lazy { }` | `Delegates.observable()` | `Delegates.vetoable()` | `by map` |
| **Swift** | `lazy var` | `didSet`/`willSet` | Custom in `didSet` | N/A |
| **C#** | `Lazy<T>` wrapper | Manual | Manual in setter | Manual |
| **Python** | `@property` + manual caching | `@property` + manual | `@property` + manual | `__getattr__` |

## Implications for Orthon

1. **Class delegation as language construct** — A type can delegate an interface implementation to a contained field:

   ```
   class LoggingList[T](inner: List[T]) : List[T] by inner
   ```

   The compiler generates forwarding methods for all `List` methods to `inner`. Selective override is still possible by defining a specific method explicitly.

2. **Property delegation as language construct** — A property can delegate its getter (and optionally setter) to a delegate object:

   ```
   name: String by lazy { loadName() }
   counter: Int by observable(0) { old, new -> log("$old -> $new") }
   data: String by map(configStore, "db.url")
   ```

3. **Delegate protocol** — Any type implementing a standard `Get` (and optionally `Set`) protocol can be a property delegate. This is a library interface, not a compiler-intrinsic:

   ```
   trait Get[T]:
       fun get(thisRef: Any, prop: PropertyMetadata) -> T

   trait Set[T]:
       fun set(thisRef: Any, prop: PropertyMetadata, value: T)
   ```

4. **No inheritance required** — Delegation replaces many uses of implementation inheritance (decorator pattern, proxy pattern, adapter pattern) with explicit, composable forwarding.

## Open Questions

1. Should class delegation require an explicit `by` clause, or can it be implicit via composition (Go-style automatic method promotion)?
2. Should property delegation support **chaining** (delegate whose delegate is another delegate)?
3. How does delegation interact with `PROPERTIES.md` and visibility? If a delegated interface has methods with varying visibility, how are they forwarded?
4. Should the language provide a standard library of delegates (`lazy`, `observable`, `vetoable`, `map`) or leave this to third-party libraries?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `PROPERTIES.md`
- [ ] `COMPOSITION_OVER_INHERITANCE.md`
- [ ] `LAZY_INITIALIZATION.md`
- [ ] `FUNCTIONS.md`
- [ ] `what/GLOSSARY.md`
