---
id: SEED-002
status: dormant
planted: 2026-08-24
planted_during: Phase 04.1 — concepts-human-s-verification (milestone v0.1)
trigger_when: when relevant
scope: unknown
---

# SEED-002: Replace `mut` with `var` in the docs

## Why This Matters

_To be filled in. Run `$gsd-capture --seed --enrich SEED-002` to add context._

## When to Surface

**Trigger:** when relevant

This seed will surface during `$gsd-new-milestone` when the milestone scope matches.

## Scope Estimate

**Unknown** — run `$gsd-capture --seed --enrich SEED-002` to estimate effort.

## Breadcrumbs

- `what/THESES.md` (line ~267) — thesis records `mut` as the mutable marker
- `what/PRIMITIVE_BLOCKS.md` — reference primitive uses `&mut T` for the exclusive (mutable) mode
- `what/SEMANTIC_MODEL.md` — Mutation semantic dimension; `mut` modifier discussion
- `what/CORE_CONCEPTS.md` — `mut` qualifier in collection literal and related summaries
- `what/GLOSSARY.md` (line ~192) — glossary entry referencing the `mut` qualifier
- `what/concepts/{DECLARATION_BY_ASSIGNMENT, COLLECTION_LITERAL_SYNTAX, ITERATOR_PROTOCOL, SPAN, UNPACKING, DELEGATION}.md` — `mut` occurrences across accepted concepts
- Related decision context: Phase 3 D-10 (one `mut` keyword for binding-level and reference-level mutation marking); Phase 5 (Syntax Design) is where concrete keyword choice is finalized
- Note: `mut` appears in 12 files (~41 matches) under `what/` — the rename is a cross-cutting docs sweep

## Notes

_Captured via one-shot seed capture. Enrich with trigger, why, and scope at your convenience._
