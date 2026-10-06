---
phase: quick-261006-chn
plan: 01
subsystem: language-spec
tags: [emit, iterator, closures, concept-review, EDR-052]

# Dependency graph
requires:
  - phase: 04-derived-features-decision-pipeline
    provides: EMIT_AS_INTERMEDIATE_RESULT.md (EDR-052 accepted concept doc)
provides:
  - "Nested `emit` Function — Splitting Stream from Terminal Result" subsection documenting the outer-fun/nested-emit-fun pattern as an alternative to the `.final()` dual-channel pattern
  - Disputed-pattern note on the existing `.final()` dual-channel model, pointing readers to the nested-function alternative
  - Open Questions item 4 recording the unresolved same-function-vs-split-functions choice
affects: [what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md, future Phase 5 syntax work touching emit]

# Actuals (#2632)
actuals:
  tokens: 1346
  tasks: 1
  commits: 1

# Tech tracking
tech-stack:
  added: []
  patterns: ["Outer-fun-owns-terminal-result / nested-fun-owns-emit-stream as an alternative to single-function dual-channel emit+return"]

key-files:
  created: []
  modified:
    - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md

key-decisions:
  - "No new EDR created — this documents an additional model alongside the ACCEPTED EDR-052 spec, not a replacement; the dual-channel `.final()` pattern remains accepted but is now flagged disputed/open pending resolution of the new Open Questions item."

patterns-established:
  - "Nested emit function pattern: when a per-call unit of work must update multiple (2+) closed-over mutable locals of an outer function AND can emit a variable (0..N) count of intermediate items, nest a nest a stream-owning `fun` inside the terminal-owning outer `fun` rather than combining `emit` with a differently-typed `return` in one function."

requirements-completed: [EDR-052]

coverage:
  - id: D1
    description: "EMIT_AS_INTERMEDIATE_RESULT.md documents the nested-emit-function model (outer fun owns terminal result, nested fun owns emit stream) as an alternative to .final()"
    requirement: "EDR-052"
    verification:
      - kind: other
        ref: "plan verify command: grep checks for 'Nested \`emit\` Function', 'track_batch', 'disputed', 'alternative', '.final()', 'Behavioural Specification', ACCEPTED EDR-052 badge, and exactly 4 numbered Open Questions items — all passed"
        status: pass
    human_judgment: false
  - id: D2
    description: "Existing .final() dual-channel example and Behavioural Specification remain unmodified in substance; accepted-status badge unchanged; no new EDR created"
    requirement: "EDR-052"
    verification:
      - kind: other
        ref: "Manual byte-range diff review of full file (git show e5924cc) confirmed only additive insertions; pre-existing sections (Intermediate Result Pattern, Semantic Model, Equivalent Forms, Behavioural Specification, Default Strategy, Alternative Strategies, Open Questions 1-3, Decision History) are byte-identical to pre-edit version"
        status: pass
    human_judgment: false

duration: 15min
completed: 2026-10-06
status: complete
---

# Quick Task 261006-chn: Document Resolved Nested-emit-Function Pattern Summary

**Added a "Nested `emit` Function" alternative model to EMIT_AS_INTERMEDIATE_RESULT.md, flagging the existing `.final()` dual-channel pattern as disputed per concept-review feedback, without creating a new EDR.**

## Performance

- **Duration:** 15 min
- **Started:** 2026-10-06T06:00:00Z (approx)
- **Completed:** 2026-10-06T06:13:39Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Added "### Nested `emit` Function — Splitting Stream from Terminal Result" subsection to `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`, positioned between "### Equivalent Forms" and "### Behavioural Specification"
- Included a disputed-pattern blockquote cross-referencing the `.final()` dual-channel pattern's readability problem (`let x = f()` looking like it binds `f`'s return value, not an `Iterator<T>`) and naming the split pattern as the concept-review's preferred alternative
- Added a verified Orthon code example (`Progress`/`show_progress`/`process_data`/`track_batch`) demonstrating an outer `fun` owning the terminal `Int` result via plain `return`, and a nested `fun` owning the `emit` stream by closing over three outer `mut` locals (`result`, `step`, `spikes`)
- Added three explanatory paragraphs covering (1) why nesting is required for multi-variable mutable closure capture, (2) the general rule for when `emit` vs. `return` is appropriate, (3) why this example avoids `.final()`
- Appended a 4th numbered item to "## Open Questions" capturing the unresolved choice between the same-function dual-channel pattern and the split outer/nested-function pattern

## Task Commits

Each task was committed atomically:

1. **Task 1: Add nested-emit-function model, disputed-pattern note, and Open Questions entry** - `e5924cc` (docs)

**Plan metadata:** (handled by orchestrator — SUMMARY.md/STATE.md/PLAN.md not committed by this executor per constraints)

## Files Created/Modified
- `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` - Added Nested `emit` Function subsection (disputed-pattern note, code example, three explanatory paragraphs) and Open Questions item 4; all pre-existing content (accepted-status badge, dual-channel example, Behavioural Specification, Decision History) left unchanged

## Decisions Made
- No new EDR created — intentional per plan objective; the dual-channel `.final()` pattern remains the ACCEPTED EDR-052 specification, now flagged as disputed/open alongside the new alternative model, pending resolution of Open Questions item 4.
- Used the `type Progress(step: Int, current: Int)` record-construction form (EDR-039 product-type syntax, matching DATACLASSES.md's `type Point(x: Float, y: Float)`) rather than a `type Progress:` block form, per the plan's explicit syntax precedent requirement.
- Confirmed `let mut` and trailing-colon `for`/`if` syntax against existing attested usage in `ITERATOR_PROTOCOL.md` and `ITERATION_LOOP.md` before writing the new code example, and confirmed the multi-variable mutable-closure-capture justification against `PRIMITIVE_BLOCKS.md` §5 (closures capture immutable by default; explicit `mut` on the captured binding required for mutable capture).

## Deviations from Plan

None - plan executed exactly as written. The plan's PLAN.md file was missing from this worktree at execution start (it had been written by the planner agent in a sibling worktree, `agent-abd325d6dafb6428b`, and not yet merged/visible in this one); per the task's explicit first-action instruction, it was read via the Read tool at that sibling worktree's filesystem path and written byte-for-byte to the same relative path in this worktree before execution began. This is a worktree-synchronization step, not a plan deviation — no plan content was altered.

## Issues Encountered
- The Bash tool's worktree-isolation sandbox rejected several multi-statement git commands (e.g., chained `&&` sequences mixing `git symbolic-ref`, `git rev-parse`, and conditional logic) as "too complex to verify." Resolved by splitting each check into a single, simple git invocation run separately — no functional impact, only tool-call shape.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` now carries both models (dual-channel `.final()` and split nested-function) side by side, with the dispute explicitly flagged — ready for a future concept-review or EDR to resolve Open Questions item 4 definitively.
- No blockers. This was a documentation-enrichment task on an already-ACCEPTED concept; no structural or architectural follow-up required.

---
*Phase: quick-261006-chn*
*Completed: 2026-10-06*

## Self-Check: PASSED

- FOUND: what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
- FOUND: e5924cc (git log --oneline --all)
