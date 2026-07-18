# EDR-007: Concept Design Review

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-006

---

### Context

Language concepts — Data, Functions, Equality, Ownership, etc. —
form the vocabulary of Orthon. Designing a concept in isolation
risks inconsistency with other concepts, violations of design
principles, and architectural erosion.

Without a structured design procedure, concept quality becomes
dependent on individual designer discipline — some concepts will
be thoroughly examined, others will not.

### Decision

Adopt an **11-step Concept Design Review** procedure documented in
`docs/how/concept-design-review.md`.

The 11 steps are:

| Step | Activity | Gate Alignment |
|------|----------|---------------|
| 1 | Anchor in Vision | USER_VALUE_GATE |
| 2 | State the Problem | USER_VALUE_GATE |
| 3 | Define Core Semantics | LOGICAL_CONSISTENCY_GATE |
| 4 | Check Design Principles | CONCEPTUAL_SIMPLICITY_GATE |
| 5 | Verify Orthogonality | CONCEPTUAL_SIMPLICITY_GATE |
| 6 | Minimal Viable Semantics | CONCEPTUAL_SIMPLICITY_GATE |
| 7 | Architectural Integrity | ARCHITECTURAL_INTEGRITY_GATE |
| 8 | Implementation Independence | IMPLEMENTATION_INDEPENDENCE_GATE |
| 9 | Long-term Maintainability | LONG_TERM_MAINTAINABILITY_GATE |
| 10 | Interaction Analysis | All gates |
| 11 | EDR (Architecture) | Decision formalisation |

Each step produces concrete output and aligns with one or more
validation gates.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Inconsistent concept quality | High | Some concepts well-designed, others rushed |
| Missed interactions | High | Concepts designed in isolation with hidden conflicts |
| Principle violations | Medium | Design principles ignored through oversight |

### Consequences

- **Positive:**
  - Every concept receives consistent, thorough examination
  - Steps map directly to validation gates
  - Final step produces an EDR — decisions are never lost
- **Negative:**
  - 11 steps is significant overhead per concept
  - Risk of mechanical execution without genuine design thinking

### Evolution

Steps can be **reordered** or **added/removed** as the design process matures.

### Compliance

Every concept entering Milestone 2 must complete the 11-step procedure.
Step 11 produces an Architecture-category EDR.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Ad-hoc concept design | Inconsistent quality; no traceability |
| Gates only (no procedure) | Gates evaluate, but don't guide the design process |
| Single-pass design review | Doesn't separate concerns; easy to miss dimensions |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-002 (Gate System) | Gates evaluate the output of each step |
| EDR-004 (Checklist) | Checklist is applied after the procedure |
| EDR-005 (Fitness Functions) | Checked alongside gates before approval |
| EDR-001 (EDR System) | Step 11 produces an Architecture-category EDR |
