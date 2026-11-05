---
title: Dialectical Axis Tree and Tradeoff Ledger
date: 2026-11-06
status: complete
---

# Quick Task 261106-3ja: Dialectical Axis Tree and Tradeoff Ledger

## Outcome

Added the dialectical axis tree and its overlay to Orthon's design
documentation, and made the "new contradiction" an explicit field in the
decision trail.

## Changes

1. **Axis tree** — `why/DESIGN_INFLUENCES.md` now carries "The Axes of
   Language Design": 14 axis families with their witness positions
   (external view, no Orthon positions), plus a pointer to the overlay.
2. **Method** — `how/process/DIALECTICAL_METHOD.md`: three sources,
   completeness fixpoint, three steps (reconstruct → overlay → name
   trade-off/new contradiction), acceptance bar, relocation hypothesis,
   stop rule.
3. **Ledger** — `what/DIALECTICAL_TRADEOFFS.md`: Section A (committed
   axes) with the concurrency axis filled as the worked pilot row;
   Section B (open axes) with five gaps routed to Decision Pipeline Q1.
4. **EDR-102 (Process, Proposed)** — adds a mandatory `New contradiction`
   field to all three EDR templates and a gate criterion to
   `_language-design.md`; leaves the locked `DESIGN_PRINCIPLES.md`
   untouched. Indexed in `INDEX.md` (both tables, summary, last-updated).
5. **Document map** — `AGENTS.md` registers the two new documents.

## Commits

- `c007106` docs: add dialectical axis tree to design influences
- `d15098a` docs: add dialectical method and tradeoff ledger
- `3c88538` docs: add process EDR-102 for new-contradiction field

## Notes

- EDR-102 is **Proposed** — the author should review and flip to
  **Accepted** (Tier 3 has no mandatory human sign-off, but review is
  recommended before acceptance).
- Pre-existing INDEX defect noticed but not fixed (out of scope): EDR-099
  and EDR-100 are missing from the "All Records" table; EDR-101 sits on a
  line separated from the main table body. Flagging for a follow-up.
- The open-axes register is the seed list for the next overlay pass; the
  concurrency row demonstrates the required five-cell completeness.
