# Composable Collection Operations

> **✅ ACCEPTED — [EDR-032](../how/decision_records/architecture/EDR-032-composable-collection-ops.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **Classification:** **StdLib** — These operations are compositions of
> the [`ITERATOR_PROTOCOL`](ITERATOR_PROTOCOL.md) and do not require new
> language semantics. See EDR-032 for the full classification rationale.
>
> **See also:** [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Combinator, Iterator,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Manual index-based loops, empty accumulator lists, and explicit flags for search force the programmer to describe *how* to iterate instead of *what* to obtain. This pattern duplicates business logic with iteration mechanics and defers type errors to runtime.

Orthon eliminates index-based and accumulator-based loops as a primary pattern by providing declarative, composable collection operations built on the [`ITERATOR_PROTOCOL`](ITERATOR_PROTOCOL.md).

| Crutch pattern | Modern replacement |
|---|---|
| `for i in range(len(lst)):` | `for item in list:` |
| Empty list + loop + `append` | `.map()`, `.filter()` |
| Loop + `break` + `found` flag | `.find()`, `.any()` |
| Loop + accumulator | `.fold()`, `.reduce()` |

## Principles

1. **Built on ITERATOR_PROTOCOL** — All composable collection operations are methods on `Iterator[T]`. They require no language-level support beyond what the iterator protocol provides.
2. **Lazy by default** — Combinator chains produce lazy iterators. Materialisation is explicit (`.collect()`, `.to_list()`, `.to_set()`).
3. **Declarative over imperative** — The programmer describes *what* to obtain, not *how* to iterate. Manual `for` loops remain available as escape hatches.
4. **Type-safe** — All combinator operations are fully typed at compile time. No runtime type errors from incorrect transformations.
5. **Loop fusion is an optimisation** — Combining multiple combinator passes into a single loop (loop fusion) is an Implementation Strategy concern, not language semantics. The language specifies what operations mean; the compiler/runtime decides how to optimise them.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Iterator Combinator Policy | Defines which combinators are provided by the StdLib |
| Laziness Policy | Governs that combinator chains are lazy by default; materialisation is explicit |
| Loop Fusion Policy | Implementation Strategy concern — governs whether and how multiple combinators are fused |

## Model (What)

### Core Combinators

All combinators are methods on `Iterator[T]`:

```orthon
# Transformation
iterator@map(fn: Fun(T) -> U) -> Iterator[U]
iterator@filter(fn: Fun(T) -> Bool) -> Iterator[T]

# Aggregation
iterator@fold(initial: U, fn: Fun(U, T) -> U) -> U
iterator@reduce(fn: Fun(T, T) -> T) -> Option[T]

# Selection
iterator@find(fn: Fun(T) -> Bool) -> Option[T]
iterator@any(fn: Fun(T) -> Bool) -> Bool
iterator@all(fn: Fun(T) -> Bool) -> Bool
iterator@nth(n: Int) -> Option[T]

# Cardinality
iterator@count() -> Int
iterator@first() -> Option[T]
iterator@last() -> Option[T]

# Materialisation
iterator@collect() -> Vec[T]
iterator@to_list() -> List[T]
iterator@to_set() -> Set[T]
iterator@to_map(fn: Fun(T) -> (K, V)) -> Map[K, V]

# Sub-iteration
iterator@take(n: Int) -> Iterator[T]
iterator@skip(n: Int) -> Iterator[T]
iterator@take_while(fn: Fun(T) -> Bool) -> Iterator[T]
iterator@skip_while(fn: Fun(T) -> Bool) -> Iterator[T]

# Combination
iterator@chain(other: Iterator[T]) -> Iterator[T]
iterator@zip(other: Iterator[U]) -> Iterator[(T, U)]
iterator@enumerate() -> Iterator[(Int, T)]
```

### Usage Examples

```orthon
# Map and filter
let result = numbers
    @filter(fn(x) -> x > 0)
    @map(fn(x) -> x * 2)
    @collect()

# Find first match
let found = users@find(fn(u) -> u.age > 18)

# Fold / reduce
let sum = numbers@fold(0, fn(acc, x) -> acc + x)
let max = numbers@reduce(fn(a, b) -> if a > b then a else b)

# Chaining with take
let first_five = numbers
    @filter(fn(x) -> x % 2 == 0)
    @take(5)
    @collect()
```

### Relationship to ITERATOR_PROTOCOL

The composable collection operations are the **consumption-side combinator layer** on top of `Iterator[T]`. The iterator protocol provides the fundamental `next()` mechanism (one element at a time); combinators provide declarative transformations that compose over that mechanism.

```
Generator/Collection → Iterator[T] → Combinators (map/filter/...) → Materialise
       (production)    (protocol)        (transformation)           (consumption)
```

## Default Strategy

All combinators are provided by the Standard Library as functions on `Iterator[T]`. They are lazy by default: each combinator returns a new `Iterator` value that applies the transformation on demand. Materialisation is explicit (`.collect()`, `.to_list()`, etc.).

Loop fusion is an Implementation Strategy optimisation — the default strategy does not guarantee fusion beyond what monomorphisation naturally achieves.

## Alternative Strategies

| Strategy | Trade-offs |
|---|---|
| **Eager by default** | Simpler mental model but higher allocation overhead and no short-circuit fusion. Rejected — laziness is the modern default (Phase 3 D-06). |
| **Built-in syntax (comprehensions)** | More concise but adds syntax to the language. Comprehensions could be added as syntactic sugar over combinator chains without changing semantics. |
| **Parallel combinators** | `.par_map()`, `.par_filter()` etc. Could be added as a separate extension or as a strategy parameter. Deferred to v0.2+. |

## Open Questions

1. Should parallel combinator variants (`.par_map()`, `.par_filter()`) be part of the StdLib or a separate concurrency library?
2. Should comprehensions be added as syntactic sugar for combinator chains?
3. How should early exit (break/return) patterns be expressed in combinator chains — through combinators like `.take_while()` or through a dedicated mechanism?

## Decision History

- **2026-07-27:** Accepted via EDR-032. Classified as StdLib — all operations are compositions of `Iterator[T]` protocol methods. Loop fusion deferred to Implementation Strategy. Laziness inherited from Phase 3 D-06.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [x] `what/concepts/ITERATOR_PROTOCOL.md` — primary dependency
- [ ] `what/concepts/LAZY_SEQUENCE_GENERATORS.md`
