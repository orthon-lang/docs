---
quick_id: 260720-dla
status: planned
---

# Quick Task 260720-dla: Broaden Phase 2 scope to all CONCERNS.md items

**Description:** Phase 2 (Concerns Remediation) currently scopes itself to only the "critical and high-severity blockers" from CONCERNS.md. The user wants Phase 2 to resolve ALL concerns in CONCERNS.md, not just critical/high ones. Update ROADMAP.md (and REQUIREMENTS.md traceability) accordingly.

## Context

`.planning/codebase/CONCERNS.md`'s Summary Table lists 12 issues. Cross-checking against the current DEBT-01..10 requirements and Phase 1's PROC-01/02:

- Already covered by Phase 2 (DEBT-01..10): empty architecture files, deprecated `_adr.md`, superseded `EXECUTION_IMAGE.md`, missing date stamps, EDR numbering gap, sparse `FITNESS_FUNCTIONS.md`, minimal `DOCUMENTATION_PRINCIPLES.md`.
- Already owned by Phase 1 (PROC-01/02): DRAFT concept acceptance gate, Concept Design Review process undefined.
- **Missed even under the old "critical/high" scoping** (inconsistency the user is flagging): concept depth inconsistency (High), milestone ownership undefined (Critical), template compliance not enforced (High).
- **Missed medium/low items**: cross-reference link fragility, glossary maintenance burden, large-monolithic-document versioning, documentation growth/info-architecture planning.

## Task 1: Add DEBT-11..17 to REQUIREMENTS.md

**Files:** `.planning/REQUIREMENTS.md`
**Action:**
1. Append 7 new items under "Concerns Remediation" section (after DEBT-10):
   - DEBT-11: concept-depth audit against `_concept.md` template across all 22 concepts; under-developed concepts flagged/remediated
   - DEBT-12: ownership + target metadata assigned to tracked TODO/requirement items
   - DEBT-13: cross-reference/"See also" link audit performed repo-wide; broken/superseded links fixed; validation approach documented
   - DEBT-14: template-compliance enforcement mechanism applied (audit run, gap list produced)
   - DEBT-15: glossary maintenance workflow documented
   - DEBT-16: versioning/change-log policy documented for large monolithic docs
   - DEBT-17: documentation growth / information-architecture plan documented
2. Add corresponding rows to the Traceability table (DEBT-11..17 → Phase 2 → Pending).
3. Update `**Coverage:**` totals (49 → 56 v1 requirements) and Phase 2 summary line (10 → 17 requirements).
4. Update the "Last updated" footer line.

**Verify:** `grep -c "DEBT-" .planning/REQUIREMENTS.md` reflects 17 item definitions + 17 traceability rows; coverage math is internally consistent.
**Done:** File edited, counts consistent.

## Task 2: Rewrite Phase 2 section in ROADMAP.md

**Files:** `.planning/ROADMAP.md`
**Action:**
1. Change Phase 2 **Goal** to drop the "critical and high-severity blockers" framing and state that ALL CONCERNS.md findings are resolved.
2. Update **Requirements** line to list DEBT-01 through DEBT-17.
3. Add success criteria 6-9 covering: concept-depth audit, milestone ownership, cross-reference audit, template-compliance enforcement, glossary workflow, large-doc versioning, growth/info-architecture plan (grouped sensibly, doesn't need to be 1:1 with each DEBT item).

**Verify:** Phase 2 section references DEBT-01..17; no other phase's requirement list duplicates DEBT-11..17.
**Done:** Section edited.

## Task 3: Write SUMMARY.md, update STATE.md, commit

**Files:** `.planning/quick/260720-dla-.../260720-dla-SUMMARY.md`, `.planning/STATE.md`
**Action:** Standard quick-task close-out — summary with status: complete, STATE.md Quick Tasks Completed row + Last activity line, single commit.
**Verify:** `git log -1 --stat` shows ROADMAP.md, REQUIREMENTS.md, STATE.md, and quick-task artifacts.
**Done:** Committed.
