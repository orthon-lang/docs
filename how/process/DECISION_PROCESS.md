# Decision Process

> One-page decision authority for the Orthon language design project.
>
> **Status:** DRAFT — created during Phase 1 (Foundation).
> **See also:** [`Concept Design Review`](../concept-design-review.md),
> [`DECISION_VALIDATION.md`](../gates/DECISION_VALIDATION.md),
> [`_language-design.md`](../gates/_language-design.md),
> [`DECISION_PIPELINE.md`](DECISION_PIPELINE.md)

---

## Purpose

Every consequential design decision in Orthon must follow a defined
process. This document establishes **who decides what, using which
criteria, and how the decision is recorded.**

## Decision Tiers

| Tier | Scope | Decision Maker | Record Keeping | Examples |
|------|-------|----------------|----------------|----------|
| **Tier 1 — Principle** | Design Principles, Vision, Semantic Model | Solo author + EDR | EDR (Architecture category) | "Data is immutable by default" |
| **Tier 2 — Language Concept** | A new language feature | Solo author + Pipeline + EDR | EDR (Architecture category) + concept doc | "Add pattern matching" |
| **Tier 3 — Process** | How design work is done | Solo author | EDR (Process category) | "Adopt 8-phase pipeline" |
| **Tier 4 — Inline** | Small decisions within a section | Solo author | Inline note in document | "Tuples are ordered by default" |

## Decision Pipeline

For any proposed feature (Tier 1–2), run through the
[**Decision Pipeline**](DECISION_PIPELINE.md) — 10 questions that
determine whether the feature should exist, at what level, and with
what priority.

## Recording

- **Tier 1–2 decisions** → EDR in `how/decision_records/{category}/`
- **Tier 3 decisions** → EDR in `how/decision_records/process/`
- **Tier 4 decisions** → inline `> **Decision:**` note in the relevant document
- All EDRs indexed in [`how/decision_records/INDEX.md`](../decision_records/INDEX.md)

## Guiding Rules

1. **No decision without rationale.** Every EDR documents context,
   the choice, consequences, and alternatives considered.
2. **No principle change without EDR.** `DESIGN_PRINCIPLES.md` is
   closed for modification; changes require a Tier 1 EDR.
3. **No concept without decomposition.** Every language feature must
   decompose to Primitive Blocks (from Phase 3).
4. **No feature without pipeline.** Every feature runs through the
   Decision Pipeline before detailed design begins.
