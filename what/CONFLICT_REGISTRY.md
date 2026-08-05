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
| — | — | — | — | — |

## Resolved Conflicts

<!-- To be filled during Phase 6 — one row per resolved conflict -->

| ID | Concepts Involved | Resolution | EDR |
|----|-------------------|------------|-----|
| C-001 | ITERATION_LOOP (EDR-053), ITERATOR_PROTOCOL (EDR-022), SPAN (EDR-064), INDEXING (EDR-082) | Adopted 1-based as the single base and inclusive-inclusive `1..N` as the single range norm; canonical index iteration `for i in 1..len(array)`; `enumerate` from 1; Span single-base 1-based. Amendments to EDR-053/EDR-022/EDR-064 and GLOSSARY examples applied at acceptance. | [EDR-082](../how/decision_records/architecture/EDR-082-1-based-indexing.md) |

**Goal:** Zero `Open` entries at end of Phase 6.
