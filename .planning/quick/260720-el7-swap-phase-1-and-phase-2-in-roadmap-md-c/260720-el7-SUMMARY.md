---
phase: quick-260720-el7
plan: 01
subsystem: planning-docs
tags: [roadmap, requirements, state, reordering]
dependency-graph:
  requires: []
  provides: [swapped-phase-1-2-order]
  affects: [.planning/ROADMAP.md, .planning/REQUIREMENTS.md, .planning/STATE.md]
tech-stack:
  added: []
  patterns: [scoped-edit-tool-usage, no-full-file-rewrite]
key-files:
  created: []
  modified:
    - .planning/ROADMAP.md
    - .planning/REQUIREMENTS.md
    - .planning/STATE.md
decisions:
  - "Swapped execution order: Phase 1 is now Concerns Remediation (depends on nothing), Phase 2 is now Foundations, Process & Vision (depends on Phase 1), per explicit user request to run tech-debt leftovers before process/vision work."
metrics:
  duration: "~10 min"
  completed: 2026-07-20
status: complete
---

# Quick Task 260720-el7: Swap Phase 1 and Phase 2 in ROADMAP.md Summary

Swapped the execution order of the two initial roadmap phases so tech-debt/architecture-spec remediation runs before the process/vision foundations work, and propagated the renumbering across ROADMAP.md, REQUIREMENTS.md, and STATE.md so all three documents stay internally consistent.

## What Changed

**`.planning/ROADMAP.md`:**
- Overview paragraph narrative reordered: "first clear the flagged architecture/tech-debt blockers (Phase 1), then fix the *process* gap ... (Phase 2)"
- `## Phases` summary list: Concerns Remediation bullet now listed first (Phase 1), Foundations/Process/Vision bullet now second (Phase 2)
- Phase Details section: the two full phase bodies were relocated and renumbered — `### Phase 1: Concerns Remediation` (Depends on: Nothing, first phase) now physically precedes `### Phase 2: Foundations, Process & Vision` (Depends on: Phase 1). All Goal/Requirements/Success Criteria/Plans text preserved byte-for-byte within each section — only heading numbers, order, and Depends-on lines changed.
- `### Phase 3` Depends-on line updated from "Phase 1 (requires the acceptance gate...)" to "Phase 2 (requires the acceptance gate...)" since PROC-01/02 now live in the renumbered Phase 2.
- `## Progress` table: first two rows swapped to list Concerns Remediation then Foundations, Process & Vision, both still "0/TBD, Not started".
- Phases 3, 4, 5 content otherwise untouched.

**`.planning/REQUIREMENTS.md`:**
- Traceability table: PROC-01..04 and VISION-01..03 rows changed from "Phase 1" to "Phase 2"; DEBT-01..17 rows changed from "Phase 2" to "Phase 1". CONCEPT-*, ANTIPAT-*, XCUT-01, NAME-01, FREEZE-* rows untouched. Coverage totals unchanged (57/57, 100%, unmapped: 0).
- `**Phase summary:**` block: Concerns Remediation (17 requirements, DEBT-01..17) now listed as Phase 1, before Foundations, Process & Vision (7 requirements, PROC-01..04/VISION-01..03) as Phase 2.

**`.planning/STATE.md`:**
- `**Current focus:**` changed to "Phase 1 — Concerns Remediation"
- `Phase: 1 of 5 (...)` changed to "Concerns Remediation"
- Decisions bullet rewritten to describe the 2026-07-20 swap: Concerns Remediation is now Phase 1 (no dependency), Phase 2 (Foundations/Process) now depends on Phase 1, both still precede Phase 3's concept-design work.
- Blockers/Concerns bullets updated: the ANTIPAT-11/PROC-01-02 dependency bullet now cites Phase 2; the DEBT-01..04/architecture-spec bullet now cites Phase 1.
- Quick Tasks Completed table, Deferred Items, and Session Continuity left untouched (historical records).

## Verification

Both tasks' automated verification blocks (grep-based, checking heading order, Depends-on values, table rows, and cross-file phase-mapping consistency) passed. Ran `git log --oneline` to confirm both task commits landed with the expected file scope.

## Deviations from Plan

None — plan executed exactly as written across both tasks.

## Self-Check: PASSED

- FOUND: .planning/ROADMAP.md (Phase 1 = Concerns Remediation, Phase 2 = Foundations/Process/Vision, in that physical order)
- FOUND: .planning/REQUIREMENTS.md (DEBT-01..17 → Phase 1; PROC-01..04/VISION-01..03 → Phase 2)
- FOUND: .planning/STATE.md (Current focus/Position and Decisions/Blockers bullets cite corrected phase numbers)
- FOUND commit 8d6ca78: feat(quick-260720-el7): swap Phase 1/Phase 2 order in ROADMAP.md
- FOUND commit 0a9ac35: feat(quick-260720-el7): propagate Phase 1/2 renumbering into REQUIREMENTS.md
