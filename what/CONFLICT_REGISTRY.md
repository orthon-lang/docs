# Conflict Registry

> **⚠️ DRAFT — Placeholder for Phase 6.**
> This document tracks all concept-boundary conflicts discovered
> during cross-cutting analysis, and their resolution status.
>
> **Status:** Placeholder — to be filled during Phase 6 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 6,
> [`CROSS_CUTTING.md`](CROSS_CUTTING.md)

---

## Open Conflicts

<!-- To be filled during Phase 6 — one row per unresolved conflict -->

| ID | Concepts Involved | Nature of Conflict | Resolution Plan | Status |
|----|-------------------|-------------------|-----------------|--------|
| C-001 | ITERATION_LOOP (EDR-053), ITERATOR_PROTOCOL (EDR-022), SPAN (EDR-064), INDEXING (draft) | Accepted loop/iterator/span concepts use 0-based index ranges (`for i in 0..len(array)`, `enumerate`, `span[0]`); 1-based indexing (INDEXING_ONE_BASED) makes index 0 invalid — direct contradiction. | Adopt inclusive-inclusive `1..N` as the single range norm; canonical index iteration `for i in 1..=len(array)`; `enumerate` from 1 (B3); Span single-base 1-based (B2); amend EDR-053/EDR-022/EDR-064 semantics and examples at EDR-082 acceptance. | Decision made 2026-08-05 (B2 + B3 + B4); amendment pending EDR-082 |

## Resolved Conflicts

<!-- To be filled during Phase 6 — one row per resolved conflict -->

| ID | Concepts Involved | Resolution | EDR |
|----|-------------------|------------|-----|
| — | — | — | — |

**Goal:** Zero `Open` entries at end of Phase 6.
