---
title: Apply the comptime phase-marker decision (bake at call site)
date: 2026-08-23
priority: medium
status: pending
---

## What

Apply the 2026-08-23 exploration outcome for the comptime phase marker to
the accepted documents:

- `what/concepts/COMPILE_TIME_EXECUTION.md` — resolve/annotate OQ 4–9
  with the exploration notes (partially recorded 2026-08-23: OQ7, OQ9,
  Synthesis, Decision History).
- `what/concepts/DECLARATION_BY_ASSIGNMENT.md` — OQ3 resolved: no
  separate `const` concept (the phase marker lives at the expression
  level).
- Draft the EDR amendment to EDR-031: Decision item 1 (`comptime`
  keyword) and item 6 (marker at the definition site) → keyword candidate
  `bake`, marker at the call site. Requires Human Sign-off per AGENTS.md
  §7.4 (non-automatable gate).
- If the keyword is confirmed, update `what/GLOSSARY.md` (Comptime
  entry) and the Phase 5 syntax records.

## Why

The exploration (`$gsd-explore` 2026-08-23, see
[[2026-08-23-comptime-phase-marker-bake]]) converged on: comptime is an
orthogonal phase axis (not a fifth execution_context policy); the phase
marker belongs at the call site (colourless functions); `bake` is the
leading keyword candidate. This revises EDR-031's accepted syntax-level
decisions and must not be treated as final without the EDR amendment and
Human Sign-off.

## Suggested action

1. Confirm the keyword (`bake` vs `consteval`) and the call-site position
   in Phase 5.
2. Draft `EDR-NNN-comptime-phase-marker-bake.md` in
   `how/decision_records/architecture/` using
   `how/templates/_edr-architecture.md`, amending EDR-031 Decision items
   1 and 6.
3. Get Human Sign-off (Reviewed-by / Date / Verdict) — non-automatable.
4. After acceptance: update `COMPILE_TIME_EXECUTION.md`,
   `DECLARATION_BY_ASSIGNMENT.md`, `GLOSSARY.md`, and the Phase 5 syntax
   records.
