---
id: SEED-003
status: dormant
planted: 2026-08-24
planted_during: Phase 04.1 — concepts-human-s-verification (milestone v0.1)
trigger_when: when relevant
scope: unknown
---

# SEED-003: Abolish `let` as the general immutable form in the 33 accepted concepts

## Why This Matters

_To be filled in. Run `$gsd-capture --seed --enrich SEED-003` to add context._

## When to Surface

**Trigger:** when relevant

This seed will surface during `$gsd-new-milestone` when the milestone scope matches.

## Scope Estimate

**Unknown** — run `$gsd-capture --seed --enrich SEED-003` to estimate effort.

## Breadcrumbs

- `what/concepts/DECLARATION_BY_ASSIGNMENT.md` — `let` currently reserved for shadowing only; initial declaration is by first assignment; variables immutable by default (`mut` required for reassignment). The `let`-for-shadowing keyword choice is deferred to Phase 5 (see `how/syntax/SHADOWING_SYNTAX.md`).
- `what/concepts/LITERAL_TYPES.md` — `let x = "GET"` preserves the singleton type while `var y = "GET"` widens to `String`; removing `let` as the immutable form would collapse this distinction.
- `what/CORE_CONCEPTS.md` — accepted-concept registry summaries reference `let`/`var` across entries (e.g., DECLARATION_BY_ASSIGNMENT, LITERAL_TYPES, UNPACKING).
- `what/PRIMITIVE_BLOCKS.md`, `what/THESES.md`, `what/GLOSSARY.md` — spec-level references to the `let` keyword.
- `let` appears in ~30 of the 33 accepted concept docs under `what/concepts/` — a cross-cutting sweep, similar in scope to the SEED-002 `mut` → `var` rename.
- Related seed: `SEED-002` (replace `mut` with `var` in the docs) — same binding-keyword family; both touch the Phase 3 binding/mutation model (D-03, D-10).
- Syntax decisions deferred to Phase 5: `how/syntax/SHADOWING_SYNTAX.md`, `how/syntax/TYPE_ANNOTATION_SYNTAX.md`.

## Notes

_Captured via one-shot seed capture. Enrich with trigger, why, and scope at your convenience._

## Measured (2026-09-27)

- Recount refines the scope: `let x =` bindings appear **113 times across 30
  files** under `what/concepts/` (the seed's "~30 of the 33 accepted concept
  docs" is confirmed).
- Documentation-only sweep, but it interacts with `LITERAL_TYPES` widening
  semantics — `let` currently carries the singleton-type case.
- Blocked by the same Phase 5 syntax decisions as SEED-002
  (`how/syntax/SHADOWING_SYNTAX.md`, `how/syntax/TYPE_ANNOTATION_SYNTAX.md`).
