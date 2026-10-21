---
id: SEED-261021-gof
status: dormant
planted: 2026-10-21
planted_during: Phase 04.3 planning (Specification Integrity Infrastructure), per .planning/STATE.md
trigger_when: when work on task T7 (spec subsystems) starts in Phase 04.3
scope: unknown
---

# SEED-261021-gof: Information hiding at the specification level

## Why This Matters

The design principles have no counterpart of Parnas information hiding (a module hides a decision likely to change). Orthogonality keeps constructs from conflicting but does not bound the blast radius of a change; the cost of cascading changes in the specification follows coupling between decisions. The nearest existing item is task T7 of `.planning/notes/2026-10-17-spec-level-complexity-management.md` (named spec subsystems with owned concepts and bounded cross-subsystem edges).

## When to Surface

**Trigger:** when work on T7 (spec subsystems) starts in Phase 04.3.

Question to settle then: does a spec-level hiding principle belong in `how/DESIGN_PRINCIPLES.md` (the file is locked, so a Tier 1 EDR in the Architecture category), or only in the spec-integrity tooling.

## Scope Estimate

**Unknown.** Run `/gsd-capture --seed --enrich SEED-261021-gof` to estimate effort.

## Breadcrumbs

- `.planning/notes/2026-10-21-complexity-as-redesign-signal.md` (source)
- `.planning/notes/2026-10-17-spec-level-complexity-management.md` (task T7)
- `how/DESIGN_PRINCIPLES.md`

## Notes

Captured from the 2026-10-21 /gsd-explore session. A count of the terms "information hiding" and "deep module" across `why/`, `how/`, `what/` and `notes/` found zero files; this is a keyword search only, so the concept may exist under other wording.
