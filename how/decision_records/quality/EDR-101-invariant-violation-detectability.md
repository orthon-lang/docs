# EDR-101: Invariant Violation Detectability

**Status:** Accepted

**Date:** 2026-11-03

**Category:** Quality

**Scope:** Project

---

### Context

The [fitness function catalogue](../../architecture/FITNESS_FUNCTIONS.md)
measures architectural properties, while [EDR-099](../process/EDR-099-nyquist-validation-method.md)
adopts Nyquist Validation to examine whether evidence can reveal a
significant semantic violation. A gate can pass even when its examples
would show the same outcome for a correct and an incorrect design.
This leaves significant failures at concept boundaries, across event
orders, or across implementation strategies undetected.

### Decision

Add Invariant Violation Detectability to the fitness function catalogue.
For each new or substantively revised concept, identify significant
semantic invariants and plausible violation scenarios. For each scenario,
record the canonical invariant, expected and violating outcomes, and an
observation that distinguishes them. If no significant invariant is
identified, record why the check does not apply.

Count identified significant scenarios without a discriminating
observation. A nonzero count blocks Human Sign-off until evidence is
strengthened or a scenario is shown not to be significant. A zero count
applies only to the scenarios identified; it does not establish complete
coverage. A documented scenario is design evidence, not a passed test.

### Consequences

- **Positive:** Reviewers can identify gaps in the evidence for semantic
  guarantees before a concept is approved.
- **Negative:** Identifying scenarios and observations adds review effort;
  the result depends on which scenarios reviewers identify.

### Compliance

Apply the catalogue entry during concept review and record the selected
scenarios, observations, and uncovered count in the review evidence.
Resolve significant uncovered scenarios before Human Sign-off. This
check supplements the existing gates and fitness functions.

### Alternatives Considered

| Alternative | Rationale for rejection |
|-------------|-------------------------|
| Rely on gate verdicts alone | A verdict does not show whether its evidence distinguishes the outcomes. |
| Require a fixed numerical coverage percentage | The set of possible violations is not known in full, so a percentage would overstate confidence. |
| Require executable tests now | The current repository contains a language design, not a compiler or runtime. |

### Gate Validation

| Gate | Verdict | Rationale |
|------|---------|-----------|
| User value | Pass | Reviewers need evidence that significant semantic failures can be noticed. |
| Logical consistency | Pass | The count concerns identified scenarios and does not claim universal coverage. |
| Conceptual simplicity | Pass | One count reuses the scenarios and observations required by EDR-099. |
| Long-term maintainability | Pass | The check can use executable evidence when implementations exist. |

### Relationship to Other Records

| Record | Relationship |
|--------|--------------|
| [EDR-005](EDR-005-fitness-functions.md) | Establishes the extensible fitness function catalogue. |
| [EDR-099](../process/EDR-099-nyquist-validation-method.md) | Defines Nyquist Validation, the method used by this function. |
