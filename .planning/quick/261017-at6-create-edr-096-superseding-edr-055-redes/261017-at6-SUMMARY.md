---
phase: quick-261017-at6
plan: 01
subsystem: language-specification
tags: [edr, unpacking, destructuring, pattern-syntax, supersession]
requires:
  - EDR-055
  - EDR-095 (precedent for supersession layout)
provides:
  - EDR-096 (Redesigned Unpacking Syntax)
  - UNPACKING.md rewritten to the redesigned syntax
affects:
  - what/GLOSSARY.md
  - what/CORE_CONCEPTS.md
  - how/DESIGN_PRINCIPLES.md (POLA)
  - EDR-053, EDR-055, EDR-065
key-files:
  created:
    - how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md
  modified:
    - how/decision_records/architecture/EDR-055-unpacking.md
    - how/decision_records/architecture/EDR-065-named-and-optional-parameters.md
    - how/decision_records/architecture/EDR-053-iteration-loop.md
    - how/decision_records/INDEX.md
    - how/gates/DECISION_LOG.md
    - what/concepts/UNPACKING.md
    - what/concepts/PATTERN_MATCHING.md
    - what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md
    - what/GLOSSARY.md
    - what/CORE_CONCEPTS.md
    - how/DESIGN_PRINCIPLES.md
    - what/PRIMITIVE_BLOCKS.md
    - what/concepts/GRADUAL_TYPING.md
    - what/concepts/CONTRACTS.md
    - what/concepts/ITERATION_LOOP.md
    - what/syntax/ITERATION_SYNTAX.md
    - what/concepts/LAZY_SEQUENCE_GENERATORS.md
    - what/concepts/INDEXING.md
decisions:
  - "EDR-096 records the 13 locked decisions D-01..D-13 as Accepted; EDR-055 is Superseded (syntax decisions only)"
  - "Open items DEF-a..DEF-f and drafting questions (g), (h) recorded as open, not decided"
metrics:
  duration: single session, 2026-10-17
  completed: 2026-10-17
status: complete
actuals:
  tokens: 15000
  tasks: 3
  commits: 3
plan_head_before: 3136e31fb275bed70cc48f787e23a5f0a54c86a6
plan_head_after: 13463a913892d145e649eeab5ed601535307560b
---

# Phase quick-261017-at6 Plan 01: EDR-096 Superseding EDR-055 Summary

EDR-096 records the redesigned Orthon unpacking syntax (prefix-star rest `*x`, explicit `value as label` labels, `tuple(a)` 1-tuple, keyword-free destructuring with whole-pattern `var`, irrefutable-only patterns outside `match`), supersedes the syntax decisions of EDR-055, and is propagated through 18 documents.

## Sign-off

The Human Sign-off line in EDR-096 reads `Reviewed-by: mniedre · Date: 2026-10-17 · Verdict: LOCKED`. The sign-off was given by the author in the session, as relayed by the orchestrator. It was not self-certified by the agent (AGENTS.md section 7.4).

## Tasks and commits

| Task | Name | Commit | Key files |
|------|------|--------|-----------|
| 1 | Decision spine: EDR-096, supersede EDR-055, INDEX, DECISION_LOG, EDR-065 and EDR-053 amendments | 9b63ca6 | EDR-096, EDR-055, INDEX.md, DECISION_LOG.md, EDR-065, EDR-053 |
| 2 | Concept layer: UNPACKING.md rewrite, PATTERN_MATCHING, NAMED_AND_OPTIONAL_PARAMETERS, GLOSSARY, CORE_CONCEPTS | c530b03 | what/concepts/UNPACKING.md and four others |
| 3 | POLA amendment, PRIMITIVE_BLOCKS note, stale example cleanup, sweep | 13463a9 | DESIGN_PRINCIPLES.md and eight others |

Commit count measured from the ledger: `git rev-list --count 3136e31..HEAD` = 3 (no uncommitted changes remain; no files were deleted).

## Verification results (observed)

| Check | Result |
|-------|--------|
| Task 1 automated checks (header, D-01..D-13, 7 gate rows, Deferred / Open, EDR-055 Superseded with body intact, INDEX >= 5 mentions and `Superseded 1`, DECISION_LOG entry, EDR-065 and EDR-053 amendments, Cyrillic, links) | `TASK1_OK`, broken links = 0 |
| Task 2 automated checks (UNPACKING section order, required strings, Label/Rest Pattern/Tuple entries, withdrawn-spelling greps, Cyrillic, links) | `TASK2_OK`; broken links = 2 (both pre-existing baseline: PATTERN_MATCHING to EDR-025, GLOSSARY to EDR-014); UNPACKING.md = 0 |
| Task 3 automated checks (Semantic Purity checksum, POLA, PRIMITIVE_BLOCKS, no `:=`, generator/loop examples, repo sweep, follow-up list, Cyrillic, links) | `TASK3_OK`; Semantic Purity checksum still `1aba5d72de2425cc4bcf007b7f27c8e9c946ba82`; broken links = 19 over the changed-file set, equal to the planning-time baseline, none new |
| Plan-level withdrawn-spelling grep over the six named docs | empty (grep rc=1) |
| GLOSSARY entries present | Label, Rest Pattern, Tuple, Unpacking (Destructuring): 4 lines |
| Repo-wide sweep (what/, how/, excluding research and historical records) for dotted rest, let/var-prefixed tuple or brace destructuring, brace record loops | empty |
| Cyrillic check over all changed files | exits 0 |
| `git status` after the last commit | clean for tracked files; this SUMMARY is intentionally uncommitted |

The plan's `<verification>` scope item about `git status` listing only `files_modified`: one extra file (`what/concepts/INDEXING.md`) was changed, see Deviations.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Consistency] Parenthesized tuple `case` patterns in PATTERN_MATCHING.md**
- **Found during:** Task 2
- **Issue:** The existing tuple destructuring example used `case (0, 0)`, `case (x, y)`, which contradicts D-05 (patterns in `case` are comma lists without parentheses) now documented in the same file.
- **Fix:** Rewrote those four example lines to `case 0, 0`, `case x, 0`, `case 0, y`, `case x, y`. No other line of the existing text was changed; the ownership wording is untouched (only a pointer note was added).
- **Files modified:** what/concepts/PATTERN_MATCHING.md
- **Commit:** c530b03

**2. [Plan step 7 - unanticipated example] INDEXING.md loop example**
- **Found during:** Task 3 repo-wide sweep
- **Issue:** `what/concepts/INDEXING.md` line 44 used the parenthesized loop pattern `for (i, v) in items.enumerate():`.
- **Fix:** Changed to `for i, v in items.enumerate():` (one line). The file is not in the plan's `files_modified` list; plan step 7 explicitly directs fixing destructuring examples the planning list missed and mentioning them in the SUMMARY.
- **Files modified:** what/concepts/INDEXING.md
- **Commit:** 13463a9

**3. [Plan step 7 - follow-up list] Extended EDR-096 follow-up list**
- **Found during:** Task 3 live sweep for call-site `name: value`
- **Issue:** The live sweep found call-site named arguments in CONCURRENCY (`what/concepts/`), EDR-047, EDR-049 and EDR-054 that the planning-time list omitted.
- **Fix:** Added them to the Negative Consequences follow-up item in EDR-096 (one Edit). These files were not changed.
- **Commit:** 13463a9

**4. INDEX.md By Category row title**
- The By Category EDR-096 row uses the short title "Redesigned Unpacking Syntax" (the neighbouring EDR-095 row there also uses a short title); the All Records row uses the full title.

No auth gates occurred. No package installs. No other deviations.

## Known Stubs

None.

## Threat Flags

None (documentation only). Mitigations from the plan threat register were applied: sign-off line comes from the author's explicit answer; EDR-055 body and DECISION_LOG history were not edited (only header, a note and appended text).

## Decision Summary Check (AGENTS.md section 11.2, step 7)

A plain-language recap of what EDR-096 now records. Please correct anything that does not match your intent, including any assumption these summaries share.

- **D-01.** One ignored position is still written `_`, because `?` is already used by null-safety and Result propagation.
- **D-02.** The remainder of a pattern is `*x`; `*_` throws it away. The dotted rest and brace rest forms are withdrawn, so `..` is only Range.
- **D-03.** The prefix star looks the same whether it collects (`first, *tail = nums`) or spreads (`nums2 = (first, *tail)`). POLA in DESIGN_PRINCIPLES now allows direction by position when the two forms are mirror images; Semantic Purity is untouched.
- **D-04.** Spreading works on tuples and finite collections (List, Array). Lazy iterators are never spread silently; you collect first (`*gen.to_list()`). Set is open.
- **D-05.** Patterns in `for`, `case` and declarations are comma lists without parentheses; parentheses only for nested patterns and the empty tuple. Tuple literals on the expression side keep parentheses.
- **D-06.** Labels are explicit: `value as label`. Position stays the implicit 1-based label. No labels from bare identifiers. Labels are outside types and equality, override position when binding arguments, and duplicates or `3 as x as y` are errors. This replaces call-site `name: value` (EDR-065); declarations keep `name: Type`.
- **D-07.** Exactly one star per pattern, at any position, positional patterns only.
- **D-08.** The 1-tuple exists, spelled only `tuple(a)`; `(a,)` is forbidden; `(a)` is grouping; `tuple()` is `()`; `(a, b)` is sugar for `tuple(a, b)`; the pattern is `tuple(x)`.
- **D-09.** Brace record patterns are gone. An anonymous record is a tuple whose elements all have labels; nominal types are destructured with their constructor pattern `Person(n as name, a as age)`.
- **D-10.** Destructuring declares without a keyword (`x, y = point`); `var` makes all names in the pattern mutable; the `let` form is withdrawn; a pattern mixing new and existing names is a compile error.
- **D-11.** No per-element mutability; use a separate `var z = x`.
- **D-12.** No defaults inside patterns; the default belongs to the whole parameter, absence is Option with `??`.
- **D-13.** Outside `match` only irrefutable patterns are allowed; a tuple pattern on `Option<T>` is a type error; no implicit lifting.
- **Open, not decided:** DEF-a (whether variadic functions, call-site spreading and a labeled remainder are in v0.1), DEF-b (move versus borrow on destructuring, decided jointly; tracked discrepancy with copy-on-write), DEF-c (let-else / single-arm irrefutable match), DEF-d (Set spreading), DEF-e (Map / brace literal syntax), DEF-f (prefix star versus multiplication).

Do any of these summaries feel wrong, or does an assumption behind them (for example that the star form in call positions is decided even though variadic functions are not) not match what you intended?

## Items for the author to review (inferred by the executor, not taken verbatim from the SPEC)

1. **Sign-off line transcription.** Written exactly as the orchestrator relayed it; no extra text added.
2. **D-10 complement reading.** The EDR and UNPACKING.md state that a pattern whose names are all already declared is a reassignment (each name must be mutable) and only the mixed case is an error. This follows EDR-074 but the SPEC only states the mixed-case error.
3. **Drafting questions (g) and (h).** (g) whether redundant outer parentheses around a top-level pattern are rejected or tolerated; (h) representation of the collected remainder when the scrutinee is a List or Array. Recorded in the EDR and in UNPACKING.md Open Questions as questions, not decisions.
4. **Gate verdict proposals.** The seven Gate Validation verdicts and notes in EDR-096 and the DECISION_LOG entry are the planner's proposals (three Pass with Flag verdicts: star versus multiplication, ownership discrepancy, novelty of `value as label`).
5. **Record loop rewrite.** The ITERATION_LOOP.md brace record loop became `for Person(n as name, a as age) in people:` with the print line using `n` and `a`.
6. **Generator examples.** The two LAZY_SEQUENCE_GENERATORS.md examples now use `var a, b = (0, 1)` and the reassignment `a, b = (b, a + b)`.
7. **Parameter tuple patterns keep parentheses.** UNPACKING.md says that in a parameter list the comma already separates parameters, so a tuple pattern there is written `fun first_of((x, _): Pair<Int, Int>)`. This is the planned example; the explanatory sentence is the executor's reasoning.
8. **Example code in UNPACKING.md.** Illustrative snippets not taken from the SPEC (for instance `*init, last = nums`, the narrowing example with `!= None` taken from TYPE_LEVEL_NULL_SAFETY.md, and the description of rest desugaring in words) are for you to confirm.
9. **Pre-existing issues not touched.** INDEX.md has a stray line `6` before the Status Summary heading and GLOSSARY.md has a garbled Representation Symmetry entry (its code fence runs into the R section); both pre-date this task and were left alone. The 19 broken links in the changed-file set are the known baseline.

## Self-Check: PASSED

- Created file present: how/decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md (FOUND).
- Commits present on the branch: 9b63ca6, c530b03, 13463a9 (FOUND in `git log`).
- Verification scripts printed TASK1_OK, TASK2_OK, TASK3_OK.
