# Layer-Specific Validation Procedures

> **Status:** Design question — not yet resolved.
> **Opened:** 2026-07-21

## Problem

The [Core Inclusion Filter](../how/architecture/CORE_INCLUSION_FILTER.md)
assigns a language feature to a semantic layer (Level 0–3), but all
layers proceed through the same validation gates defined in
[`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md).
This is inconsistent: Level 0 (Data Model) and Level 3 (Standard
Library) have fundamentally different bars, yet share the same
procedure.

## The Gap

After the filter answers *"which layer?"*, there is no answer to
*"what validation procedure applies to this layer?"*

## Proposed Direction

Each layer should have its own validation procedure:

```
Level 0 (Data Model)    → Architecture EDR + all 6 gates + Core Change Criterion
Level 1 (Primitive Op)  → Architecture EDR + relevant gate subset
Level 2 (Pattern)       → Composition formula + reduced gate subset
Level 3 (Std Library)   → Library governance procedure (no EDR required)
Level 3+ (Framework)    → No language validation needed
```

## Open Questions

- Where should layer-specific procedures live?
  (a) Separate template files: `_gate-level-0.md` .. `_gate-level-3.md`
  (b) Expanded sections within `CORE_INCLUSION_FILTER.md` itself
  (c) Expanded sections within `_language-design.md`
- Should the Gate Selection table in `DECISION_VALIDATION.md` map to
  layer-specific procedures instead of (or in addition to) decision types?

## Cross-References

- [`CORE_INCLUSION_FILTER.md`](../how/architecture/CORE_INCLUSION_FILTER.md) — § Relationship to the Validation Gates
- [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md) — § Gate Selection
- [`_language-design.md`](../how/gates/_language-design.md) — operational checklist
- [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md) — § Semantic Dependency Architecture
