# EDR Template — Architecture Decisions

> Use this template for Architecture-category EDRs.
> Inherits all fields from `_edr.md`. No additional Architecture-specific
> sections at this time — the base EDR template is sufficient.
>
> Place each Architecture EDR as `docs/how/decision_records/architecture/EDR-NNN-slug.md`.

---

## EDR-NNN: [Short Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]

**Date:** YYYY-MM-DD

**Category:** Architecture

**Scope:** [Platform | Subsystem | Module | Project]

---

### Context

What design tension prompted this architectural decision?

Describe the forces: technical constraints, philosophical principles,
trade-offs, conflicting requirements. Reference VISION.md, ../DESIGN_PRINCIPLES.md,
or ARCHITECTURE.md as appropriate.

---

### Decision

What architectural decision did we make?

---

### Consequences

- **Positive:**
  - …
- **Negative:**
  - …

---

### Compliance

How will we verify this architectural decision is followed?

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| … | … |

### Gate Validation

> **Required for all Architecture-category EDRs.**
> See `DECISION_VALIDATION.md` § Gate Selection for gate applicability rules.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass / Fail / Flag | |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass / Fail / Flag | |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass / Fail / Flag | |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass / Fail / Flag | |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass / Fail / Flag | |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass / Fail / Flag | |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass / Fail / Flag | |

**Gates not applied:** [list with rationale, referencing DECISION_VALIDATION.md § Gate Selection]

**Detailed reasoning:** See `DECISION_LOG.md` entry for this EDR for per-gate reasoning trail.
