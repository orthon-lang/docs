---
quick_id: 260720-dla
status: complete
---

# Quick Task 260720-dla: Broaden Phase 2 scope to all CONCERNS.md items — Summary

**Completed:** 2026-07-20

## What changed

Phase 2 ("Concerns Remediation") previously scoped itself to only "critical and high-severity blockers" from `.planning/codebase/CONCERNS.md`. In practice it had already drifted from even that narrower scope (it covered some Medium items like the EDR numbering gap while missing High/Critical items like "concept depth inconsistency," "milestone ownership undefined," and "template compliance not enforced").

Cross-checked all 12 items in CONCERNS.md's Summary Table against existing requirements:
- DEBT-01..10 already cover: empty architecture files, deprecated `_adr.md`, superseded `EXECUTION_IMAGE.md`, missing date stamps, EDR numbering gap, sparse `FITNESS_FUNCTIONS.md`, minimal `DOCUMENTATION_PRINCIPLES.md`.
- PROC-01/02 (Phase 1) already own: DRAFT concept acceptance gate, Concept Design Review process.
- Everything else was unmapped: concept depth inconsistency, milestone ownership, cross-reference link fragility, template compliance enforcement, glossary maintenance, large-doc versioning, documentation growth/info-architecture.

## Changes made

1. **`.planning/REQUIREMENTS.md`**: Added DEBT-11 through DEBT-17 covering the previously-unmapped concerns; added traceability rows; updated coverage counts (49→56 total, Phase 2 10→17 requirements).
2. **`.planning/ROADMAP.md`**: Rewrote Phase 2's Goal statement to drop the "critical and high-severity" framing in favor of "every finding in CONCERNS.md ... is resolved or has an explicit, documented remediation." Extended Requirements list to DEBT-01..17. Added success criteria 6-9 covering the new DEBT items.
3. **`.planning/STATE.md`**: Added Quick Tasks Completed row, updated Last activity line.

## Notes

- Requirements already owned by Phase 1 (DRAFT acceptance gate, review process) were deliberately NOT duplicated into Phase 2 — they stay where they are.
- Phase 3's success criterion 1 (13 named concepts, 7-section template) already overlaps partially with new DEBT-11 (all 22 concepts); DEBT-11 is scoped as an audit + remediation for the *remaining* under-developed concepts outside Phase 3's 13, so the two don't duplicate scope.
