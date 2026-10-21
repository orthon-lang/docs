---
phase: quick-261021-ir3
plan: 01
subsystem: contracts-spec
tags: [edr, contracts, sign-off, documentation]
requires: []
provides:
  - EDR-097 Accepted with the author's sign-off
  - CONTRACTS amendment (Stage A) applied across nine documents
affects:
  - what/concepts/CONTRACTS.md
  - what/CORE_CONCEPTS.md
  - what/GLOSSARY.md
  - how/architecture/NAME_RESOLUTION.md
  - how/decision_records/INDEX.md
  - how/gates/DECISION_LOG.md
key-files:
  modified:
    - how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md
    - what/concepts/CONTRACTS.md
    - what/CORE_CONCEPTS.md
    - what/GLOSSARY.md
    - how/architecture/NAME_RESOLUTION.md
    - how/syntax/TYPE_ANNOTATION_SYNTAX.md
    - how/syntax/FUNCTION_RETURN_SYNTAX.md
    - how/decision_records/INDEX.md
    - how/gates/DECISION_LOG.md
decisions:
  - "Sign-off recorded verbatim from the author's chat statement: Reviewed-by: mniedre, Date: 2026-10-21, Verdict: LOCKED"
metrics:
  duration: "about 25 minutes"
  completed: 2026-10-21
status: complete
commits: 4
plan_head_before: 96faa5c7638d11a3ac486d1d3ba56978e8dc036c
plan_head_after: 8f8464dbd4e04595fbc643aff7b01bd6385a11bf
actuals:
  tokens: 30000
  tasks: 3
  commits: 4
---

# Phase quick-261021-ir3 Plan 01: Accept EDR-097 and apply Stage A Summary

EDR-097 is Accepted on the author's recorded sign-off, and the CONTRACTS concept, registry, glossary, name-resolution spec, two syntax hypotheses, EDR index and decision log all describe one model: the result is named in the signature (`-> result: Float`), visible only in `ensures`, and `old` is deferred out of v0.1.

## Commits

| Commit | Hash | Files |
|--------|------|-------|
| A: accept EDR-097 on the author's sign-off | 995c2c2 | EDR-097 |
| B: apply EDR-097 Stage A to the CONTRACTS concept | 65538e8 | what/concepts/CONTRACTS.md |
| Task 2: align CORE_CONCEPTS, GLOSSARY, NAME_RESOLUTION and syntax notes | 6bc5104 | CORE_CONCEPTS, GLOSSARY, NAME_RESOLUTION, TYPE_ANNOTATION_SYNTAX, FUNCTION_RETURN_SYNTAX |
| Task 3: register EDR-097 in INDEX and log the gates | 8f8464d | INDEX.md, DECISION_LOG.md |

All four carry the `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>` trailer. `git rev-list --count 96faa5c..HEAD` = 4.

## Files changed

- **EDR-097**: Status Accepted; header and bottom blockquote both carry `Reviewed-by: mniedre · Date: 2026-10-21 · Verdict: LOCKED`, with the note that it was recorded from the author's explicit statement in chat and not self-certified; draft banner replaced by an `Accepted 2026-10-21.` note; Compliance item 4 lost "after sign-off"; Gate Validation lead-in put in past tense plus one sentence on the author's confirmation, closing "Confirm or change..." sentence removed. Decision, Alternatives, Consequences, tables, run text and Resolutions untouched.
- **CONTRACTS.md**: named result in Model, examples (`sqrt`, `withdraw`, `clamp`), Issue item 1, desugaring; "Where Contracts Are Evaluated" rewritten (error example, scope, duplicates, body-declaration warning, unnamed result, successful exit, function-value error); `old` bullet with dedicated error and two workarounds (no operator-form parentheses); new `#### Where Contracts Apply`; failed-predicate bullet and `Contracts as business rules`; Open Questions 1, 3, new 5; Decision History (EDR-097 bullet, rewritten 2026-10-19 entry, TRAIT_CONTRACT_UNIFICATION pointer, EDR-090 cross-reference).
- **CORE_CONCEPTS.md** (CONTRACTS entry only): Summary, EDR row (amended by EDR-097), Open Questions and Resolved Questions rows.
- **GLOSSARY.md**: Contract entry (`-> result: Float`, Source with EDR-056 and EDR-097, See also), new `### Named Result` between Named Before Symbolic and None.
- **NAME_RESOLUTION.md**: Invariant 6, Key Design Decision 6, two Relationships rows, Last updated 2026-10-21.
- **TYPE_ANNOTATION_SYNTAX.md**, **FUNCTION_RETURN_SYNTAX.md**: provisional coupling note (the first also got Last updated 2026-10-21).
- **INDEX.md**: EDR-097 row in both tables, Accepted 80 to 81, Last updated line.
- **DECISION_LOG.md**: appended entry "CONTRACTS amendment (EDR-097)" with three gate subsections, not-applied gates, Overall with R7 recorded as deferred.

## Verification (all run from the repo root against base 96faa5c)

- Task 1 block: sign-off string present twice and Status Accepted (OK); no `implicit variable`, no operator-form `old`, no "no EDR status" (OK); `-> result: ` count at least 6, `Where Contracts Apply`, `Contracts as business rules`, `EDR-097` present (OK); EDR-056 and research drafts diff empty (OK).
- Task 2 block: CORE_CONCEPTS CONTRACTS entry has two EDR-097 mentions plus Open and Resolved Questions rows (OK); Named Result directly follows Named Before Symbolic (OK); NAME_RESOLUTION invariant, GENERICS.md and EDR-097 present (OK); both coupling notes present (OK).
- Final block over all nine files: INDEX rows/count/date (OK); DECISION_LOG last entry is the EDR-097 entry and links `#gate-validation` (OK); no `implicit variable` and no operator-form `old` under `what/` (OK); 47 added relative links checked, none broken; no approximate-equality glyph or regex-match operator; `git diff --name-only 96faa5c` (excluding .planning and .gsd) equals exactly the nine named files; EDR-056 and `how/concepts/research` diffs empty; 4 trailers; `git status --short` shows only the five pre-existing untracked entries (`.gsd/`, two `.planning/phases` directories, `.planning/state.json`, plus the quick-task directory), no modified or staged tracked file. No file deletions.

## Deviations from Plan

### Auto-fixed issues

**1. [Rule 3 - Blocking/verification] zsh word-splitting in the plan's verify blocks**
- **Found during:** Task 2 verification
- **Issue:** The plan's verify blocks use unquoted variables such as `$F` and `$ALL` holding several paths. zsh does not word-split them, so the link-check diff and glyph grep ran on a single bogus path and passed vacuously (ugrep warned "No such file").
- **Fix:** Re-ran the link, glyph and file-list checks with shell arrays (`"${ALL[@]}"`) over all nine files, including Task 1's files. All passed (results above). No file content changed because of this.

**2. [Style] Edits applied with a Python replace script instead of the Edit tool**
- Each replacement asserted a unique match before writing, equivalent to Edit; the diffs were reviewed before each commit.

**3. [Plan wording] Part A step 5**
- The plan said to add "exactly one new sentence" to the Gate Validation lead-in. I also converted the two forward-looking sentences to past tense as the plan required, so the lead-in now says the author confirmed the selection and verdicts twice in slightly different words. Content is accurate; wording is slightly redundant.

Otherwise the plan executed as written. The `Where Contracts Apply` example uses a free function `square` and the trait `Shape` with `impl Shape for Square` (syntax copied from TRAITS.md); the method that uses `self` is the impl method, since TRAITS.md has no inherent-impl syntax to copy.

## Items not done / notes

- **R7 (EDR-097 Compliance item 8)**: the LLM-generation measurement was deliberately not run, as the plan states; it is recorded as deferred in the EDR and in the DECISION_LOG entry.
- **Commits on `main`**: the pre-commit protected-branch assertion would flag `main`, but this repository has `branching_strategy: none`, every prior commit is on `main`, and the orchestrator specified sequential execution on `main`. No `gsd_run` binary exists in the environment, so the SDK check could not be run; commits were made directly on `main` as instructed.
- No ROADMAP.md, STATE.md or PLAN.md changes; the docs commit is left to the orchestrator.
- Pre-existing dangling link in the GLOSSARY Contract entry (`#function-contract` has no entry) was left as it was.

## Known Stubs

None.

## Threat Flags

None. T-ir3-01 (sign-off spoofing) mitigated: the exact line appears twice, with the chat-origin note. T-ir3-02 and T-ir3-03 mitigated: explicit-path staging, file list equals the nine named files, Decision/Alternatives/Consequences unchanged.

## Self-Check: PASSED

- Commits 995c2c2, 65538e8, 6bc5104, 8f8464d exist in `git log 96faa5c..HEAD`.
- All nine modified files exist and are in the diff.
