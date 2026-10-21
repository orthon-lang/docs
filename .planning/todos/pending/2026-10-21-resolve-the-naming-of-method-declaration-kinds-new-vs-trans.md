---
created: 2026-10-21T16:55:28.218Z
title: Resolve the naming of method declaration kinds: new vs trans and into
area: general
severity: minor
files:
  - what/SEMANTIC_MODEL.md (§ Mutation)
  - how/decision_records/architecture/EDR-013-semantic-model.md
  - how/decision_records/architecture/EDR-016-primitive-blocks.md
  - how/decision_records/architecture/EDR-047-async-await.md
  - how/concepts/research/essential/EXCLUSIVE_DECLARATIONS.md
  - how/concepts/research/essential/EFFECT_FOOTPRINT.md
  - what/concepts/CONSTRAINED_TYPES.md
  - what/concepts/DELEGATION.md
---

## Problem

The accepted documents and the research documents name the method declaration kinds differently.

- **Accepted:** three kinds, `fun` / `proc` / `new` — `SEMANTIC_MODEL.md` § Mutation (EDR-013), EDR-016 ("`fun` / `proc` / `new` are tags on the `function` primitive"), EDR-047 (async as a modifier on `proc` / `fun` / `new`).
- **Proposed (research):** four kinds, `fun` / `proc` / `trans` / `into` — `EXCLUSIVE_DECLARATIONS.md` and `EFFECT_FOOTPRINT.md`. `trans` replaces `new` because `new` reads as a constructor; `into` is the consuming kind (the Rust `FnOnce` analogue).

Related inconsistencies found on 2026-10-21:

- `CONSTRAINED_TYPES.md` says construction is not `new` / `make`, so `new` as a kind name collides with the constructor question.
- `DELEGATION.md` still uses the older `mut self` spelling in places.

The choice matters for the contract unification: the kind of a method member of a contract must match the kind of its implementation (D1 of the review), and the kind of the callable member of a function's anonymous type is derived from its captures (`fun` / `proc` / `into`). The hypothesis writes the kinds neutrally ("the transforming kind") until this is decided.

## Solution

1. Decide the canonical set: keep three kinds with `new`, adopt four kinds with `trans` and `into`, or another set. `into` needs its own decision (is a consuming kind part of v0.1?).
2. If the set or a name changes, record it as an EDR that amends EDR-016 and the Mutation section of `SEMANTIC_MODEL.md` (the accepted text is closed for casual change; this is at least a Tier 2 decision), and update EDR-047's wording.
3. Align the documents: grep for `new ` declarations in `what/` and `how/`, update the examples, and settle the `mut self` examples in `DELEGATION.md`.
4. Keep the contract documents neutral until the decision is recorded; then replace the neutral wording.
5. English only; docs-only change; run through `/gsd-quick`.
