# Collections Functional API

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

How does a language provide declarative, chainable data transformations over collections — `map`, `filter`, `reduce`, `flatMap` — without requiring a separate streaming library or producing excessive intermediate allocations?

Java solved this with the **Streams API** (`java.util.stream`), which provides a fluent chain of functional operations:

```java
list.stream()
    .filter(x -> x > 0)
    .map(x -> x * 2)
    .collect(Collectors.toList());
```

This works but has problems:
- Requires converting the collection to a stream first (`.stream()`).
- Requires collecting back (`.collect(...)`). Intermediate state leaks into API.
- Parallel streams are a confusing second path.
- Primitive streams (`IntStream`, `LongStream`, `DoubleStream`) are separate types.

Kotlin solved this by making functional operations **extension functions on collection types** directly:

```kotlin
list.filter { it > 0 }
    .map { it * 2 }
```

No `.stream()` or `.collect()` — the operations return `List` directly. The implementation uses inline functions (no lambda object allocation) and `Sequence` for lazy evaluation.

The core problem: **should Orthon have built-in functional operations on collections as a core language feature, or delegate this to the standard library?**

## Examples

| Language | API surface | Lazy variant | Requires wrapper? | Inline/optimized? |
|---|---|---|---|---|
| **Java** | Streams API | `.stream()` | Yes — `.stream()` / `.collect()` | No — lambda objects |
| **Kotlin** | Extension functions on `Iterable` | `.asSequence()` | No — direct on collections | Yes — `inline` functions |
| **Rust** | `Iterator` trait methods | Lazy by default | No — `iter()` returns iterator | Yes — zero-cost abstractions |
| **C#** | LINQ (`IEnumerable<T>`) | Lazy by default | No | No — delegate objects |
| **Python** | Built-in `map()`, `filter()`, list comprehensions | `itertools` / generators | No | No — function call overhead |
| **Go** | No built-in — manual loops or external libs | N/A | N/A | N/A |

## Implications for Orthon

1. **Functional operations as extension functions on collection types** — `map`, `filter`, `reduce`, `flatMap`, `fold`, `zip`, `partition`, `any`, `all`, `none` are extension functions on `Collection[T]` (see `EXTENSION_FUNCTIONS.md`):

   ```
   result = list
       .filter { it > 0 }
       .map { it * 2 }
       .sorted()
   ```

2. **No `.stream()` / `.collect()` ceremony** — Operations return a concrete collection type directly. The runtime chooses between eager and lazy internally based on the operation chain.

3. **Eager by default, lazy via explicit `Sequence`** — `map`/`filter` chains evaluate eagerly by default (producing intermediate collections). For large data or long chains, `asSequence()` converts to lazy evaluation:

   ```
   result = list
       .asSequence()
       .filter { it > 0 }
       .map { it * 2 }
       .toList()
   ```

4. **Inline-friendly** — Lambda parameters should be inline-expandable to avoid allocation overhead. This requires function inlining support (see `FUNCTIONS.md`).

5. **No separate primitive-stream types** — Generics with trait bounds cover both primitive and reference element types uniformly.

6. **Set and Map operations** — The same functional API applies to `Set` and `Map` types with appropriate operations (e.g., `mapKeys`, `mapValues` for dictionaries).

## Open Questions

1. Should functional operations be eager or lazy by default? Kotlin (eager) and Rust (lazy) chose differently.
2. Should the language provide **parallel** variants of functional operations (`.parallelMap()`), or leave parallelism to explicit constructs?
3. How do functional operations interact with the mutability divide (`DYNAMIC_COLLECTIONS.md`)? Does `map` on a `List` return a new `List`, or mutate in place?
4. Should the standard library guarantee that common chains (`filter` + `map` + `take`) compile to a single fused loop?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `DYNAMIC_COLLECTIONS.md`
- [ ] `EXTENSION_FUNCTIONS.md`
- [ ] `FUNCTIONS.md`
- [ ] `MUTABILITY.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
