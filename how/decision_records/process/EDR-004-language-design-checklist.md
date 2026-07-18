# EDR-004: Language Design Gate Checklist

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-003

---

### Context

Decision Validation Gates (EDR-002) define *what* must be checked.
Validation Methods (EDR-003) define *how* to think during evaluation.
But without a concrete checklist, there is no operational form —
evaluators may pass a gate superficially without checking all
criteria.

A checklist transforms the gates from philosophical framework into
an actionable review form.

### Decision

Adopt `_language-design.md` — a fill-in checklist that
operationalises the validation gates into a single concrete review
form. Each criterion in the checklist is traceable to one or more
validation gates, ensuring broad coverage.

The checklist includes: Vision alignment, Problem-first statement,
Principle compliance, Orthogonality, Minimality, Core stability,
Least Astonishment, Explicitness, Named equivalence, All canonical
forms, Impact analysis, Interaction analysis, Policy footprint,
Traceability, Gate guard, Evolution strategy, and Decision journal
entry.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Perfunctory gate evaluation | High | "Yes, it's consistent" without evidence |
| Incomplete coverage | Medium | Some criteria consistently skipped |
| Inconsistent reviews | Medium | Different reviewers check different things |

### Consequences

- **Positive:**
  - Consistent, thorough reviews across all proposals
  - Traceability from checklist item to specific gate
  - Decision journal provides project memory
- **Negative:**
  - Form-filling overhead
  - Risk of checklist becoming a ritual without genuine evaluation

### Evolution

- Checklist items can be **added** when a new gate is introduced.
- Checklist items can be **removed** if a criterion becomes obsolete.

### Compliance

The checklist itself is the compliance mechanism. Its application is
verified through EDR-007 (Concept Design Review).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Gates alone (no checklist) | Too abstract; no operational form |
| Automated checks only | Cannot verify philosophical or semantic criteria |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-002 (Gate System) | Checklist operationalises the gates |
| EDR-007 (Concept Design Review) | Checklist is applied after the 11-step procedure |
