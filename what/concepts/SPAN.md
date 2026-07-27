# Span (Memory View / Array View)

## Issue (Why)

How do you provide safe, non-owning access to a contiguous region of memory? Working with arrays and slices involves:
- **Implicit copying** — slicing in some languages creates a copy; programmer intent (reference vs copy) is unclear.
- **Borrowed views** — no copying, but no ownership tracking.
- **Bounds unsafety** — C-style pointer+length has no bounds checking.
- **Lifetime confusion** — a view can outlive the data it points to.

## Principles

1. **Non-owning** — A span does not own the underlying memory; it is a borrowed view.
2. **Lifetime-tracked** — The compiler verifies that the span does not outlive its source data.
3. **Bounds-checked** — Access through a span is bounds-checked (at least in debug mode).
4. **No implicit copying** — Slicing a collection into a span never copies data.
5. **Interchangeable** — Spans can be constructed from arrays, slices, and contiguous collections uniformly.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Lifetime Policy | Determines how span lifetimes are tracked (static borrow checking, runtime GC, or regions) |
| Bounds Checking Policy | Governs whether bounds checks are always on, debug-only, or optimised away |
| Slicing Policy | Controls whether slicing produces a span (reference) or a copy |

## Model (What)

A `Span<T>` is a lightweight, non-owning view over a contiguous sequence of `T` elements:

```orthon
# Constructing spans
let arr = [1, 2, 3, 4, 5]
let span1: Span<Int> = Span(arr)         # whole array
let span2: Span<Int> = arr[1..3]         # sub-range view (no copy)
let span3: Span<Int> = arr[1..]          # from index 1 to end

# Access
let first = span[0]      # bounds-checked
let slice = span[1..3]   # sub-span (no copy)

# Mutation (if source is mutable)
fun process(data: mut Span<Int>):
    data[0] = 42
```

Key features:
- Reference semantics — no implicit copy.
- Sub-span slicing produces spans, not copies.
- Uniform construction from arrays, slices, contiguous buffers.
- Mutable variant via `mut Span<T>`.
- Lifetime check — span cannot outlive the backing storage.

## Default Strategy

Slicing an array produces a `Span` (reference, no copy). Span access is bounds-checked in debug and release-safe mode, unchecked in release-unsafe mode. Lifetime of the span is tied to the owner via borrow checking.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Copying slice | Slicing produces a new owned copy (Python `list[:]`). |
| Memory view | Non-copying but no lifetime tracking (Python `memoryview`). |
| GC-tracked view | View kept alive by GC as long as any reference exists. |
| Unsafe span | No bounds checking; `unsafe` block required to construct. |

## Open Questions

1. Should there be a separate `StrSpan` for string slices?
2. How does span interact with ownership / borrow checking?
3. Should spans support multi-dimensional indexing (matrix view)?

## Decision History

- **EDR-064:** Span accepted as **Language** — compiler-recognized memory view with bounds checking and lifetime tracking. Span requires compiler support for lifetime verification (the view cannot outlive the backing storage) and bounds checking at access sites.
- **Classification per D-03:** Language. Span is a compiler-recognized memory view type with special semantics: lifetime tracking, bounds checking, and slicing protocol. These are compiler-level guarantees not expressible via library composition.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
