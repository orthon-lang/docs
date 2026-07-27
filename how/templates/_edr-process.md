# EDR Template — Process Decisions

> Use this template for Process-category EDRs.
> Inherits all fields from `_edr.md`, plus two Process-specific sections:
> **Without It** and **Evolution**.
>
> Place each Process EDR as `docs/how/decision_records/process/EDR-NNN-slug.md`.

---

## EDR-NNN: [Short Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]

**Date:** YYYY-MM-DD

**Category:** Process

**Scope:** [Project | Team | Policy]

---

### Context

What problem in the development process does this solve?
What failure mode does it prevent?
What triggered the need for this process decision?

> *Anchor in the project's process philosophy.*

---

### Decision

What process or approach did we adopt?

---

### Without It

What do we lose if this process is removed or skipped?

| Risk | Severity | Manifestation |
|------|----------|---------------|
| … | … | … |

---

### Consequences

- **Positive:**
  - …
- **Negative:**
  - …

---

### Evolution

Under what circumstances should this process be deprecated, extended, or replaced?

---

### Compliance

How will we verify this process is followed?

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| … | … |

---

### Gate Validation

> Process decisions affect *how* the project operates, not *what* the
> language is. They must balance rigour against overhead — a process
> that is not followed is worse than no process at all.
>
> See `DECISION_VALIDATION.md` § Gate Selection for methodology
> references.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass / Fail / Flag | Does this process solve a real problem? Process overhead must be justified. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass / Fail / Flag | Is the process internally consistent? Does it contradict other processes? |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass / Fail / Flag | Is the process as simple as it can be? Does the benefit outweigh the overhead? |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass / Fail / Flag | Will this process be sustainable over time? What keeps it from bit-rotting? |

**Gates not applied:**
| Gate | Rationale |
|------|-----------|
| `ARCHITECTURAL_INTEGRITY_GATE` | Process decisions don't affect language architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Process decisions are about project operations, not implementation strategies. |
| `LLM_GENERABILITY_GATE` | Process decisions don't produce code — LLM generability is not applicable. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for this EDR for per-gate reasoning trail.
