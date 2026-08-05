# Indexing

> Accepted concept. Acceptance: [EDR-082](../../how/decision_records/architecture/EDR-082-1-based-indexing.md).
> Full reasoning trail: [`how/concepts/research/important/INDEXING_ONE_BASED.md`](../../how/concepts/research/important/INDEXING_ONE_BASED.md).

## Issue (Why)

Orthon collections must identify elements by position. The index base
(0 vs 1) is a **Language decision**: it affects the meaning of every `a[i]`
expression and every index-based `for` loop, and cannot be changed by a
library. Orthon adopts **1-based indexing** to match natural human counting,
eliminate the off-by-one error class, and allow direct translation from
mathematical and domain notation. 0-based indexing is rooted in C pointer
arithmetic (`arr[i]` ≡ `*(base + i * sizeof(T))`) — a hardware abstraction
with no place in a language built on the Data abstraction.

## Decision

- **Index base = 1.** First element at index 1; last at `len(collection)`.
  The base is a semantic parameter of the `@get` protocol contract: first at
  `@get(1)`, last at `@get(len(a))`.
- **`a[i]` ≡ `a@get(i)`** — a Level 2 language pattern over the Metadata
  Protocol (`@`-prefix), decomposing to `function` + `call`. Not attribute
  access (`.` is reserved for user-defined named-member access).
- **Range norm: inclusive-inclusive `1..N`** — the only range semantic,
  uniform across index access, slices, and iteration. The language owns the
  `+1` length arithmetic (`len(slice)` = element count). Empty slice =
  `end < start`. `0..<N` is an FFI-boundary interop utility only.
- **`enumerate` defaults to 1** — a plain StdLib method on `Iterator[T]`,
  defined as `enumerate(items) ≡ zip(1..=len(items), items)`. No start
  parameter; offsets use an explicit preliminary range.
- **Single-base rule: `Span` is 1-based** — raw C buffers translate at the FFI
  boundary, never a second base.
- **No configurable base** in v0.1.

## Canonical Forms

```orthon
items[1]               -- first element
items[len(items)]      -- last element
items[1..k]            -- first k elements (inclusive-inclusive)
for i in 1..len(array):    -- index-based iteration
    process(array[i])
for (i, v) in items.enumerate():   -- pairs (1, first), (2, second), …
    process(i, v)
```

## Policy Footprint

| Policy Type | Role |
|-------------|------|
| Collection Indexing Policy | `OneBased` (see `IMPLEMENTATION_POLICIES.md`) |
| FFI Boundary Policy | Governs index translation at the C boundary (deferred to FFI, Milestone 8) |
| Range Semantics Policy | Inclusive-inclusive `1..N` (deferred to RANGE concept, Type A) |

## Related Concepts

- **ITERATION_LOOP** (EDR-053) — canonical index iteration `for i in 1..len(array)`.
- **ITERATOR_PROTOCOL** (EDR-022) — `enumerate` base pinned to 1.
- **SPAN** (EDR-064) — single-base rule.
- **COMPOSABLE_COLLECTION_OPS** (EDR-032) — `zip` in the `enumerate` composition.
- **RANGE** (Type A, pending) — range literal representation.
- **FFI** (Milestone 8) — index translation at the boundary.

## Cross-Concept Amendments (C-001)

Applied at acceptance (EDR-082): ITERATION_LOOP, ITERATOR_PROTOCOL, SPAN, and
GLOSSARY examples updated from 0-based to the 1-based canonical forms.
