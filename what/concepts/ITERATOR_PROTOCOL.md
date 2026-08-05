# Iterator Protocol

> **✅ ACCEPTED — [EDR-022](../how/decision_records/architecture/EDR-022-iterator-protocol.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`RANGE.md`](RANGE.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Evaluation,
> [`GLOSSARY.md`](../GLOSSARY.md) § Iterator Protocol, IntoIterator,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

How does a language provide lazy, composable, memory-efficient iteration over sequences without forcing the programmer to manage iterator state manually?

Languages have evolved several approaches:

| Approach | Example | Lazy? | Composable? | Zero-cost? |
|----------|---------|-------|-------------|------------|
| **Manual loop** | C-style `for` | Yes | No | Yes |
| **Iterator interface** | Rust `Iterator`, Java `Iterator<T>` | Yes | Via combinators | Varies |
| **Generator functions** | Python `yield` | Yes | Via combinators | Varies |
| **List comprehensions** | Python `[x for x in ...]` | No (eager) | No | No (allocates) |
| **Stream API** | Java `Stream<T>`, C# LINQ | Yes | Yes | No (boxing) |

The core tension: **laziness and composability are essential for writing efficient sequence transformations without intermediate allocations, but they require a well-defined protocol that the compiler can optimise.**

Orthon's solution: the **`Iterator[T]` trait** defines a lazy consumption protocol — single-pass, composable, with zero-cost combinator chains. Generators (see [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md)) are the production side; the iterator protocol is the consumption side.

## Principles

1. **Trait-based** — The iterator protocol is defined by the `Iterator[T]` trait. Any type that implements it can be used in `for` loops and combinator chains.
2. **Lazy** — Elements are produced on demand via `next()`. Combinators return lazy iterators, never eager collections.
3. **Single-pass** — An iterator is consumed as it is traversed. To iterate again, create a new iterator from the source.
4. **Composable** — Combinators (`.map()`, `.filter()`, `.take()`, etc.) return new `Iterator` values without materialising intermediate collections.
5. **Zero-cost** — Combinator chains compile to tight loops with no per-element allocation overhead (monomorphisation).
6. **`@` for protocol methods** — Per Phase 3 D-07, protocol-level access on iterators uses the `@` prefix (e.g., `iterator@next()`).
7. **`IntoIterator` for collections** — Collections implement `IntoIterator[T]` to enable `for` loop usage. `Iterator[T]` itself implements `IntoIterator[T]` (returning `self`).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Iterator Protocol Policy | Defines the `Iterator[T]` trait, `next()` method, and single-use semantics |
| Combinator Policy | Determines which combinators are StdLib vs. built-in — combinators are StdLib |
| Laziness Policy | Governs that combinator chains are always lazy; materialisation is explicit |
| Collection Policy | Defines `IntoIterator[T]` and how collections expose iterators |
| Range Policy | Range syntax and semantics delegated to the RANGE concept (EDR-083) — inclusive-inclusive `1..N`; ranges implement `IntoIterator` and enter combinator chains directly |
| Desugaring Policy | Formalises `for` loop desugaring to `Iterator[T]` protocol |

## Model (What)

### The `Iterator[T]` Trait

```orthon
trait Iterator[T]
    fn next(self) -> Option[T]
```

A type implements `Iterator[T]` by providing a `next` method that returns `Some(value)` for each element and `None` when the sequence is exhausted. The trait is single-use: after `next` returns `None`, calling `next` again may return `None` or behave unpredictably.

```orthon
# @ prefix for protocol method access (per D-07)
let item = iterator@next()
```

### `for` Loop Desugaring

```orthon
for item in collection:
    process(item)

# Desugars to:
let mut it = collection@iter()    # calls IntoIterator::iter
loop:
    match it@next():               # calls Iterator::next with @ prefix
        Some(item) -> process(item)
        None       -> break
```

The `for` loop accepts any type that implements `IntoIterator[T]`. The compiler rejects non-iterable types at compile time.

### All Canonical Forms

```orthon
# Form 1: for loop (preferred for simple iteration)
for item in collection:
    process(item)

# Form 2: Explicit next() calls
let mut it = collection@iter()
loop:
    let item = it@next()
    match item:
        Some(value) -> process(value)
        None        -> break

# Form 3: Combinator chain (preferred for transformations)
collection
    .filter(|x| x > 0)
    .map(|x| x * 2)
    .for_each(|x| process(x))
```

All three forms produce the same semantics. The `for` loop is preferred for simple iteration; combinators are preferred for transformations.

### Standard Combinators (StdLib)

Combinators are methods on `Iterator[T]` with default implementations. They are part of the Standard Library, not the core language:

| Combinator | Signature | Description |
|------------|-----------|-------------|
| `.map(fn)` | `Iterator[U]` where `fn: T -> U` | Transform each element |
| `.filter(pred)` | `Iterator[T]` where `pred: T -> Bool` | Keep elements matching predicate |
| `.take(n)` | `Iterator[T]` | Yield first `n` elements, then stop |
| `.skip(n)` | `Iterator[T]` | Skip first `n` elements |
| `.flat_map(fn)` | `Iterator[U]` where `fn: T -> Iterator[U]` | Map to iterator, then flatten |
| `.zip(other)` | `Iterator[(T, U)]` where `other: Iterator[U]` | Pair elements from two iterators |
| `.enumerate()` | `Iterator[(Int, T)]` | Pair each element with its 1-based index; `enumerate(items) ≡ zip(1..len(items), items)` (EDR-082/EDR-083) |
| `.collect()` | `Collection[T]` | Materialise into a concrete collection |
| `.fold(init, fn)` | `U` where `fn: (U, T) -> U` | Reduce to a single value |
| `.for_each(fn)` | `Void` where `fn: T -> Void` | Side-effect for each element |
| `.count()` | `Int` | Count elements (consumes iterator) |
| `.all(pred)` | `Bool` | Check all elements satisfy predicate |
| `.any(pred)` | `Bool` | Check any element satisfies predicate |

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

### Named Function Equivalents

Per the Named Before Symbolic principle, combinator methods have free function equivalents:

```orthon
collection.filter(pred)     # method form
filter(collection, pred)     # free function form

collection.map(fn)           # method form
map(collection, fn)          # free function form
```

### `IntoIterator[T]` for Collections

Types that can produce iterators implement `IntoIterator[T]`:

```orthon
trait IntoIterator[T]
    fn iter(self) -> Iterator[T]
```

This trait is what the `for` loop and combinator entry points actually accept. `Iterator[T]` itself implements `IntoIterator[T]` (returning `self`), so both iterators and collections work with `for`:

```orthon
for item in iterator:        # Iterator[T] implements IntoIterator[T]
    ...

for item in collection:      # Collection implements IntoIterator[T]
    ...
```

### Range Expressions

Ranges are first-class values defined by the RANGE concept ([`RANGE.md`](RANGE.md), EDR-083), not by the iterator protocol. A range is inclusive-inclusive `1..N`, produces N elements, and implements `IntoIterator[Int]`, so it enters `for` loops and combinator chains directly:

```orthon
for i in 1..10:              # 1, 2, ..., 10 — inclusive-inclusive (10 elements)
    ...

for i in (1..10).step(2)     # 1, 3, 5, 7, 9 — step via a method on Range
    ...
```

Range iteration is zero-cost: it compiles to a simple counter loop with no heap allocation. See [`RANGE.md`](RANGE.md) for the full definition (literal `1..N`, `range(a, b)` named form, `.step(n)`, empty ranges, FFI-only `0..<N`).

### Ownership and Mutability

- An iterator borrows its source by default (read-only iteration).
- `.collect()` produces an owned collection.
- For mutable iteration (modifying elements in place), the iterator yields `&mut T` references.

```orthon
# Read-only iteration (default)
for item in collection:
    print(item)

# Mutable iteration
for item in collection@iter_mut():
    item.field = new_value
```

### Interaction with Generators

The iterator protocol is the **consumption** side; generators (see [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md)) are the **production** side:

- A generator function returns an `Iterator[T]`.
- Iterator combinators work on any `Iterator[T]`, including generators.
- The two concepts are orthogonal — you can have one without the other.

### Relationship with COMPOSABLE_COLLECTION_OPS

The `IntoIterator[T]` trait and iterator combinators (map, filter, collect) provide the foundation for composable collection operations. The COMPOSABLE_COLLECTION_OPS concept (Plan 04-03) extends this foundation with collection-specific operations (sort, unique, join, group_by) that are built on top of the iterator protocol. See EDR-022 § Related Concepts for the dependency chain.

## Default Strategy

The `Iterator[T]` trait is the single iteration protocol. The `for` loop desugars to `IntoIterator::iter()` + `Iterator::next()`. Combinators are implemented as default methods on `Iterator[T]` (StdLib). Range expressions desugar to range-iterator constructors. All combinators return lazy iterators with no intermediate allocation. Monomorphisation eliminates combinator overhead.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Eager iteration | Materialise all elements before iteration | Small collections where lazy overhead dominates |
| Parallel iteration | Split iterator across threads | Large datasets, multi-core targets |
| External iteration | Manual `next()` calls without `for` desugaring | FFI boundaries, custom control flow |
| Streaming iteration | Iterator that yields `Result<T, E>` | I/O streams where each element may fail |

## Open Questions

1. Should `Iterator[T]` also support `size_hint()` for optimising collection pre-allocation?
2. Should there be a `DoubleEndedIterator[T]` (`.next_back()`) for iterators that can be traversed from both ends?
3. Should combinators support parallel execution (e.g., `.par_map()`) or should that be a separate concept?
4. Should `for` accept owned collections directly (via `IntoIterator`) or only references?
5. How does the iterator protocol interact with the `delegate` execution policy — can an iterator be delegated?
6. Should `IntoIterator[T]` be renamed to `Iterable[T]`? (2026-08-05 review — a separate `Iterable` was rejected in discussion, but a pure rename was not previously considered; would amend EDR-022.)

## Decision History

- **Trait-based protocol** adopted over keyword-based iteration (Python `__iter__`/`__next__`). Rationale: Traits provide static dispatch, type safety, and extensibility — any type can implement `Iterator[T]`.
- **`@` for protocol method calls** adopted (Phase 3 D-07). Rationale: `iterator@next()` distinguishes protocol access from attribute access, consistent with Metadata Protocol.
- **Combinators as StdLib** adopted over built-in combinators. Rationale: Core language defines the protocol only; combinators are default method implementations on the trait, living in the Standard Library.
- **Single-pass** adopted over multi-pass. Rationale: Simpler protocol, enables zero-cost combinator chains, consistent with consumption semantics.
- **Lazy by default** adopted. Rationale: Eager evaluation would break infinite sequences and increase allocation. Materialisation is explicit via `.collect()`.
- **`IntoIterator` for `for` loops** adopted. Rationale: Enables `for` to work uniformly on collections, iterators, ranges, and I/O streams.
- **Accepted via EDR-022** on 2026-07-27.
- **2026-08-05 — Range semantics superseded by EDR-083.** Range is now a separate first-class value concept (see [`RANGE.md`](RANGE.md)); this document no longer owns range syntax. Range expressions follow the inclusive-inclusive `1..N` norm; `.enumerate()` base is pinned to 1 per EDR-082/EDR-083.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/concepts/LAZY_SEQUENCE_GENERATORS.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
