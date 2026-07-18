# TDR-004: Fitness Functions

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Fitness Function

**Milestone:** 0

---

### Context

Gates validate proposals at a point in time. But architectural
health decays gradually — through accumulated revisions, scope
creep, and well-intentioned changes that individually pass gates
but collectively erode the design.

The project needs measurable, continuous guards that signal when
the architecture is drifting from its intended shape — not just
at proposal time, but across the entire design lifecycle.

### Decision

Adopt a catalogue of **Fitness Functions** — measurable architectural
checks applied when a concept is revised or a new concept is
introduced.

Three initial fitness functions:

| Fitness Function | What it measures |
|-----------------|-----------------|
| Concept Stability | Semantic churn after gate approval — signals poor factoring |
| Layered Isolation | Cross-layer ripple — signals boundary breach |
| Composition Surface | Canonical forms count — must not decrease across revisions |

Fitness Functions are checked alongside Gates before a concept is
approved. New functions can be added as new architectural concerns
are identified.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Undetected architectural decay | High | Design drifts from ARCHITECTURE.md without signal |
| Concept instability unnoticed | Medium | A concept is revised repeatedly without root-cause analysis |
| Layer breaches accumulate | High | Strategy concerns leak into Core definitions |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| Every concept revision | New concepts (first approval uses Gates only) |
| Cross-cutting changes | Documentation changes |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-001 (Gate System) | Fitness Functions complement Gates with continuous measurement |
| TDR-009 (Layered Architecture) | Fitness Functions guard the architecture defined in TDR-009 |

### Consequences

- **Positive:**
  - Early warning of architectural drift
  - Measurable, not subjective
  - Extensible catalogue
- **Negative:**
  - Adds another check before approval
  - Requires discipline to measure consistently

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Rely on gates alone | Gates are point-in-time; don't catch gradual drift |
| Periodic architecture reviews | Too infrequent; drift accumulates between reviews |
| No fitness functions | Accepts architectural decay as inevitable |

### Evolution

- New fitness functions can be **added** as new architectural
  concerns emerge.
- Functions can be **retired** if the concern they measure is no
  longer relevant.
- Functions should remain **measurable** — avoid subjective criteria.

### Affected Documents

- [ ] `DECISION_VALIDATION.md`
- [x] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [x] `ARCHITECTURE.md`
- [ ] `_language-design.md`
- [ ] `concept-design-review.md`
