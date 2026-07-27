---
phase: quick-260727-flc
plan: 01
subsystem: decision-records
tags: [edr, index, cross-reference-fix, tech-debt]
dependency-graph:
  requires: []
  provides: [EDR-014-llm-generability-gate]
  affects:
    - how/decision_records/INDEX.md
    - how/gates/DECISION_VALIDATION.md
    - how/gates/_language-design.md
    - how/architecture/FITNESS_FUNCTIONS.md
    - how/decision_records/architecture/EDR-013-semantic-model.md
    - notes/llm-generability-gate.md
    - TODO.md
tech-stack:
  added: []
  patterns: ["git mv rename to preserve history", "single-line targeted cross-reference repoint"]
key-files:
  created:
    - how/decision_records/architecture/EDR-014-llm-generability-gate.md
  modified:
    - how/decision_records/INDEX.md
    - how/gates/DECISION_VALIDATION.md
    - how/gates/_language-design.md
    - how/architecture/FITNESS_FUNCTIONS.md
    - how/decision_records/architecture/EDR-013-semantic-model.md
    - notes/llm-generability-gate.md
    - TODO.md
decisions:
  - "Appended EDR-014 as the last row in INDEX.md's All Records table (not inserted chronologically by date) — matching the table's existing non-strict-date-order precedent (EDR-013 already appears after EDR-012 despite EDR-011 predating both)."
metrics:
  duration: 10min
  completed: 2026-07-27
status: complete
---

# Quick Task 260727-flc: Renumber Duplicate EDR-011 (Architecture) to EDR-014 Summary

Renumbered the orphaned architecture decision record that collided with the canonical Process-category EDR-011, repointing all six live cross-references and adding it to the decision journal INDEX.

## What Was Built

Two files in `how/decision_records/architecture/` both claimed the ID "EDR-011": the canonical, correctly-indexed `EDR-011-process-inventory.md` and an orphaned `EDR-011-llm-generability-gate.md` that predated Phase 2 and was never listed in INDEX.md. This collision was surfaced by `02-VERIFICATION.md` because EDR-013's "Relationship to Other Records" table cited "EDR-011 (LLM Generability Gate)" ambiguously against the authoritative journal.

Resolved by renaming the orphaned file to `EDR-014-llm-generability-gate.md` (the next free ID per INDEX.md), updating its self-referencing heading, repointing six live cross-references, and adding EDR-014 as a first-class entry in INDEX.md's All Records table, Architecture category sub-table, and Status Summary count.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Renumber the orphaned architecture EDR from EDR-011 to EDR-014 | 80ca46c, 9bd7614 | how/decision_records/architecture/EDR-011-llm-generability-gate.md → EDR-014-llm-generability-gate.md (git mv + heading edit) |
| 2 | Repoint every live cross-reference to the renamed EDR-014 file | ad87edc | how/gates/DECISION_VALIDATION.md, how/gates/_language-design.md, how/architecture/FITNESS_FUNCTIONS.md, how/decision_records/architecture/EDR-013-semantic-model.md, notes/llm-generability-gate.md, TODO.md |
| 3 | List EDR-014 in INDEX.md — All Records, Architecture category, Status Summary, footer date | 8e504e5 | how/decision_records/INDEX.md |

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Recovered a heading edit dropped by a failed multi-path `git add`**
- **Found during:** Post-execution verification (self-check step)
- **Issue:** After `git mv`-renaming the file and editing its heading from "EDR-011" to "EDR-014", the follow-up `git add <old-path> <new-path>` command included the now-nonexistent old path. Git aborted the entire multi-pathspec add with a fatal error and staged nothing new, so commit 80ca46c captured only the rename (0 insertions/0 deletions) — the heading edit remained an uncommitted working-tree change.
- **Fix:** Detected via a final `git status --short` check showing the file as modified after all three tasks were "complete." Staged and committed the dropped content change in a standalone fix commit (9bd7614) rather than amending 80ca46c, per the no-amend git safety policy.
- **Files modified:** how/decision_records/architecture/EDR-014-llm-generability-gate.md
- **Commit:** 9bd7614

The INDEX.md footer date already read "2026-07-27" (today's date) at execution time, so no footer edit was needed, matching the plan's explicit contingency for that case. No other deviations — remaining tasks executed exactly as written.

## Verification Results

All six plan-level verification checks passed:

1. Rename completed: `EDR-014-llm-generability-gate.md` exists, `EDR-011-llm-generability-gate.md` no longer exists.
2. Heading updated: `head -1` on the renamed file returns `# EDR-014: LLM Generability Gate`.
3. Zero remaining `EDR-011` citations across the six repointed files.
4. INDEX.md: `EDR-014` appears exactly twice (All Records + Architecture category); Status Summary Accepted count reads `12`.
5. Canonical `how/decision_records/process/EDR-011-process-inventory.md` unchanged — still the sole file claiming ID EDR-011.
6. `git log --follow` on the renamed file shows continuous history back through the pre-rename commit, confirming `git mv` preserved history rather than delete+create.

## Scope Boundary Respected

`.planning/` was not touched by any of the three fix commits, including `.planning/REQUIREMENTS.md`'s incidental "EDR-011" mention in its "Orthon for LLM" deferred-items section — per the plan's explicit out-of-scope declaration, these are historical planning-session records analogous to git history.

## Known Stubs

None — this is a pure documentation cross-reference fix with no code or data flow.

## Self-Check: PASSED

- FOUND: how/decision_records/architecture/EDR-014-llm-generability-gate.md
- FOUND: commit 80ca46c (rename)
- FOUND: commit ad87edc (cross-reference repoint)
- FOUND: commit 8e504e5 (INDEX.md update)
- FOUND: commit 9bd7614 (dropped heading edit recovered)
- Confirmed `git show HEAD:how/decision_records/architecture/EDR-014-llm-generability-gate.md` head line reads `# EDR-014: LLM Generability Gate` — committed content matches working tree, no residual uncommitted changes.
