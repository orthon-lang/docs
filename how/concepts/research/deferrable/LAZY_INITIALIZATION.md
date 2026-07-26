# Lazy Initialization

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

How does a programmer defer the initialization of a value until it is first accessed, without writing thread-safe caching logic?

Common scenarios:

- **Expensive computation** — A large data structure that may not be needed in every execution path.
- **Conditional resources** — A database connection or network client that is only used in specific flows.
- **Circular dependency avoidance** — Two objects that reference each other, where eager initialization creates a chicken-and-egg problem.
- **Late-binding** — A value that depends on runtime conditions not available at construction time.

The manual Java solution is a well-known pattern:

```java
private volatile ExpensiveObject cached;
public ExpensiveObject getExpensive() {
    if (cached == null) {
        synchronized (this) {
            if (cached == null) {
                cached = computeExpensive();
            }
        }
    }
    return cached;
}
```

This is ~15 lines of identical boilerplate for every lazy field. Thread safety is easy to get wrong.

The core problem: **should Orthon have built-in language constructs for deferred initialization, or should this remain a library concern?**

A related problem: **late-initialized non-nullable values** — a field that cannot be initialized at construction time but is guaranteed to be set before first use (dependency injection, test setup). Kotlin's `lateinit` solves this by telling the compiler: "trust me, this will be set before it's read."

## Examples

| Language | Lazy initialization | lateinit / deferred non-null |
|---|---|---|
| **Kotlin** | `val x: Type by lazy { compute() }` | `lateinit var x: Type` |
| **Swift** | `lazy var x: Type = compute()` | Implicit via `!` (implicitly unwrapped optionals) |
| **C#** | `Lazy<Type>` wrapper class | `[NotNull]` attribute (no language-level) |
| **Rust** | `lazy_static!`, `OnceCell`, `OnceLock` | `UnsafeCell` / `MaybeUninit` |
| **Java** | Manual double-checked locking | Manual (or `@Nullable` + contract) |
| **Scala** | `lazy val x: Type = compute()` | Implicit via abstract `val` override |

## Implications for Orthon

1. **Lazy property as a language-level or stdlib construct** — A property can be declared lazy, meaning its initializer is not evaluated until the first access:

   ```
   config: Config by lazy { load_config() }
   ```

   If property delegation exists (see `DELEGATION.md`), `lazy` can be a library-provided delegate. If delegation does not exist, lazy should be a language keyword or built-in modifier.

2. **Thread safety by default** — The built-in `lazy` delegate must be thread-safe (initializer runs at most once, even under concurrent access). A non-thread-safe variant (`lazy(NONE)`) may be available for single-threaded contexts.

3. **`lateinit` for deferred non-null** — A mutable field can be declared with deferred initialization, suppressing compile-time definite-assignment checks:

   ```
   lateinit var logger: Logger   // will be set before first use
   ```

   Reading before assignment is a runtime error (checked). The compiler tracks assignments and can warn about potential read-before-write paths where possible.

4. **No implicit lazy** — Lazy evaluation must be syntactically visible. The programmer should never discover that a "simple field access" triggers an expensive computation.

5. **Interaction with immutability** — A `lazy val` is externally immutable (read-only) but internally mutable (caches the computed value). This is safe because the mutation is internal, synchronized, and idempotent.

## Open Questions

1. Should `lazy` be a built-in modifier (`lazy val`) or a delegate (`by lazy { }`)? The answer depends on whether `DELEGATION.md` exists in the language.
2. Should there be a synchronisation mode parameter for `lazy` (thread-safe, none, publication-only)?
3. For `lateinit`, should the compiler generate debug checks (throw on read-before-write) that can be disabled for release builds?
4. Can `lateinit` apply to properties of any type, or only reference types (non-primitives)?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `PROPERTIES.md`
- [ ] `DELEGATION.md`
- [ ] `OBJECT_INITIALIZATION.md`
- [ ] `MUTABILITY.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
