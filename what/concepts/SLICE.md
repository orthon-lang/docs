# Slice

> **✅ ACCEPTED — [EDR-084](../how/decision_records/architecture/EDR-084-slice.md).**
>
> **Status:** Accepted 2026-08-05.
>
> **See also:** [`RANGE.md`](RANGE.md) (EDR-083),
> [`INDEXING.md`](INDEXING.md) (EDR-082),
> [`SPAN.md`](SPAN.md) (EDR-064),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md) (EDR-022),
> [`GLOSSARY.md`](../GLOSSARY.md) § Slice

---

## Issue (Why)

How does Orthon select a contiguous sub-sequence of a random-access composite by position — without copying, without a second indexing convention, and without leaking the `+1` length arithmetic to the programmer?

INDEXING (EDR-082) defines one-element access `a[i] ≡ a@get(i)`. RANGE (EDR-083) defines the inclusive-inclusive range value. A slice is the **multi-element form**: a Range applied to a random-access composite.

## Principles

1. **Multi-element `@get`** — slicing is the multi-element form of positional access; same protocol, same 1-based base.
2. **Zero-copy** — contiguous slicing produces a non-owning `Span` view (EDR-064).
3. **Inclusive bounds** — `items[a..b]` is a..b inclusive; length is owned by the language.
4. **Contiguity matters** — strided (non-contiguous) slicing yields an iterator, never a `Span`.
5. **Empty is a value** — `items[1..0]` is an empty slice, not an error.

## Policy Footprint

| Policy Type | Role |
|-------------|------|
| Collection Indexing Policy | `OneBased` (EDR-082) — slice bounds are 1-based |
| Slicing Policy | Zero-copy `Span` view for contiguous ranges (EDR-064) |
| Range Semantics Policy | Inclusive-inclusive bounds (EDR-083) |
| FFI Boundary Policy | `0..<N` interop utility only (deferred to FFI, Milestone 8) |

## Model (What)

### Canonical Forms

```orthon
items[1..k]      # first k elements — contiguous → Span view
text[1..3]       # string slice
tuple[1..2]      # tuple slice
span[1..3]       # sub-span view (no copy)
```

`items[a..b]` selects the elements at indices `a` through `b` inclusive. It is the multi-element form of `a[i]` (INDEXING) and reuses the same `@get`/`Indexable`-like protocol.

### Empty Slice

```orthon
items[1..0]      # empty slice, len == 0 (end < start)
```

An empty slice is a value, not a syntax error. `len(items[1..0]) == 0`.

### Strided Slice

```orthon
items[1..k].step(2)                 # non-contiguous → iterator of elements
items[1..k].step(2).collect()       # → owned collection
# never a Span — a span is contiguous by definition
```

A strided slice is non-contiguous, so it produces an `Iterator` of the selected elements (or a materialised collection via `.collect()`), never a `Span` view.

### Interaction with Span

Slicing a `Span` or array-backed collection produces a non-owning `Span` view (EDR-064): zero-copy, bounds-checked, lifetime-tracked. Slicing a string or tuple uses the same `@get`-based protocol; string slicing may produce a `StrSpan` (SPAN Open Q1, deferred).

### Interaction with Indexing

`a[i]` is the one-element form; `a[1..k]` is the multi-element form. Both decompose to the `@get` protocol on the `Indexable`-like trait (INDEXING EDR-082).

## Default Strategy

`items[a..b]` desugars to a bounds-checked range extraction on the `Indexable`-like trait. Contiguous slice of a `Span`/array → zero-copy `Span` view (pointer + length). Strided → iterator. Empty → zero-length value.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Copying slice | Materialises a copy | Explicitly via `.collect()` |
| 0-based slice | Second indexing base | Rejected — single-base rule (EDR-082) |
| Slice-only-on-Span | Restrict slicing to memory views | Rejected — strings/tuples also slice (INDEXING) |

## Open Questions

1. **`StrSpan`** — does string slicing produce a `StrSpan`? (SPAN Open Q1, deferred.)
2. **Multi-dimensional slices** — matrix views (`m[1..2, 3..4]`)? Deferred (SPAN Open Q3 / FFI Milestone 8).

## Decision History

- **Slice = Range applied to a random-access composite** adopted (INDEXING's `@get` scope + RANGE's value).
- **Zero-copy contiguous slice → Span view** adopted (SPAN EDR-064).
- **Strided → iterator, not Span** adopted (contiguity invariant).
- **Empty slice is a value** adopted (INDEXING norm).
- **Accepted via EDR-084** on 2026-08-05.

---

### Affected Documents

- [x] `what/concepts/SPAN.md` (referenced; example sweep in TODO IX-B1)
- [x] `what/concepts/INDEXING.md`
- [x] `what/concepts/RANGE.md`
- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/CONFLICT_REGISTRY.md`
- [x] `how/concepts/research/essential/RANGE_SLICE.md`
