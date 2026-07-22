---
phase: quick-260722-q8g
plan: 01
subsystem: docs
tags: [concept-research, dataclasses, mixin, structural-typing, traits, composition]

# Dependency graph
requires: []
provides:
  - "DRAFT concept research document synthesizing dataclass + stateless mixin + structural protocol as a recommended (not mandatory) composition recipe"
  - "Explicit flag of an unresolved tension between TRAITS.md (no fields, nominal impl) and MIXIN.md/STRUCTURAL_TYPING.md (fields allowed, structural satisfaction)"
affects: [Concept Design Review, TRAITS.md, MIXIN.md, STRUCTURAL_TYPING.md]

# Tech tracking
tech-stack:
  added: []
  patterns: ["_concept.md full template structure (Issue/Principles/Policy Footprint/Model/Default Strategy/Alternative Strategies/Open Questions/Decision History/Affected Documents) applied to a synthesis-style concept doc"]

key-files:
  created: [how/concepts/research/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md]
  modified: []

key-decisions:
  - "Followed STRUCTURAL_TYPING.md/COMPOSITION_OVER_INHERITANCE.md as the structural precedent (full _concept.md template) rather than DATACLASSES.md/MIXIN.md's shorter Issue/Principles/Examples/Implications form, per plan instruction"
  - "Explicitly flagged the TRAITS.md-vs-MIXIN.md/STRUCTURAL_TYPING.md conflict on trait fields and satisfaction mode as an open, unresolved tension rather than silently picking a side"

requirements-completed: []

coverage:
  - id: D1
    description: "CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md created with DRAFT banner, full _concept.md heading order, and required cross-references"
    verification:
      - kind: other
        ref: "plan's embedded verify script (heading order via grep -n line-number comparisons, cross-reference presence, Cyrillic regex check) — all passed"
        status: pass
    human_judgment: false

duration: 6min
completed: 2026-07-22
status: complete
---

# Quick Task 260722-q8g: Class-or-Structure Composition Concept Summary

**DRAFT concept research doc synthesizing Python's dataclass + stateless-mixin + structural-Protocol "gentleman's set" into an Orthon-translated composition recipe, with the TRAITS.md-vs-MIXIN.md field/satisfaction tension explicitly flagged as unresolved**

## Performance

- **Duration:** 6 min
- **Started:** 2026-07-22T16:04:00Z (context load) — file creation ~2026-07-22T16:03Z
- **Completed:** 2026-07-22T16:04:00Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Created `how/concepts/research/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`, a DRAFT concept document following the `_concept.md` template's full heading order exactly (Issue (Why) -> Principles -> Policy Footprint -> Model (What) -> Default Strategy -> Alternative Strategies -> Open Questions -> Decision History -> Affected Documents).
- Documented the "three tools, not always all three" framing (data-only aggregate / stateless behavior mixin / structural protocol), including a worked Python example (`Addable` Protocol + `AddMixin` + `Vector` dataclass) and a "Translating to Orthon" subsection mapping each tool onto `DATACLASSES.md`, `MIXIN.md`, `TRAITS.md`, and `STRUCTURAL_TYPING.md`.
- Explicitly disambiguated this document's narrower scope (which tool combination to use) from `COMPOSITION_OVER_INHERITANCE.md`'s broader "is-a vs has-a" philosophy, and flagged the genuine unresolved conflict between `TRAITS.md` (no fields, nominal `impl`) and `MIXIN.md`/`STRUCTURAL_TYPING.md` (fields allowed, structural satisfaction) as an Open Question requiring Concept Design Review resolution.

## Task Commits

Each task was committed atomically:

1. **Task 1: Write CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md** - `c523c3a` (docs)

_Note: single-task plan; no TDD gate applicable (documentation-only content)._

## Files Created/Modified
- `how/concepts/research/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md` - New DRAFT concept research document (227 lines) covering the recommended dataclass+mixin+protocol composition pattern, its Orthon translation, and open tensions with existing trait research.

## Decisions Made
- Used the full `_concept.md` template structure (matching `STRUCTURAL_TYPING.md`/`COMPOSITION_OVER_INHERITANCE.md`'s precedent) rather than the shorter Issue/Principles/Examples/Implications form used by `DATACLASSES.md`/`MIXIN.md`, per explicit plan instruction.
- Flagged (rather than silently resolved) the TRAITS.md-vs-MIXIN.md/STRUCTURAL_TYPING.md tension on trait fields and satisfaction mode, since resolving it is out of scope for a DRAFT research document and belongs to Concept Design Review.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- The new DRAFT document is ready for Concept Design Review (Milestone 2) alongside its four cross-referenced siblings (`DATACLASSES.md`, `MIXIN.md`, `COMPOSITION_OVER_INHERITANCE.md`, `STRUCTURAL_TYPING.md`).
- No other file in the repository was modified — cross-linking from sibling documents back to this new doc is deferred to Concept Design Review, as specified in the plan's verification step.
- The flagged TRAITS.md-vs-MIXIN.md/STRUCTURAL_TYPING.md tension on trait fields/satisfaction is a blocker for formal acceptance of this pattern and should be resolved during the broader concept review pass, not in a follow-up quick task.

---
*Phase: quick-260722-q8g*
*Completed: 2026-07-22*

## Self-Check: PASSED

- FOUND: how/concepts/research/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md
- FOUND: c523c3a (task commit)
