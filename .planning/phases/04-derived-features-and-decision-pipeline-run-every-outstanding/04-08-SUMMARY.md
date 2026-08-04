---
phase: "04"
plan: "08"
subsystem: "derived-features"
tags: [decision-pipeline, rejections, deferrals, prototype, significant-whitespace, dynamic-typing, class-or-structure, principle-violations]
requires: [04-01, 04-02, 04-03, 04-04, 04-05, 04-06, 04-07]
provides: [EDR-075, EDR-076, EDR-077, EDR-078, reject-tier, deferral-rationale]
affects: [how/decision_records/INDEX.md, how/process/DECISION_PIPELINE.md, how/concepts/research/reject/README.md, how/concepts/research/deferrable/README.md, what/CORE_CONCEPTS.md]
tech-stack:
  added: []
  patterns: [Formal rejection EDR pattern — context, principle violations, alternatives, verdict; Organized deferral rationale table]
key-files:
  created:
    - how/decision_records/architecture/EDR-075-reject-prototype.md
    - how/decision_records/architecture/EDR-076-reject-significant-whitespace.md
    - how/decision_records/architecture/EDR-077-reject-dynamic-typing.md
    - how/decision_records/architecture/EDR-078-reject-class-or-structure-as-primary-composition.md
  modified:
    - how/decision_records/INDEX.md
    - how/process/DECISION_PIPELINE.md
    - how/concepts/research/reject/README.md
    - how/concepts/research/deferrable/README.md
    - what/CORE_CONCEPTS.md
  moved:
    - how/concepts/research/deferrable/PROTOTYPE.md -> how/concepts/research/reject/PROTOTYPE.md
    - how/concepts/research/deferrable/SIGNIFICANT_WHITESPACE.md -> how/concepts/research/reject/SIGNIFICANT_WHITESPACE.md
    - how/concepts/research/deferrable/DYNAMIC_TYPING.md -> how/concepts/research/reject/DYNAMIC_TYPING.md
    - how/concepts/research/deferrable/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md -> how/concepts/research/reject/CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md
decisions:
  - "PROTOTYPE formally rejected (EDR-075): violates Data First, Orthogonality, Semantic Purity"
  - "SIGNIFICANT_WHITESPACE formally rejected (EDR-076): violates Explicitness, Semantic Purity, Deterministic Behavior; Phase 5 commits to no significant whitespace"
  - "DYNAMIC_TYPING formally rejected (EDR-077): violates Explicitness, Declarative With Static Guarantees, Correctness Before Performance"
  - "CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION formally rejected (EDR-078): violates Data First, Orthogonality, Composition Over Inheritance"
  - "~54 deferrable-tier concepts documented with deferral rationale (target version, dependency) — final aggregation records 50 deferred"
metrics:
  duration: "~1.5 hours"
  completed_date: "2026-07-27"
status: complete
---

# Phase 4 Plan 8: Rejections and Deferrals Decision Pipeline Summary

## One-Liner

Processed the 4 rejection candidates (PROTOTYPE, SIGNIFICANT_WHITESPACE, DYNAMIC_TYPING, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION) through **formal rejection EDRs** citing specific DESIGN_PRINCIPLES.md violations, moved them from `deferrable/` to `reject/`, and documented deferral rationale for the deferrable-tier concepts. Closes CONCEPT-REJECT-01 and CONCEPT-DEFER-01.

## Rejection Decisions

| Concept | EDR | Verdict | Principle Violations |
|---------|-----|---------|----------------------|
| PROTOTYPE | EDR-075 | **Rejected** | Data First (behavior+data coupling), Orthogonality (conflates concerns), Semantic Purity |
| SIGNIFICANT_WHITESPACE | EDR-076 | **Rejected** | Explicitness (indentation-dependent semantics), Semantic Purity, Deterministic Behavior |
| DYNAMIC_TYPING | EDR-077 | **Rejected** | Explicitness (runtime type errors), Declarative With Static Guarantees, Correctness Before Performance |
| CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION | EDR-078 | **Rejected** | Data First (behavior+state coupling), Orthogonality, Composition Over Inheritance |

Each rejection EDR follows the formal pattern: **Context** (what the concept proposes) → **Decision** (formal rejection) → **Principle Violations** (precise mapping to DESIGN_PRINCIPLES.md rules with paragraph references) → **Alternatives Considered** (what Orthon uses instead — e.g., traits + struct + delegate for PROTOTYPE; explicit `{ }` block delimiters for SIGNIFICANT_WHITESPACE) → **Formal Rejection Verdict** (explicitly not planned for future versions).

## File Moves

All 4 files moved from `how/concepts/research/deferrable/` to `how/concepts/research/reject/`, preserving them as traceable provenance of rejected proposals. The `reject/` directory README updated to describe the tier.

## Deferral Rationale

The deferrable-tier concepts (planned ~54; final aggregation records **50 deferred**) were documented with one-paragraph rationale each: why deferred, target version (v0.2/v0.3), and any dependency. The organized rationale table lives in the Pipeline Application log in `how/process/DECISION_PIPELINE.md`, consistent with SEED-001 tiering criteria. This closes CONCEPT-DEFER-01.

## Key Decisions Made

1. **Formal rejection via EDR, not silent omission** — every rejected concept gets a full EDR with context, principle violations, and verdict, preventing future re-litigation.
2. **Explicit alternatives** — each rejection names the Orthon mechanism that replaces the rejected concept.
3. **Deferral is documented, not abandoned** — deferrable concepts carry rationale and a target version, keeping them actionable for v0.2/v0.3.

## Deviations from Plan

None — plan executed exactly as specified. 4 rejection EDRs created (EDR-075 through EDR-078), 4 files moved to `reject/`, deferral rationale documented, INDEX.md and DECISION_PIPELINE.md updated.

## Files Created/Modified

- **4 rejection EDRs** in `how/decision_records/architecture/` — EDR-075 through EDR-078
- **4 files moved** from `deferrable/` to `reject/`
- **Updated:** `how/decision_records/INDEX.md`, `how/process/DECISION_PIPELINE.md`, `reject/README.md`, `deferrable/README.md`, `what/CORE_CONCEPTS.md`

## Self-Check

- [x] 4 rejection EDRs created and indexed (EDR-075 through EDR-078)
- [x] All 4 rejected files present in `how/concepts/research/reject/`
- [x] Deferral rationale documented for deferrable-tier concepts
- [x] INDEX.md updated with rejection EDR entries (Status: Rejected)
