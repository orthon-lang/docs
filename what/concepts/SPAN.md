# Span (Memory View / Array View)

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via ECR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you provide safe, non-owning access to a contiguous region of memory?

Working with arrays and slices involves:
- **Implicit copying** — slicing in some languages (Python `list[1:3]`) creates a copy; programmer intent (reference vs copy) is unclear.
- **Borrowed views** — `memoryview` in Python, `ArraySegment` in C# — no copying, but no ownership tracking; aliasing bugs.
- **Bounds unsafety** — C-style pointer+length has no bounds checking; out-of-bounds access is UB.
- **Lifetime confusion** — a view can outlive the data it points to (dangling pointer).

The core problem: **a non-owning window into memory needs static or dynamic lifetime tracking** so it cannot dangle, and must make the ownership semantics explicit.

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

A `Span<T>` is a lightweight, non-owning view over a contiguous sequence of `T` elements. It behaves like `&[T]` in Rust or `std::span<T>` in C++20.

```
// Constructing spans
let arr = [1, 2, 3, 4, 5]
let span1: Span<Int> = Span(arr)         // whole array
let span2: Span<Int> = arr[1..3]         // sub-range view (no copy)
let span3: Span<Int> = arr[1..]          // from index 1 to end

// Access
let first = span[0]      // bounds-checked
let slice = span[1..3]   // sub-span (no copy)

// Mutation (if source is mutable)
func process(data: mut Span<Int>)
    data[0] = 42
```

Key features:
- **Reference semantics** — `Span<T>` is always a reference; no implicit copy.
- **Sub-span slicing** — slices of spans produce spans, not copies.
- **Factory from arrays, slices, contiguous buffers** — uniform construction.
- **Mutable variant** — `mut Span<T>` for write access.
- **Lifetime check** — span cannot outlive the backing storage.

## Default Strategy

Slicing an array produces a `Span` (reference, no copy). Span access is bounds-checked in debug and release-safe mode, unchecked in release-unsafe mode. Lifetime of the span is tied to the owner via borrow checking.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Copying slice | Slicing produces a new owned copy (Python `list[:]`). |
| Memory view | Non-copying but no lifetime tracking (Python `memoryview`). |
| GC-tracked view | View kept alive by GC as long as any reference exists (Java `ByteBuffer`). |
| Unsafe span | No bounds checking; `unsafe` block required to construct. |

## Open Questions

1. Should `Span` be a library type or a language built-in with special syntax?
2. Should there be a separate `StrSpan` for string slices?
3. How does span interact with ownership / borrow checking?
4. Should zero-size span construction from `null`/`nil` be allowed?
5. Should spans support multi-dimensional indexing (matrix view)?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
