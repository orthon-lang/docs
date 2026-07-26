---
title: Rewrite REQUIREMENTS.md Phase 2/3/4 sections to match actual concept inventory
date: 2026-07-26
priority: high
status: done
completed: 2026-07-26
completed_via: quick task 260726-s31 (commits b99f075, 5978d7a, 8c2ee5d)
---

**Note:** "Also touch while in there" listed ROADMAP.md too — that part was
initially deferred (see 260726-s31's SUMMARY.md), then completed directly
2026-07-26 (commit `b0e6f7b`), closing out this todo's full original scope.

## What

`.planning/REQUIREMENTS.md` was last scoped when `how/concepts/research/`
held ~22 files. It now holds 132 (40 essential + 38 important + 54
deferrable), already triaged by `SEED-001` and physically organized into
tier directories. Three requirement sections are stale as a result:

1. **`SEM-01..03` (Phase 2 — Semantic Model)** — doesn't name any source
   file. Should reference the ~10 `essential/` files identified as
   Semantic-Model raw material in
   [[2026-07-26-tier-vs-phase-mapping]] (DATA_MODEL, OWNERSHIP,
   OWNERSHIP_METAPROPERTY, OWNERSHIP_TRANSFER_OPERATOR, MUTABILITY,
   VALUE_SEMANTICS, IDENTITY_BASED_SAFETY, VISIBILITY_AND_ENCAPSULATION,
   SCOPED_RESOURCE_LIFECYCLE, EXPRESSION_ORIENTED_LANGUAGE).

2. **`PRIM-01..03` (Phase 3 — Primitive Blocks)** — same problem. Should
   reference the ~10 `essential/` files identified as Primitive-Block raw
   material (FOUNDATIONAL_ABSTRACTIONS, EXCLUSIVE_DECLARATIONS,
   STRUCT_AS_VALUE_TYPE, CLASS_WITH_ACT, ACT_AS_FUNCTION, FUNCTIONS,
   FINAL_BY_DEFAULT, NAMESPACES, DELEGATE, COMPOSITION_OVER_INHERITANCE).

3. **`CONCEPT-01..13` (Phase 4 — Derived Features)** — hardcodes 13 concept
   names from the old ~22-file inventory. Replace with requirement(s) that
   scale to the real ~114-file Phase 4 input (17 remaining essential files
   + 38 important + 54 deferrable + reject/ once populated), sequenced
   essential-first per `SEED-001`, rather than one requirement ID per named
   concept.

## Why

Without this, Phase 4 planning (`/gsd-plan-phase 4`) will either silently
under-scope against the stale 13-item list, or the planner will have to
re-derive this mapping from scratch under time pressure. The classification
work is already done — see [[2026-07-26-tier-vs-phase-mapping]].

## Also touch while in there

- `SEED-001` (`.planning/seeds/SEED-001-concept-research-tier-triage.md`):
  its essential-tier table currently implies all ~30 essential files are
  "Phase 4, first pass." Update it (or add a note) pointing at the
  Phase 2/3/4/Policy split so it doesn't contradict the rewritten
  requirements.
- `.planning/ROADMAP.md` Phase 2/3/4 sections: same staleness, should cite
  the same file lists.
- `how/concepts/research/README.md`: currently states all three tiers map
  to "Phase 4" passes — needs a caveat that `essential/` is not
  Phase-4-exclusive.

## Suggested entry point

`/gsd-quick` or `/gsd-plan-phase` once Phase 1.1 completes — this is a
planning-document edit, not code, so it doesn't need a full phase-research
cycle.
