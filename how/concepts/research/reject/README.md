# Rejected Concepts

> Research concepts that have been formally rejected via Engineering Decision Record.
> These concepts are structurally incompatible with Orthon's core design principles and
> will not appear in any future version of the language specification.

## Purpose

The `reject/` directory archives research documents for concepts that were formally evaluated
and rejected. Unlike the `deferrable/` tier (where concepts may be revisited for v0.2/v0.3),
rejected concepts are not candidates for future inclusion — they contradict fundamental
Orthon principles and accepting them would require a Tier 1 EDR changing those principles.

## Key Distinction: Rejected vs. Deferred

| Aspect | Rejected | Deferred |
|--------|----------|----------|
| **Decision** | Structurally incompatible | Not yet ready / lower priority |
| **Target version** | None — never | v0.2 or v0.3 |
| **Re-entry path** | Tier 1 EDR changing core principles | Standard Decision Pipeline |
| **Principle conflict** | Direct and unresolvable | None or manageable |

## Current Rejected Concepts

| Concept | EDR | Rejection Rationale (Summary) |
|---------|-----|-------------------------------|
| Prototype-based Object Model | [EDR-075](../../../../how/decision_records/architecture/EDR-075-reject-prototype.md) | Violates Data First (conflates data and behaviour), Explicitness (implicit delegation), Orthogonality (composition and delegation merged) |
| Significant Whitespace | [EDR-076](../../../../how/decision_records/architecture/EDR-076-reject-significant-whitespace.md) | Violates Explicitness (formatting determined semantics), Consistency (LLM generation errors), Semantics Before Optimization (typing convenience over correctness) |
| Dynamic Typing | [EDR-077](../../../../how/decision_records/architecture/EDR-077-reject-dynamic-typing.md) | Violates Declarative With Static Guarantees (runtime deferral), Explicitness (runtime errors), Correctness Before Performance (iteration speed over correctness) |
| Class/Structure as Primary Composition | [EDR-078](../../../../how/decision_records/architecture/EDR-078-reject-class-or-structure-as-primary-composition.md) | Violates Data First (behaviour imposed on data), Minimal Core (class is composition of primitives, not a primitive), Orthogonality (class hierarchy coupling) |

## Lifecycle

1. A concept is researched and placed in the `deferrable/` tier.
2. During Phase 4 (Decision Pipeline), the concept is formally evaluated through the pipeline questions and gate validation.
3. If found structurally incompatible with Orthon's principles, a rejection EDR is created.
4. The research document is copied to `reject/` with a header note linking to the rejection EDR.
5. The original `deferrable/` file is removed.

Rejected concepts are preserved for historical reference — they document why certain design paths
were closed and prevent re-litigation of settled decisions.
