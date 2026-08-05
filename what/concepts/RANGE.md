# Range

> **✅ ACCEPTED — [EDR-083](../how/decision_records/architecture/EDR-083-range.md).**
>
> **Status:** Accepted 2026-08-05.
>
> **See also:** [`INDEXING.md`](INDEXING.md) (EDR-082),
> [`SLICE.md`](SLICE.md) (EDR-084),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md) (EDR-022),
> [`ITERATION_LOOP.md`](ITERATION_LOOP.md) (EDR-053),
> [`GLOSSARY.md`](../GLOSSARY.md) § Range,
> [`IMPLEMENTATION_POLICIES.md`](../how/IMPLEMENTATION_POLICIES.md) § Range Semantics Policy

---

## Issue (Why)

How does Orthon express a contiguous run of integers — as an index expression, a slice bound, or an iteration source — without a second, half-open semantic that leaks the `+1` length arithmetic to the programmer?

EDR-082 (INDEXING) established the norm: inclusive-inclusive `1..N` is the *only* range semantic, applied uniformly to index access, slices, and iteration. But the range itself was still buried inside ITERATOR_PROTOCOL (EDR-022) with 0-based, split exclusive/inclusive spellings (`0..10`, `0..=10`) that contradict the accepted norm. This concept defines the range as a **first-class value**, semantically separate from the Iterator trait.

## Principles

1. **First-class value** — `Range` is a value type (a compact descriptor), not an iterator and not a collection.
2. **Single semantic** — inclusive-inclusive `1..N` is the only range semantic; `..=` is eliminated.
3. **Named before symbolic** — `range(a, b)` is the named canonical form of `a..b`.
4. **Iterable value** — `Range` implements `IntoIterator[Int]`; combinator chains apply directly and stay lazy.
5. **Zero-cost** — ranges compile to a simple counter loop; no heap allocation.
6. **FFI isolation** — `0..<N` exists only at the FFI boundary, never in application code.

## Policy Footprint

| Policy Type | Role |
|-------------|------|
| Range Semantics Policy | `InclusiveInclusive` — `1..N` is the only semantic; `..=` eliminated |
| Collection Indexing Policy | `OneBased` (EDR-082) — ranges align with the 1-based index base |
| Iterator Protocol Policy | `Range` implements `IntoIterator[Int]` (EDR-022) |
| Combinator Policy | Combinators apply directly to range values (EDR-032) |
| FFI Boundary Policy | `0..<N` interop utility only (deferred to FFI, Milestone 8) |

## Model (What)

### Range Literal

```orthon
1..5          # 1, 2, 3, 4, 5 — inclusive-inclusive, 5 elements
1..len(items) # all indices of items (last == len)
```

`a..b` is inclusive-inclusive: it produces the sequence `a, a+1, …, b`, which is `b - a + 1` elements. There is no `..=` spelling and no half-open form in application code.

### Named Canonical Form

```orthon
range(1, 5)            # ≡ 1..5
range(1, len(items))   # ≡ 1..len(items)
```

`range(a, b)` is the named form of `a..b` (Named Before Symbolic). Both are equivalent.

### Empty Range

```orthon
1..0          # empty — zero elements (end < start), a value, not an error
range(5, 1)   # empty
```

An empty range is a value with `end < start`. `len(1..0) == 0`. It iterates zero times and slices to an empty result.

### Iteration and Combinators

```orthon
for i in 1..5:              # 1, 2, 3, 4, 5
    process(i)

(1..5).map(|x| x * 2)                          # 2, 4, 6, 8, 10
(1..100).filter(|x| x % 2 == 0).take(3)        # 2, 4, 6 — lazy chain
```

`Range` implements `IntoIterator[Int]`, so `for` loops and combinator chains accept it directly, without an explicit `.iter()` step. Chains stay lazy (Laziness Policy).

### Step

```orthon
(1..5).step(2)    # 1, 3, 5 — strided range (non-contiguous)
(1..5).step(1)    # 1, 2, 3, 4, 5
(1..5).step(-1)   # 5, 4, 3, 2, 1 — descending
```

`.step(n)` is a method on `Range` returning a *strided* `Range` value (still a data value, still `IntoIterator`). `step(0)` is a compile-time error. A negative step iterates in descending direction. A strided range is non-contiguous: applied to indexing, it yields an iterator of elements, never a contiguous `Span` view (see [`SLICE.md`](SLICE.md)).

### Range as Indexing Expression

```orthon
items[1..k]    # slice — first k elements (inclusive-inclusive)
```

A range used inside `[...]` selects a contiguous sub-sequence of a random-access composite via the `@get` protocol (INDEXING EDR-082). The range supplies the bounds; slicing supplies the semantics (see [`SLICE.md`](SLICE.md), EDR-084).

### FFI Interop Utility

```orthon
# At the FFI boundary only — never in application code
0..<n          # half-open C-style bounds for index translation
```

`0..<N` exists only as an interop utility at the FFI boundary. Visibility is determined by the FFI index-translation policy (Milestone 8).

### Interaction with `enumerate`

`enumerate` pairs each element with its 1-based ordinal:

```orthon
items.enumerate()            # (1, first), (2, second), …
# ≡ zip(1..len(items), items)   # spelling per EDR-083 (no `..=`)
```

The `..=` spelling used in EDR-082/INDEXING is superseded: the composition is `zip(1..len(items), items)`.

## Default Strategy

`Range` is a compact `pack` composite (`start`, `end`). The `a..b` literal is a Language-level pattern desugaring to a `Range` value; `range(a, b)` is the StdLib constructor. `IntoIterator` on `Range` is implemented as a counter loop — zero-cost, no heap allocation in every strategy. `.step(n)` produces a strided descriptor with the same zero-cost iteration property.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Half-open ranges (`a..b` exclusive, `..=` inclusive) | Rust-style dual spelling | Rejected — contradicts the inclusive-inclusive norm (EDR-082) |
| Range as built-in keyword type | Compiler-owned range type | Rejected — composes from existing primitives; only the literal needs Language support |
| Range as iterator directly | Range IS an Iterator, not a value | Rejected — conflates the descriptor with consumption state; breaks orthogonality |
| Eager materialisation | Range → owned collection eagerly | Only when explicitly requested (`range(1, 5).collect()`) |

## Open Questions

1. **Range of non-integer types** — should `Range` generalize to `Range[T]` over any ordered `pack` (e.g., `a..b` over `Char`, `Date`)? v0.1 commits to `Range[Int]`; generalization is a StdLib extension question.
2. **Open-ended bounds** — can `a..` (open end) and `..b` (open start) exist, or is a range always fully bounded? Deferred — not required by the v0.1 use cases (indexing, slices, integer loops).

## Decision History

- **First-class value** adopted over range-inside-iterator. Rationale: separates the descriptor from the consumption protocol; enables slicing and reuse.
- **Inclusive-inclusive `1..N`** adopted as the only semantic (EDR-082 norm); `..=` eliminated.
- **`range(a, b)` named form** adopted per Named Before Symbolic.
- **StdLib `Range` type + Language literal** adopted (LIBRARY_BOUNDARY per EDR-082).
- **`IntoIterator` on `Range`** adopted (SEQUENCE_METHODS) — combinators apply directly.
- **`.step(n)` returns a strided Range value** adopted (RANGE_STEP); `step(0)` is a compile error.
- **Accepted via EDR-083** on 2026-08-05.

---

### Affected Documents

- [x] `what/concepts/INDEXING.md` (enumerate formula spelling)
- [x] `what/concepts/ITERATOR_PROTOCOL.md` (range delegation)
- [x] `what/concepts/ITERATION_LOOP.md` (range delegation)
- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/CONFLICT_REGISTRY.md`
- [x] `how/IMPLEMENTATION_POLICIES.md`
- [x] `how/concepts/research/essential/RANGE_SLICE.md`
- [x] `how/concepts/research/important/RANGE_STEP.md`
- [x] `how/concepts/research/important/SEQUENCE_METHODS.md`
