# EDR-084: Slice — Range Applied to a Random-Access Composite

**Status:** Accepted

**Date:** 2026-08-05

**Category:** Architecture

**Scope:** Subsystem (Collection Semantics / Data Model)

---

### Context

INDEXING ([EDR-082](./EDR-082-1-based-indexing.md)) defines one-element positional access: `a[i] ≡ a@get(i)`, applying to every random-access `pack` composite (tuples, strings, `Span`, ranges) via an `Indexable`-like trait. RANGE ([EDR-083](./EDR-083-range.md)) defines the inclusive-inclusive `1..N` range value. The remaining gap: **how does Orthon select a contiguous sub-sequence by position — the multi-element form of `@get` — without copying and without a second indexing convention?**

SPAN ([EDR-064](./EDR-064-span.md)) already establishes the memory-view model (non-owning, bounds-checked, no implicit copy), and its model shows slicing (`arr[1..3]`). This EDR formalises slice semantics on top of the Range value and the Span view, resolving the interaction left open between INDEXING, RANGE, and SPAN.

---

### Decision

1. **A slice is a Range applied to a random-access composite** via the `@get`/`Indexable` protocol: `items[1..k]`, `text[1..k]`, `tuple[1..k]`. Indexing `a[i]` is the one-element form (EDR-082); slicing is the multi-element form.
2. **Slicing never copies.** For a contiguous range, slicing a `Span` (or an array-backed collection) produces a **non-owning `Span` view** (EDR-064) — zero-copy, bounds-checked, lifetime-tracked.
3. **Range semantics are inclusive-inclusive** (EDR-083): `items[1..k]` is the first k elements; `len(slice)` is the element count (`b - a + 1`); the programmer never writes `j - i + 1`.
4. **An empty slice is a value** — `items[1..0]` (end < start) is an empty slice with `len == 0`, not a syntax error.
5. **A strided slice is non-contiguous** — `items[1..k].step(2)` yields an iterator of the selected elements (or a materialised collection via `.collect()`), **never a `Span` view** (a span is contiguous by definition).
6. **Multi-dimensional slicing is deferred** to the SPAN concept (Open Q3) / FFI concept (Milestone 8).
7. **`0..<N` at the FFI boundary** follows RANGE (EDR-083) — half-open bounds exist only as an index-translation interop utility, never in application code.

---

### Consequences

**Positive:**
- One positional-access story: `a[i]` (one element) and `a[1..k]` (contiguous run) both decompose to the `@get` protocol.
- Zero-copy slicing — matches SPAN's non-owning view model.
- The `+1` length arithmetic is owned by the language.
- Empty and strided slices have defined, non-surprising semantics.

**Negative:**
- Slicing is only zero-copy for contiguous ranges; strided slicing allocates an iterator/collection.
- Multi-dimensional slicing is deferred (SPAN/FFI), so matrix views are not yet specified.
- Requires the `Indexable`-like trait contract (INDEXING) to be uniform across all random-access composites.

---

### Compliance

1. `items[a..b]` desugars to a `@get`-based range extraction on any random-access composite implementing the `Indexable`-like trait.
2. Contiguous slicing of a `Span`/array produces a non-copying `Span` view.
3. `len(slice) == b - a + 1`; empty slice `end < start` is a value.
4. Strided slicing produces an iterator/collection, not a `Span`.
5. Half-open bounds (`0..<N`) appear only at the FFI boundary.
6. `what/concepts/SLICE.md` defines the canonical semantics.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Slice semantics inside SPAN only | Span is the view type, but slicing also applies to strings and tuples; a separate concept keeps the positional-access story uniform (INDEXING). |
| Copying slice (Python `list[:]`) | Contradicts SPAN's no-implicit-copy principle and wastes memory. |
| Slices with 0-based bounds | Contradicts the single-base rule (EDR-082) and the inclusive norm (EDR-083). |
| Slice as a separate value type distinct from Span | A slice of a contiguous region IS a span view; a distinct type would duplicate Span's contract. |

### Gate Validation

> Required for all Architecture-category EDRs — a new Core Language construct requires the full catalogue of 7 gates (`DECISION_VALIDATION.md` § Gate Selection).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | `items[1..k]` reads as "first k elements"; no copy surprise; length falls out of the norm. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | Slice = Range + `@get`; empty and strided cases precisely defined; no overlap with Span's own contract. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Removing slicing forces manual loops + `@get` per element; the concept adds no new primitive (composes Range + `@get`). |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Level 2 pattern over `@get` (INDEXING) + Range value (EDR-083) + Span view (EDR-064); no layer violation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Zero-copy view vs strategy: view semantics identical across strategies; allocation strategy is a Strategy choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | "A slice is a zero-copy sub-view of a contiguous run, selected by an inclusive range." No "but/except". |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | `items[1..k]` maps directly to mathematics; single inclusive norm removes off-by-one ambiguity in generation; empty/strided edge cases are statically describable. |

**Gates not applied:** none — a new Core Language construct requires the full catalogue.

**Detailed reasoning:** See `DECISION_LOG.md` § Entry: Slice (EDR-084) for the per-gate reasoning trail.

---

### Related Concepts

- `what/concepts/SLICE.md` — Full concept specification
- `what/concepts/RANGE.md` — Range value supplying slice bounds (EDR-083)
- `what/concepts/INDEXING.md` — `@get` protocol, one-element form, `Indexable`-like trait (EDR-082)
- `what/concepts/SPAN.md` — Non-owning memory view; slice target for arrays/contiguous buffers (EDR-064)
- `what/concepts/ITERATOR_PROTOCOL.md` — `IntoIterator`; strided slices iterate (EDR-022)
- `how/concepts/research/essential/RANGE_SLICE.md` — Original research (slice = Range applied to a Span)
- FFI (Milestone 8) — `0..<N` interop and C-facing constructor surface
