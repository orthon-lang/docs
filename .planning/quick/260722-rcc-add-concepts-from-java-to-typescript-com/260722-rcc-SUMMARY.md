---
phase: quick-260722-rcc
plan: 01
subsystem: concept-research
tags: [type-system, union-types, literal-types, type-level-computation, typescript-comparison]
status: complete
dependency-graph:
  requires: []
  provides:
    - how/concepts/research/UNION_INTERSECTION_TYPES.md
    - how/concepts/research/LITERAL_TYPES.md
    - how/concepts/research/TYPE_LEVEL_COMPUTATION.md
  affects:
    - how/concepts/research/ALGEBRAIC_DATA_TYPES.md (future disambiguation reference)
    - how/concepts/research/ENUM_ALTERNATIVES.md (candidate for Concept Design Review revisit)
    - how/concepts/research/DYNAMIC_COLLECTIONS.md (undefined union syntax now has a home)
tech-stack:
  added: []
  patterns:
    - "Short-form DRAFT research template (Issue (Why), Examples, Implications for Orthon, Open Questions, Decision History, Cross-References) reused from DYNAMIC_TYPING.md / OPEN_CLASSES.md / SMART_CAST.md"
key-files:
  created:
    - how/concepts/research/UNION_INTERSECTION_TYPES.md
    - how/concepts/research/LITERAL_TYPES.md
    - how/concepts/research/TYPE_LEVEL_COMPUTATION.md
  modified: []
decisions:
  - "Bundled all eight TypeScript type-level sub-features (Conditional Types, Mapped Types, Template Literal Types, keyof, typeof, Indexed Access Types, infer, Utility Types) into a single TYPE_LEVEL_COMPUTATION.md file rather than seven separate files, per the plan's explicit instruction mirroring the prior SINGLETON_CLASS.md precedent."
  - "Named the Turing-completeness tension against Orthon's minimal-core and LLM-generability pillars explicitly in TYPE_LEVEL_COMPUTATION.md rather than softening it, per plan instruction."
metrics:
  duration: "~55 minutes"
  completed: "2026-07-22"
---

# Phase quick-260722-rcc Plan 01: Add Java-to-TypeScript Comparison Concepts Summary

Added three new DRAFT concept research files covering the TypeScript type-system
mechanisms genuinely missing from the research corpus: structural union/intersection
type combinators, value-as-type literal types, and the Turing-complete compile-time
type-computation cluster (conditional/mapped/template-literal types, keyof, typeof,
infer, and Utility Types bundled as one concept).

## What Was Built

Three additive, English-only DRAFT research documents in
`how/concepts/research/`, each following the short-form template established by
`DYNAMIC_TYPING.md`, `OPEN_CLASSES.md`, and `SMART_CAST.md` (DRAFT status banner,
then `## Issue (Why)`, `## Examples`, `## Implications for Orthon`,
`## Open Questions`, `## Decision History`, `### Cross-References`):

1. **`UNION_INTERSECTION_TYPES.md`** — TypeScript's `A | B` / `A & B` structural,
   untagged, set-theoretic type combinators over arbitrary types. Explicitly
   disambiguates from `ALGEBRAIC_DATA_TYPES.md`'s nominal tagged sum types
   (named-variant declaration + exhaustive `match` on a memory-layout tag) and
   names the gap left open by `DYNAMIC_COLLECTIONS.md`'s undefined
   `List[Int | String]` usage. Comparison table covers TypeScript, Scala 3,
   Python, Rust, Kotlin, and Java/C#.
2. **`LITERAL_TYPES.md`** — TypeScript's mechanism where a specific value
   (`"GET"`, `42`, `true`) inhabits its own singleton type, composing into
   closed unions as a lightweight enum alternative. Disambiguates from
   `ENUM_ALTERNATIVES.md` (named as a fourth option beyond that document's
   three named-constant/sum-type/enum strategies) and cross-references
   `UNION_INTERSECTION_TYPES.md` as the composition mechanism literal types
   feed into.
3. **`TYPE_LEVEL_COMPUTATION.md`** — Bundles Conditional Types, Mapped Types,
   Template Literal Types, `keyof`, `typeof` (type position), Indexed Access
   Types, `infer`, and the Utility Types built from them into one file, framed
   as a single compile-time functional mini-language over types. Disambiguates
   from `GENERICS.md`'s parametric polymorphism and `GRADUAL_TYPING.md`'s
   typed/untyped boundary question, and names the Turing-completeness tension
   against Orthon's minimal-core and LLM-generability pillars directly.

## Approach

For each file, I materialized/read the exact short-form template precedent
(`DYNAMIC_TYPING.md`, and `OPEN_CLASSES.md`/`SINGLETON_CLASS.md` via `git show`
from `main`, since this worktree's branch predates the commits that added
those two files to the tree — the branch point sits one commit behind `main`'s
`265af5d4` pre-dispatch commit, itself an ancestor relationship with no
divergent history) to confirm the exact banner wording, heading order, and
Cross-References footer style. I then read the six directly-relevant existing
research files (`ALGEBRAIC_DATA_TYPES.md`, `DYNAMIC_COLLECTIONS.md`,
`ENUM_ALTERNATIVES.md`, `GENERICS.md`, `GRADUAL_TYPING.md`, `SMART_CAST.md`,
`STRUCTURAL_TYPING.md`, `METAOBJECTS.md`) to ground each disambiguation claim
in the actual existing content rather than an assumed scope. Each file was
written in full per the plan's exact section-by-section content instructions,
then verified against that task's automated bash assertion script before
committing.

## Deviations from Plan

None — plan executed exactly as written. All three files match the specified
heading order, DRAFT banner date (2026-07-22), and cross-reference sets. All
three automated verify scripts passed on first attempt.

### Worktree Note (not a deviation, informational)

This worktree's branch (`worktree-agent-af3ab58a18b13d8a9`) was created before
`main` accumulated a few additional commits, including the commit that added
this plan's own `260722-rcc-PLAN.md` file. Per the dispatch instructions, the
plan file was materialized via `git show <commit>:<path>` before execution
began (an ancestor-relationship gap, not divergent/conflicting history — no
recovery or rewrite of the worktree was needed).

## Known Stubs

None. All three files are complete, standalone DRAFT research documents with
no placeholder content, no unwired data, and no "coming soon" markers.

## Self-Check: PASSED

- FOUND: how/concepts/research/UNION_INTERSECTION_TYPES.md
- FOUND: how/concepts/research/LITERAL_TYPES.md
- FOUND: how/concepts/research/TYPE_LEVEL_COMPUTATION.md
- FOUND: ecf6c28 (UNION_INTERSECTION_TYPES.md commit)
- FOUND: cfcb206 (LITERAL_TYPES.md commit)
- FOUND: 8152d41 (TYPE_LEVEL_COMPUTATION.md commit)
