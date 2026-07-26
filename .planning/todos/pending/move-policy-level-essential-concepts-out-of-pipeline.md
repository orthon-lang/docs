---
title: Move Policy-level essential-tier concepts out of the Semantic Model / Primitive Blocks / Decision Pipeline track
date: 2026-07-26
priority: medium
status: pending
---

## What

Three files currently sitting in `how/concepts/research/essential/` describe
**Implementation Strategy / Policy** decisions (per `what/GLOSSARY.md`'s
Policy definition — "implementation-level decision"), not Core Language
semantics:

- `ALLOCATION.md` — stack vs. heap vs. arena is an Allocation Policy
- `REGION_BASED_MEMORY_MANAGEMENT.md` — **not purely Policy**: it embeds
  semantic claims (linear/affine ownership — "each value has exactly one
  owning name", no reference types, closure-capture-by-move, `mut`
  semantics) alongside genuine Allocation/Lifetime Policy mechanics (arena
  structure, bump-allocator performance, region inference). See
  [[2026-07-26-no-separate-memory-phase]].
- `EXECUTION_PROGRAM.md` — arguably core architecture (one of the five
  Vision pillars) rather than a Semantic Model dimension or Primitive Block;
  needs its own call rather than defaulting into Phase 2/3

None of these fit Phase 2 (Semantic Model), Phase 3 (Primitive Blocks), or
Phase 4 (feature-level Decision Pipeline) as currently scoped — see
[[2026-07-26-tier-vs-phase-mapping]].

## Why

`AGENTS.md`'s anti-pattern list explicitly calls out "Strategy-Specific
Language Semantics" as something to avoid — specs should describe semantics
in strategy-independent terms and let `IMPLEMENTATION_POLICIES.md` decide
*how*. Leaving these three files in the essential Core-Language pipeline
risks baking a specific Policy choice into what should be Strategy-level
guidance in `how/strategies/`.

## Suggested action

For `ALLOCATION.md` and `EXECUTION_PROGRAM.md`, decide (small design
review, not a big process): does it belong in
`how/strategies/DEFAULT_STRATEGY.md` (or a new Allocation Policy section)
instead of `how/concepts/research/essential/`? If `EXECUTION_PROGRAM.md` is
core architecture rather than a Policy, it may instead warrant staying in
the Phase 4 pipeline as its own top-level Language concept (it already has
Vision-pillar status) rather than being grouped with the other two.

For `REGION_BASED_MEMORY_MANAGEMENT.md`, **don't move it wholesale — split
it**:

- The semantic content (linear/affine ownership discipline, no reference
  types, `mut` semantics, closure-capture-by-move) feeds Phase 2's Semantic
  Model as additional Ownership/Mutation raw material, alongside
  `OWNERSHIP.md` and `MUTABILITY.md`.
- The mechanism content (arena structure, module/expression arenas,
  bump-allocator performance, region-inference implementation) moves to
  `how/strategies/` alongside `ALLOCATION.md`, feeding Phase 7.
