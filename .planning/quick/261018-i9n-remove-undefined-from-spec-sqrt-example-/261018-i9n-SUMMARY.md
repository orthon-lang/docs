---
phase: quick-261018-i9n
plan: 01
subsystem: specification-editorial
tags: [contracts, equality, editorial, sqrt-example]
requires: [EDR-056, EDR-017]
provides:
  - "Canonical sqrt contract example without the undefined approximate-equality glyph"
  - "Future approximate-equality operator spelled =~ in EQUALITY docs"
key-files:
  modified:
    - what/concepts/CONTRACTS.md
    - what/GLOSSARY.md
    - how/decision_records/architecture/EDR-056-contracts.md
    - how/concepts/research/important/CONTRACTS.md
    - how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md
    - how/concepts/research/deferrable/SEMANTIC_ANNOTATIONS.md
    - how/concepts/research/deferrable/FRAME_CONDITIONS.md
    - how/concepts/research/deferrable/DIFFERENTIABLE_PROGRAMMING.md
    - what/concepts/EQUALITY.md
    - how/decision_records/architecture/EDR-017-equality.md
decisions:
  - "sqrt postcondition is two plain ensures clauses with explicit inline tolerance"
status: complete
actuals:
  tokens: 3000
  tasks: 3
  commits: 1
plan_head_before: 172e7e1380c105d01bd06156337334a3675fb4b6
plan_head_after: dd6005d866d176a2f04307b55c45db5433de543a
completed: 2026-10-18
---

# Phase quick-261018-i9n Plan 01: Remove undefined approx-equal glyph from spec Summary

The sqrt contract example is now `requires x >= 0.0`, `ensures result >= 0.0`, `ensures abs(result * result - x) <= 0.000001 * x` everywhere it appears, and the future approximate-equality operator is spelled `=~` in the EQUALITY docs only.

## What was done

- **Task 1** (canonical accepted spec): `what/concepts/CONTRACTS.md` (Issue item 1 prose, sqrt block, plus one sentence noting that `ensures result >= 0.0` pins the sign because the relational clause alone admits the negated root), `what/GLOSSARY.md` (Contract entry sqrt block), EDR-056 (single table cell in the phantom-types alternatives row).
- **Task 2** (research drafts): five drafts updated with the same two-clause form or the single `abs` expression in prose.
- **Task 3**: `==~` renamed to `=~` in `what/concepts/EQUALITY.md` (NaN section) and EDR-017 (Consequences). Single commit `dd6005d`.

## Choices recorded

- **Composition-constraint placeholder** (`how/concepts/research/important/CONTRACTS.md`, higher-order contracts block): the line `requires f.ensures(f(x)) ≈ ...` became `requires f.ensures(f(x)) == ...`, glyph replaced with `==`, spacing and trailing comment preserved.
- **JSON string in SEMANTIC_ANNOTATIONS.md**: the `"ensures"` value stays a single JSON string (no arrays, no schema change): `"result >= 0.0 && abs(result * result - x) <= 0.000001 * x"`, the two clauses joined with the existing boolean-and operator.
- **Extra file by grep discovery**: `how/concepts/research/deferrable/DIFFERENTIABLE_PROGRAMMING.md` was not in the original file list; it contained `assert d2f(2.0) ≈ 6.0` in an Orthon code example, now `assert abs(d2f(2.0) - 6.0) <= 0.000001` (D-01 "everywhere").

## Final grep results

- `grep -rIl '≈'` over the spec (excluding `.git`, `.claude`, `.gsd`, `.planning`): only `how/concepts/research/important/INDEXING_ONE_BASED.md` (excluded table cell).
- `==~` outside `.git`, `.claude`, `.gsd` and `.planning/phases|quick|milestones`: no hits. (Hits remain only inside this quick task's own PLAN.md. Note: the plan's automated check, run verbatim, fails spuriously on macOS BSD grep because it prints paths without a leading `./`, so the `.planning/quick/` exclusion regex does not match; the underlying condition holds.)
- `=~` appears only in `what/concepts/EQUALITY.md:89` and `how/decision_records/architecture/EDR-017-equality.md:52`; absent from CONTRACTS.md (both), EDR-056 and GLOSSARY.md.
- HEAD commit touches exactly 10 files, adds/removes no markdown link line, no deletions, last message line is the required Co-Authored-By trailer. The unrelated untracked items were not staged.

## Deviations from Plan

None - plan executed as written. The first `git add` attempt failed under zsh because of an unsplit file-list variable (nothing was staged or committed); it was re-run with explicit paths.

## Known Stubs

None.

## Self-Check: PASSED

All 10 files modified and committed in `dd6005d`; commit hash verified via `git rev-parse HEAD`.
