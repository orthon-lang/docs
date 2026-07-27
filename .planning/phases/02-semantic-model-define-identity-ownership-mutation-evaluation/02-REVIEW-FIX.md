---
phase: 02-semantic-model-define-identity-ownership-mutation-evaluation
fixed_at: 2026-07-27T10:15:00Z
review_path: .planning/phases/02-semantic-model-define-identity-ownership-mutation-evaluation/02-REVIEW.md
iteration: 1
findings_in_scope: 11
fixed: 9
skipped: 2
status: partial
---

# Phase 02: Code Review Fix Report

**Fixed at:** 2026-07-27
**Source review:** .planning/phases/02-semantic-model-define-identity-ownership-mutation-evaluation/02-REVIEW.md
**Iteration:** 1

**Summary:**
- Findings in scope: 11 (fix_scope: all — Critical, Warning, Info)
- Fixed: 9
- Skipped: 2

**Important — branch not yet merged to `main`:** All 9 fixes were applied
and committed atomically on an isolated worktree branch,
`gsd-reviewfix/02-98697`, branched from `main` at commit `3724724`. When
cleanup attempted to fast-forward `main` to this branch, `main` had
advanced by one unrelated commit (`85637be docs(02): extract phase
learnings`, touching only `.planning/STATE.md` and a new
`02-LEARNINGS.md` — no overlap with any file this fixer touched) made
concurrently by another process during this run. Per protocol, a
diverged fast-forward is not force-merged automatically; the branch was
preserved instead of being deleted. A dry-run (`git merge-tree`) confirms
the two histories merge with **no conflicts**. To bring these 9 fixes
onto `main`, run one of:

```bash
git merge gsd-reviewfix/02-98697        # simple merge commit
# or
git rebase main gsd-reviewfix/02-98697 && git checkout main && git merge --ff-only gsd-reviewfix/02-98697
```

Once merged, delete the temp branch: `git branch -D gsd-reviewfix/02-98697`.

## Fixed Issues

### CR-01: Cross-Dimension Consistency table omits the Visibility ↔ Lifetime pair while claiming full 15-pair coverage

**Files modified:** `what/SEMANTIC_MODEL.md`
**Commit:** `c51d127`
**Applied fix:** Inserted the missing Visibility ↔ Lifetime pairwise
analysis as table row 15 ("Orthogonal" — visibility governs who can name
a declaration, lifetime governs how long the value it names persists),
renumbered the existing "Lifetime ↔ All" cross-cutting summary row from
15 to 16 (it was never a genuine pair), and updated both other in-doc
cross-references to "(pair 15)" to "(pair 16)" (the Detail heading and
the Lifetime row of the Design Principles table). Clarified the section's
intro paragraph to state explicitly that rows 1–15 are the fifteen
pairs and row 16 is an additional cross-cutting note, not one of the
fifteen. The "all fifteen pairwise interactions" claims in
`SEMANTIC_MODEL.md` and `EDR-013` are now literally true.

### CR-02: Ownership section's own canonical examples contradict Semantic Invariant 6

**Files modified:** `what/SEMANTIC_MODEL.md`
**Commit:** `02c5f7c`
**Applied fix:** Changed the Ownership section's first canonical example
from `other = data` (bare assignment, no visible marker) to
`other = move data`, using the placeholder `move` keyword the document
already names elsewhere as the syntax-agnostic baseline. This makes the
example consistent with the immediately-following Fresh-value-exemption
example, which correctly requires an explicit marker for transferring a
named binding, and with Semantic Invariant 6's requirement that a
transfer's fact be visible at the transfer site.

### CR-03: EDR-013 cites "EDR-011 (LLM Generability Gate)," but INDEX.md assigns EDR-011 to a different record

**Status:** Already resolved (no commit needed)
**Verification:** Re-read `EDR-013-semantic-model.md:235` — it now cites
`EDR-014 (LLM Generability Gate)`, not EDR-011. Confirmed
`how/decision_records/architecture/EDR-014-llm-generability-gate.md`
exists on disk (the file the reviewer found misfiled as `EDR-011-*` has
since been renumbered) and is correctly registered in `INDEX.md`'s "All
Records" and "By Category → Architecture" tables. Confirmed
`how/decision_records/process/EDR-011-process-inventory.md` is now the
sole holder of the EDR-011 ID (Process category) — no duplicate
assignment remains. Confirmed `how/gates/DECISION_VALIDATION.md:279`
already cites `EDR-014-llm-generability-gate` correctly. This finding was
fixed by prior commits in the repository's history (`git log` shows
`80ca46c feat(quick-260727-flc): renumber orphaned EDR-011 architecture
record to EDR-014` and the subsequent `EDR-14 -> EDR-15` rename) made
before this review-fix run started. No further action was required.

*(Incidental observation, not part of this finding and intentionally not
fixed here per scope discipline: `how/gates/DECISION_LOG.md:16` and
`what/GLOSSARY.md`'s "Decision Log" entry still link to
`../decision_records/process/EDR-014-decision-log.md`, a path that no
longer exists — the Decision Log record was itself renumbered to EDR-015
in the same rename pass that fixed CR-03. This appears to be a residual
broken link from that same renumbering and may be worth a follow-up
finding.)*

### WR-01: EDR-013 claims "exclusive access" is registered in GLOSSARY.md, but no such entry exists

**Files modified:** `what/GLOSSARY.md`
**Commit:** `81fe2de`
**Applied fix:** Added a dedicated "Exclusive Access" glossary entry
(alphabetically positioned in `## E`, between "EDR" and "Execution
Descriptor") defining it as Semantic Invariant 2's single access-control
rule, cross-referencing Semantic Invariant, Declaration Kind, and Binding
Identity, with source citations to `SEMANTIC_MODEL.md` § Semantic
Invariants, § Ownership, § Mutation. EDR-013's Compliance claim is now
accurate.

### WR-02: INDEX.md's Status Summary count is stale

**Files modified:** `how/decision_records/INDEX.md`
**Commit:** `55a79c1`
**Applied fix:** The review found the count stale at "11 vs. 12 actual."
By the time this fixer ran, the document had already grown to 13 actual
Accepted records (EDR-015 had been added between the review and this
fix), so the same underlying staleness defect now manifested as "12 vs.
13." Updated the Status Summary's `Accepted` count from 12 to 13,
verified against both the "All Records" table (13 rows) and the sum of
"By Category" tables (Architecture 4 + Process 8 + Quality 1 = 13).

### WR-03: EDR-013's "one non-trivial coupling" framing understates the model's own Cross-Dimension Consistency table

**Files modified:** `how/decision_records/architecture/EDR-013-semantic-model.md`
**Commit:** `9d16456`
**Applied fix:** Changed "the one non-trivial coupling identified" to
"the sharpest of several documented couplings," per the review's first
suggested option — a minimal wording qualification that brings EDR-013's
characterization back into agreement with `SEMANTIC_MODEL.md`'s
Cross-Dimension Consistency table, which documents several pairs (8, 9,
10, 15/16) with non-bare-"Orthogonal" relationship labels.

### WR-04: GLOSSARY.md structural defects — duplicate/misfiled headings and broken alphabetical order

**Files modified:** `what/GLOSSARY.md`
**Commit:** `0774ebd`
**Applied fix:** All five specific defects listed in the finding were
corrected:
- Removed the duplicate empty `### Data` stub heading, keeping the one
  populated entry.
- Removed the duplicate `### Explicit Optimization` entry (kept one copy).
- Merged `## D (cont.)` (Decision Log, Decision Validation) back into
  `## D` in correct alphabetical position (Data, Data Modifier, Decision
  Log, Decision Validation, Declaration Kind, Declaration Model,
  Dependency Flow, Deterministic Behavior).
- Moved "Program Enricher" from `## I` to its correct home under `## P`
  (after Principle of Least Astonishment).
- Moved "Stable Mental Model" and "Standard Library" from `## V` (where
  they were misfiled) to their correct alphabetical position under
  `## S` (both start with "St-", sorting between "Sequence" and
  "Structural Equality").
- Reordered `## E` so "Execution Program" precedes "Explicit
  Optimization."

Verified via heading diff: original file had 60 `###` headings with 2
duplicated names (Data, Explicit Optimization); the fixed file has 58
unique headings with zero duplicates, and all original content was
preserved (only relocated), confirmed via `git diff --stat` showing a
balanced insertion/deletion count consistent with reordering rather than
content loss.

### IN-01: Unsourced "~95%" statistic repeated without citation or methodology

**Files modified:** `what/SEMANTIC_MODEL.md`, `how/decision_records/architecture/EDR-013-semantic-model.md`
**Commit:** `cb3d2c8`
**Applied fix:** Applied the review's first suggested option — softened
both occurrences to qualitative language. `SEMANTIC_MODEL.md`: "The vast
majority of Orthon code — an estimated 95% —" became "The large majority
of ordinary-value code". `EDR-013`: "~95% of ordinary-value code" became
"the large majority of ordinary-value code". No load-bearing-sounding
unsourced number remains in either document.

### IN-02: Inconsistent pair-name ordering between table row and "Detail" callout

**Files modified:** `what/SEMANTIC_MODEL.md`
**Commit:** `c1820d7`
**Applied fix:** Renamed "Detail: Visibility ↔ Ownership (pair 8)" to
"Detail: Ownership ↔ Visibility (pair 8)" to match the table row's
dimension order, consistent with every other Detail heading in the
document.

### IN-04: Top-of-document link text doesn't match its target heading

**Files modified:** `what/SEMANTIC_MODEL.md`
**Commit:** `a19bd50`
**Applied fix:** Changed the status banner's link text from "Design
Principles verification" to "Relationship to Design Principles" to
exactly match the heading it points to (`## Relationship to Design
Principles`).

## Skipped Issues

### CR-03: EDR-013 cites "EDR-011 (LLM Generability Gate)," but INDEX.md assigns EDR-011 to a different record

**File:** `how/decision_records/architecture/EDR-013-semantic-model.md:235`, `how/decision_records/INDEX.md:24,59`
**Reason:** Already resolved by prior commits before this fix run started
(see "Fixed Issues" section above for full verification detail). Listed
here per the finding's ID for completeness/traceability; no code change
was made or needed.

### IN-03: Borderline Why-layer motivational aside inside the What-layer document

**File:** `what/SEMANTIC_MODEL.md:143-146`
**Reason:** The finding's own Fix guidance states "No immediate action
required" — it is a forward-looking style note ("worth watching as the
document grows"), not a defect requiring an edit in this pass. Left
unchanged per the finding's own recommendation.

---

_Fixed: 2026-07-27_
_Fixer: Claude (gsd-code-fixer)_
_Iteration: 1_
