---
phase: quick-261006-d5x
plan: 01
subsystem: decision-records
tags: [edr, emit, generators, concept-review, supersession]
status: complete
dependency-graph:
  requires: [EDR-052, EDR-021, EDR-050, EDR-091, EDR-092, EDR-093, EDR-085]
  provides: [EDR-095]
  affects:
    - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
    - how/decision_records/INDEX.md
    - what/GLOSSARY.md
tech-stack:
  added: []
  patterns:
    - "Split outer-fun/nested-emit-fun pattern for combining a variable-count stream with a one-shot terminal result"
    - "Supersession banner convention (ASYNC_AWAIT.md style) for concept docs reversed by a later EDR"
key-files:
  created:
    - how/decision_records/architecture/EDR-095-reject-emit-return-dual-channel.md
  modified:
    - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
    - how/decision_records/INDEX.md
    - what/GLOSSARY.md
decisions:
  - "EDR-095 (Architecture, Accepted, 2026-10-06): rejects the emit+return dual-channel pattern and .final() accessor from EDR-052; accepts the split outer-fun/nested-emit-fun model as ACCEPTED; adds an emit-applicability criterion and a compiler-diagnostic recommendation; retracts EDR-052 Principle 2 (push-model framing) as a consequence — all emit remains pull-based."
metrics:
  duration: "~35 min"
  completed: 2026-10-06
actuals:
  tokens: 9555
  tasks: 3
  commits: 3
  plan_head_before: e859f27858a2990c47265b4908cc051a702dbc65
  plan_head_after: ba2dec44d49fb4dc01d4eee4a74086cb1ece8459
---

# Phase quick-261006-d5x Plan 01: Create a New Superseding EDR Summary

EDR-095 formally rejects EDR-052's `emit`+`return` dual-channel pattern and its `.final()` accessor, promotes the already-drafted split outer-`fun`/nested-emit-`fun` model to ACCEPTED, and adds an emit-applicability criterion plus a compiler-diagnostic recommendation — then propagates that decision into `EMIT_AS_INTERMEDIATE_RESULT.md`, `INDEX.md`, and `GLOSSARY.md`.

## What Was Built

**Task 1 — Authored EDR-095** (`how/decision_records/architecture/EDR-095-reject-emit-return-dual-channel.md`):
- Re-verified EDR-095 was the next free number (EDR-094 was the current highest; no existing EDR-095 file). Used EDR-095 as planned — no renumbering needed.
- Header: Status Accepted, Date 2026-10-06, Category Architecture, Scope Subsystem, Human Sign-off (`Reviewed-by: mniedre · Date: 2026-10-06 · Verdict: LOCKED`), and `**Supersedes:** [EDR-052](./EDR-052-emit-as-intermediate-result.md)`.
- Context section states the readability defect (`let x = f()` expected to bind `f`'s `return`, not an `Iterator<T>`; `.final()` hides a second channel), names the 2026-10-06 concept-review origin and the quick-261006-chn commit (`e5924cc`) that drafted the alternative, and cites EDR-021/EDR-050/EDR-091, EDR-092/EDR-093, and EDR-085 as settled facts the decision must stay consistent with.
- Decision section: all 8 numbered theses verbatim per the plan — (1) reject dual-channel, (2) reject `.final()`, (3) accept the split model with the `track_batch`/`process_data`/`Progress` example copied verbatim (code fence and all three explanatory paragraphs) from `EMIT_AS_INTERMEDIATE_RESULT.md`, (4) no new function-declaration keyword (citing `gen(...)`'s reservation, `GENERATORS.md` Principle 1, and the closed `fun`/`proc`/`new` triad), (5) no new `execution_context` constructor (citing EDR-085's closed `defer`/`delegate`/`spawn`/`fork` set), (6) the emit-applicability criterion, (7) the compiler-diagnostic recommendation using `COMPILER_AS_STATIC_ANALYZER.md`'s Layer 6 and `code`/`severity`/`location`/`repair_hint` vocabulary, (8) the pull-based retraction of EDR-052 Principle 2, citing `LAZY_SEQUENCE_GENERATORS.md`'s desugaring description.
- Honest Consequences (Positive/Negative), verifiable Compliance (7 rules), a full 5-row Alternatives Considered table, and a 5-row Gate Validation table (`USER_VALUE_GATE`, `LOGICAL_CONSISTENCY_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE`, `LLM_GENERABILITY_GATE` — the last flagged "Pass (Flag)" since the diagnostic is a recommendation, not a specified compiler pass) plus a "Gates not applied" line mirroring EDR-052's own framing.

**Task 2 — Propagated into `EMIT_AS_INTERMEDIATE_RESULT.md`:**
- Added a `⚠️ SUPERSEDED — EDR-095` banner above the retained `✅ ACCEPTED — EDR-052` line (ASYNC_AWAIT.md convention); appended "(superseded 2026-10-06 by EDR-095)" to the EDR-052 Status line; added `COMPILER_AS_STATIC_ANALYZER.md` to "See also". EDR-052's own file and INDEX.md status were left untouched.
- Principle 2 struck through with a `[WITHDRAWN — EDR-095 ...]` note (not deleted).
- Policy Footprint: Evaluation Policy and Iterator Protocol Policy rows marked WITHDRAWN with one-clause reasons; added a new Diagnostic Policy row.
- Model (What): inserted an explanatory blockquote before the three dual-channel subsections; appended "— WITHDRAWN (EDR-095)" to the Intermediate Result Pattern, Semantic Model, and Equivalent Forms headings (bodies left unchanged as historical record); replaced the "Disputed — open" blockquote under "### Nested `emit` Function" with an "ACCEPTED model per EDR-095" blockquote; left the code example and its three paragraphs completely unchanged.
- Behavioural Specification heading marked "— PARTIALLY WITHDRAWN (EDR-095)"; items 2 and 4 struck through with WITHDRAWN notes; items 1 and 3 left intact.
- Default Strategy rewritten: states `emit` compiles exactly per `LAZY_SEQUENCE_GENERATORS.md` (no separate path, `.final()` removed), then documents the emit-applicability criterion and the compiler-diagnostic recommendation using `COMPILER_AS_STATIC_ANALYZER.md`'s vocabulary.
- Alternative Strategies: removed the now-default "No `.final()` support" row; marked "Push-based intermediate results" WITHDRAWN (kept as anti-memory); left "Eager with intermediate buffer" unchanged.
- Open Questions: removed item 4 (now resolved), kept items 1-3 unrenumbered (exactly 3 items).
- Decision History: appended the 2026-10-06 EDR-095 entry above the existing 2026-07-27 entry.
- Affected Documents checklist: checked `GLOSSARY.md` (with a note it's updated by this plan's Task 3) and added an unchecked `INDEX.md` line with the same note.

**Task 3 — Registered EDR-095 and rewrote the glossary entry:**
- `INDEX.md` "All Records" table: added the EDR-095 row immediately after EDR-094, Supersedes EDR-052.
- `INDEX.md` "Architecture" (By Category) table: added the matching row, same column layout as the neighbouring EDR-092/093/094 rows (trailing Supersedes/Amends column).
- Status Summary Accepted count incremented 79 → 80 (read the current printed value first, per the plan's contingency instruction).
- "Last updated" line set to 2026-10-06 with a new note prepended to the existing "Prior: ..." chain — no prior history lost.
- `GLOSSARY.md` § Intermediate Result rewritten around the split outer-fun/nested-emit-fun pattern (no longer describing `.final()`/dual-channel as the model), using a short illustrative excerpt of the `track_batch`/`process_data` shape; Source line updated to cite both EDR-052 and EDR-095; See-also links (`Generator`, `Lazy Sequence`, `Iterator Protocol`) checked against the glossary and confirmed current (none point at a `.final()`-specific concept).

## Deviations from Plan

None — plan executed exactly as written. The EDR-095 number was confirmed free before writing (EDR-094 was the highest existing EDR; no `EDR-095-*.md` file existed under `how/decision_records/architecture/`), so no renumbering was required anywhere in the plan's tasks.

## Verification

All three tasks' `<automated>` checks printed their expected sentinels: `TASK1_OK`, `TASK2_OK`, `TASK3_OK`.

`git status --short` after all three commits shows only the new EDR-095 file, `EMIT_AS_INTERMEDIATE_RESULT.md`, `INDEX.md`, `GLOSSARY.md` as tracked changes (plus this plan's own `.planning/quick/.../` directory, untracked and out of scope for this executor per the orchestrator's docs-commit convention). EDR-021, EDR-050, EDR-091, EDR-092, EDR-093, `GENERATORS.md`, `LAZY_SEQUENCE_GENERATORS.md`, and `ITERATOR_PROTOCOL.md` were not touched.

All relative markdown links introduced (EDR-052 path, `COMPILER_AS_STATIC_ANALYZER.md`, the four `how/gates/methods/*.md` gate-method links) were spot-checked with `test -f` and resolve. All added content is in English per project convention.

## Self-Check: PASSED

- FOUND: `how/decision_records/architecture/EDR-095-reject-emit-return-dual-channel.md`
- FOUND: commit `58d742e` (Task 1 — author EDR-095)
- FOUND: commit `e856d94` (Task 2 — propagate into EMIT_AS_INTERMEDIATE_RESULT.md)
- FOUND: commit `ba2dec4` (Task 3 — register in INDEX.md and rewrite GLOSSARY.md)
