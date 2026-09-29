---
task: 260929-b2n
type: quick
mode: documentation
status: complete
completed: 2026-09-29
subsystem: language-spec / iteration
tags: [edr, iteration-loop, for-expression, ownership, optional, amendment]
requirements:
  - "EDR-053 OQ2 (break-with-value in for) — resolved YES"
  - "EDR-053 OQ3 (for-else) — resolved NO"
  - "EDR-053 OQ4 (iteration ownership: consume vs borrow) — resolved borrow-by-default"
key-files:
  created:
    - how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md
  modified:
    - what/concepts/ITERATION_LOOP.md
    - how/decision_records/INDEX.md
    - how/decision_records/architecture/EDR-053-iteration-loop.md
decisions:
  - "EDR-094 records the 2026-09-28 ITERATION_LOOP concept review (human sign-off 2026-09-29) as an Architecture amending-EDR over EDR-053."
  - "for is an expression yielding Optional<T> in value position (break v/return v -> Some(v); run-off-end -> None); statement position yields no value; .find stays the simple-predicate combinator."
  - "for-else rejected; the search-and-not-found pattern is covered by Optional + match."
  - "Iteration is borrow-by-default with three source-side ownership markers (coll shared &T; &mut coll exclusive &mut T; $coll move T); structural mutation is not a for-mode; interior mutability not introduced in v0.1."
metrics:
  tasks: 3
  files: 4
  commits: 3
actuals:
  tasks: 3
  commits: 3
---

# Quick Task 260929-b2n: EDR-094 (For-Expression and Iteration Ownership) Summary

**One-liner:** Recorded the human-signed ITERATION_LOOP concept review as Architecture
EDR-094 amending EDR-053 — `for` becomes an expression yielding `Optional<T>`,
`for`-`else` is rejected, and iteration is borrow-by-default with source-side ownership
markers — then wired the resolution into the concept spec and the EDR registry.

## What Was Built

**Task 1 — EDR-094 authored** (commit `0a855c3`)
`how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md`,
following the Architecture EDR template and the EDR-092 amending-EDR style: Accepted /
2026-09-29 / Architecture / Subsystem header, the `Reviewed-by: mniedre · Date:
2026-09-29 · Verdict: LOCKED` sign-off line, and an `Amends: EDR-053` header stating it
resolves OQ2/OQ3/OQ4 and partially supersedes the statement-only reading of `for`.
Four numbered Decision items (Optional `for`-expression; `for`-`else` rejection;
borrow-by-default ownership with the verbatim iteration matrix and the binding/grant/rent
authority frame with five properties; B1–B4 boundary flags as deferrals), Consequences,
Compliance, an Alternatives Considered table carrying anti-memory of every rejected form,
and a full seven-gate Architecture validation table (all Pass).

**Task 2 — ITERATION_LOOP.md updated** (commit `30169ed`)
Added EDR-094 (and NULL_SAFETY) to the See-also header; added a "`for` as an Expression
— Optional Search" Model subsection with a worked `orthon` example; added an "Iteration
and Ownership" Model subsection with the verbatim matrix, source-side markers, and
authority frame; marked Open Questions 2/3/4 resolved in the existing strike-through
style with EDR-094 references; added a 2026-09-29 Decision History entry; checked this
document in Affected Documents and added explicit Phase 5 (SYNTAX.md / `what/syntax/`)
and Phase 7 (EXECUTION_MODEL.md) deferral notes with their boxes left unchecked.

**Task 3 — INDEX.md + EDR-053 back-pointer** (commit `794510f`)
Added the EDR-094 row to both the All Records and the By Category > Architecture tables;
incremented the Accepted count 78 → 79; refreshed the Last-updated note to 2026-09-29
with an EDR-094 summary (preserving prior note text); added an `Amended by: EDR-094`
header line under `**Scope:**` in EDR-053 and a 2026-09-29 entry to EDR-053's Amendments
section (bidirectional cross-link).

## Deviations from Plan

None — plan executed exactly as written. All three `<verify>` checks printed
`TASK1_OK` / `TASK2_OK` / `TASK3_OK`.

Minor authoring choice (within plan latitude): the See-also header also links
`NULL_SAFETY.md` (EDR-018) since Task 2 required cross-referencing the Optional type;
the target file was confirmed to exist before linking.

## Out-of-Scope Guard Honored

`git status --short` before the final commit showed only the four in-scope files.
`what/SYNTAX.md`, `what/syntax/*`, and `what/EXECUTION_MODEL.md` were NOT touched; they
are recorded as Phase 5 / Phase 7 deferrals with unchecked boxes in ITERATION_LOOP.md's
Affected Documents.

## Cross-References Verified

- EDR-094 ↔ EDR-053 (amends / amended-by, bidirectional) — both target files exist.
- EDR-094 ↔ ITERATION_LOOP.md (See also + Decision History).
- EDR-094 registered in INDEX.md All Records + Architecture tables.
- NULL_SAFETY.md link target confirmed present.

## Self-Check: PASSED

- `how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md` — FOUND
- Commit `0a855c3` (EDR-094) — FOUND
- Commit `30169ed` (ITERATION_LOOP.md) — FOUND
- Commit `794510f` (INDEX.md + EDR-053) — FOUND
- All three automated `<verify>` checks — PASS
