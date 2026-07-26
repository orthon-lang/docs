# Iterator Protocol

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It proposes an `Iterator` protocol separate from the `yield`-based
> generator mechanism. An iterator describes *how to consume* a sequence
> lazily; a generator describes *how to produce* one. They are orthogonal
> concepts that compose freely.
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Hypothesis

Iteration in Orthon is defined by a **protocol** (a trait), not by a keyword
or special syntax. Any type that implements `Iterator<T>` can be consumed
by `for` loops and combinator chains. Generator functions (see
[`GENERATORS.md`](../important/GENERATORS.md)) are one way to create
iterators; collections, I/O streams, and range expressions are others.

The iterator protocol is:
- **Lazy** — elements are produced on demand, not eagerly.
- **Single-pass** — an iterator is consumed as it is traversed. To iterate
  again, create a new iterator from the source.
- **Composable** — combinators (`.map()`, `.filter()`, `.take()`, etc.)
  return new iterators without materialising intermediate collections.
- **Zero-cost** — combinator chains compile to tight loops with no
  per-element allocation overhead (monomorphisation).

```
for item in collection:
    process(item)

# Combinator chain — no intermediate allocation
result = numbers
    .filter(|n| n > 0)
    .map(|n| n * 2)
    .take(10)
    .collect()
```

## Issue (Why)

How does a language provide lazy, composable, memory-efficient iteration
over sequences without forcing the programmer to manage iterator state
manually?

Languages have evolved several approaches:

| Approach | Example | Lazy? | Composable? | Zero-cost? |
|----------|---------|-------|-------------|------------|
| **Manual loop** | `for i in range(len)`, C-style `for` | Yes | No | Yes |
| **Iterator interface** | Java `Iterator<T>`, Rust `Iterator`, Python `__iter__`/`__next__` | Yes | Via combinators | Varies |
| **Generator functions** | Python `yield`, C# `yield return` | Yes | Via combinators | Varies |
| **List comprehensions** | Python `[x for x in ...]` | No (eager) | No | No (allocates) |
| **Stream API** | Java `Stream<T>`, C# LINQ | Yes | Yes | No (boxing) |

The core tension: **laziness and composability are essential for writing
efficient sequence transformations without intermediate allocations, but
they require a well-defined protocol that the compiler can optimise.**

Orthon's existing concept `GENERATORS.md` addresses *how to produce*
sequences via `yield`. This document addresses *how to consume* sequences
— the protocol that makes consumption uniform regardless of whether the
source is a generator, a collection, or an I/O stream.

## Level

**Core** — The `Iterator` trait and its combinators are part of the language
specification, not a library. The `for` loop desugars to the `Iterator`
protocol. This is consistent with Orthon's principle that sequences are
a first-class concept (see `DATA_MODEL.md`).

## Proposal

### 1. The `Iterator` Trait

```orthon
trait Iterator[T]
    fun next(self) -> Option[T]
```

A type implements `Iterator[T]` by providing a `next` method that returns
`Some(value)` for each element and `None` when the sequence is exhausted.
The trait is single-use: after `next` returns `None`, calling `next` again
may return `None` or behave unpredictably.

### 2. `for` Loop Desugaring

```orthon
for item in iterator:
    process(item)

# Desugars to:
let mut it = iterator
loop:
    match it.next():
        Some(item) -> process(item)
        None       -> break
```

The `for` loop accepts any expression of type `Iterator[T]`. The compiler
enforces this: passing a non-iterator type to `for` is a compile error.

### 3. Standard Combinators

All combinators return lazy iterators — they do not allocate intermediate
collections. Combinators are methods on `Iterator[T]` with default
implementations:

| Combinator | Signature | Description |
|------------|-----------|-------------|
| `.map(fn)` | `Iterator[U]` where `fn: T -> U` | Transform each element |
| `.filter(pred)` | `Iterator[T]` where `pred: T -> Bool` | Keep elements matching predicate |
| `.take(n)` | `Iterator[T]` | Yield first `n` elements, then stop |
| `.skip(n)` | `Iterator[T]` | Skip first `n` elements |
| `.flat_map(fn)` | `Iterator[U]` where `fn: T -> Iterator[U]` | Map to iterator, then flatten |
| `.zip(other)` | `Iterator[(T, U)]` where `other: Iterator[U]` | Pair elements from two iterators |
| `.enumerate()` | `Iterator[(Int, T)]` | Pair each element with its index |
| `.collect()` | `Collection[T]` | Materialise into a concrete collection |
| `.fold(init, fn)` | `U` where `fn: (U, T) -> U` | Reduce to a single value |
| `.for_each(fn)` | `Void` where `fn: T -> Void` | Side-effect for each element |
| `.count()` | `Int` | Count elements (consumes iterator) |
| `.all(pred)` | `Bool` | Check all elements satisfy predicate |
| `.any(pred)` | `Bool` | Check any element satisfies predicate |

Example usage:

```orthon
# Lazy chain — no intermediate allocation
result = users
    .filter(|u| u.is_active)
    .map(|u| u.name)
    .take(20)
    .collect()

# Folding
total = orders
    .map(|o| o.amount)
    .fold(0, |acc, amt| acc + amt)

# Short-circuiting
has_admins = users.any(|u| u.role == Admin)
```

### 4. Collection Conversion

Types that can produce iterators implement `IntoIterator[T]`:

```orthon
trait IntoIterator[T]
    fun iter(self) -> Iterator[T]
```

This trait is what the `for` loop actually accepts. `Iterator[T]` itself
implements `IntoIterator[T]` (returning `self`), so both iterators and
collections work with `for`.

### 5. Range Expressions

Ranges produce iterators:

```orthon
for i in 0..10:       # 0, 1, ..., 9 — exclusive end
    ...

for i in 0..=10:      # 0, 1, ..., 10 — inclusive end
    ...

for i in 0..10:step(2)  # 0, 2, 4, 6, 8
    ...
```

Range iterators are zero-cost: they compile to a simple counter loop with
no heap allocation.

### 6. Ownership and Mutability

- An iterator borrows its source by default (read-only iteration).
- `.collect()` produces an owned collection.
- For mutable iteration (modifying elements in place), the iterator yields
  `&mut T` references. The combinators `.map_mut()`, `.filter_mut()` handle
  this case explicitly.

## Trade-offs

| Aspect | Iterator Protocol (this proposal) | Generator-only (GENERATORS.md) |
|--------|-----------------------------------|--------------------------------|
| **Separation of concerns** | Production (`yield`) and consumption (iterators) are orthogonal | Both are handled by generators — one mechanism |
| **Combinator flexibility** | High — combinators are methods on the trait, extensible by users | Medium — only combinators built into the generator framework |
| **Collection integration** | Natural — collections implement `IntoIterator` | Works but collections need generator-like interface |
| **Zero-cost** | Yes — monomorphisation eliminates combinator overhead | Yes — stackless generators compile to state machines |
| **Learning curve** | Medium — programmer learns the trait and combinators | Lower — one mechanism for both producing and consuming |
| **Implementation complexity** | Higher — requires trait system with method default impls | Lower — `yield` is a keyword, no trait machinery needed |

### Why separate from GENERATORS.md?

`GENERATORS.md` focuses on the `yield` keyword and how functions produce
sequences lazily. This document focuses on how sequences are *consumed* —
the trait, the combinators, the `for` desugaring. They are independent:

- A generator function returns an `Iterator[T]` value.
- The iterator combinators (`.map()`, `.filter()`, etc.) work on any
  `Iterator[T]`, regardless of whether it was created by a generator, a
  collection, or a range.
- You could have generators without combinators (Python's original `yield`
  is just a `for` target), and you could have combinators without generators
  (Java's streams work with lambdas).

Orthogonality demands separating the two concepts. The `Iterator` trait is
the consumption side; `GENERATORS.md` is the production side.

### Why combinators on `Iterator`, not a separate `Stream` type?

Some languages (Java, C#) distinguish `Iterator` (pull-based, imperative)
from `Stream`/`Enumerable` (pull-based, functional with combinators).
Orthon places combinators directly on `Iterator[T]` for simplicity:
one protocol, one set of methods, no type conversion.

The key insight: an iterator **is** a stream — it produces values over time.
Separating them creates an unnecessary type barrier. If an iterator is
single-pass (which it is in this proposal), combinators work naturally:
`.map()` returns a new `Iterator[U]` that calls `next()` on the original,
applies the transform, and yields the result.

## Related Concepts and Alternatives

| Document | Relationship |
|----------|-------------|
| [`GENERATORS.md`](../important/GENERATORS.md) | The `yield`-based production mechanism. Generators implement `Iterator[T]` and are consumed through this protocol |
| [`EMIT_AS_INTERMEDIATE_RESULT.md`](../important/EMIT_AS_INTERMEDIATE_RESULT.md) | `emit` for intermediate results within a computation — related to but distinct from iteration |
| [`PUSH_STREAMS.md`](../important/PUSH_STREAMS.md) | Push-based streams (observables) — the dual of pull-based iterators. Both are sequence protocols |
| [`ITERATION_LOOP.md`](../important/ITERATION_LOOP.md) | Loop constructs — the `for` loop desugars to the iterator protocol |
| [`EXECUTION_PROGRAM.md`](EXECUTION_PROGRAM.md) | Sequences in Execution Programs — iterators are the pull-based sequence primitive |
| [`DATA_MODEL.md`](DATA_MODEL.md) | Sequences as a data representation — iterators provide a way to consume them |
| [`TRAITS.md`](TRAITS.md) | The trait system that `Iterator` depends on — trait bounds, default methods, static dispatch |
| [`FUNCTIONS.md`](FUNCTIONS.md) | Closures/lambdas used in combinators (`|n| n > 5`) |
| [`imperative-crutch-lazy-sequences.md`](../imperative-crutch-lazy-sequences.md) | Analysis of lazy sequence pain points in imperative languages |
| [`imperative-crutch-collections-loops.md`](../imperative-crutch-collections-loops.md) | Analysis of loop-related pain points in imperative languages |

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Iterator Protocol Policy | Defines the `Iterator[T]` trait, `next()` method, and single-use semantics |
| Combinator Policy | Determines which combinators are built-in vs. library-defined |
| Laziness Policy | Governs whether combinator chains are always lazy, or can be eager for optimisation |
| Collection Policy | Defines `IntoIterator[T]` and how collections expose iterators |
| Range Policy | Specifies range syntax (`0..10`, `0..=10`, `.step(n)`) and range iterator semantics |
| Desugaring Policy | Formalises `for` loop desugaring to `Iterator[T]` protocol |

## Open Questions

1. Should `Iterator[T]` also support `size_hint()` for optimising collection
   pre-allocation?
2. Should there be a `DoubleEndedIterator[T]` (`.next_back()`) for iterators
   that can be traversed from both ends?
3. Should combinators support parallel execution (e.g., `.par_map()`) or
   should that be a separate concept?
4. How does the iterator protocol interact with the `delegate` execution
   policy — can an iterator be delegated?
5. Should `for` accept owned collections directly (via `IntoIterator`) or
   only references?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
