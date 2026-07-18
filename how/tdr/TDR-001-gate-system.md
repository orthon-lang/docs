# TDR-001: Decision Validation Gate System

**Status:** Proposed

**Date:** 2026-07-18

**Domain:** Process

**Milestone:** 0

---

### Context

Language design proposals risk being evaluated through a single
perspective — typically the one most familiar to the evaluator.
Without systematic multi-perspective validation, proposals can pass
on technical merit while failing on user value, long-term
maintainability, or architectural integrity.

The Orthon project needs a validation framework that ensures every
proposal is examined through independent, complementary lenses
before entering the formal specification.

### Decision

Adopt a system of **six independent validation gates**, each applying
a distinct reasoning method:

| Gate | Method | Perspective |
|------|--------|-------------|
| `USER_VALUE_GATE` | Working Backwards | Does this solve a real problem? |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Is it internally consistent? |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Is it as simple as possible? |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Does it fit the architecture? |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Is it strategy-agnostic? |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Will it age well? |

Gates are **independent**: a proposal that passes one may fail another.
Each produces a binary verdict (pass / fail) or a conditional flag.
Any **fail** or unresolved **flag** sends the proposal back for revision.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Single-perspective validation | High | Proposals validated only by author's preferred lens |
| Confirmation bias | High | Evaluator finds what they expect to find |
| Groupthink | Medium | Team converges on consensus without critical examination |
| Hidden contradictions | High | Inconsistencies discovered late, when fix is expensive |

### Scope

| Applies to | Does NOT apply to |
|------------|-------------------|
| All new language constructs | Documentation fixes |
| Syntax changes | Typo corrections |
| Semantic refinements | Compiler optimisations (partial gates only) |
| Standard Library additions (partial) | |

### Relationship to Other Tools

| Tool | Relationship |
|------|-------------|
| TDR-002 (Validation Methods) | Each gate applies one method |
| TDR-003 (Checklist) | Checklist operationalises gates into review form |
| TDR-006 (Concept Design Review) | Gates evaluate the output of the 11-step procedure |

### Consequences

- **Positive:**
  - Catches failures early — before spec, before freeze
  - Each gate prevents a specific, named failure mode
  - Independent gates mean no single bias dominates
- **Negative:**
  - Adds process overhead — 6 gates per proposal
  - Requires discipline to apply consistently
  - Risk of perfunctory gate-checking without the checklist (TDR-003)

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Single comprehensive review | Single perspective; no guarantee of coverage |
| Ad-hoc peer review | Unstructured; depends on reviewer's experience and biases |
| Automated lint/checks only | Cannot verify semantic or philosophical criteria |

### Evolution

- Gates can be **extended** with new gates if a new failure mode is
  discovered that no existing gate covers.
- Gates can be **deprecated** if the failure mode they prevent is no
  longer relevant (unlikely for core validation perspectives).
- A gate may be **split** if it grows to cover two distinct concerns.

### Affected Documents

- [x] `DECISION_VALIDATION.md`
- [ ] `FITNESS_FUNCTIONS.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `ARCHITECTURE.md`
- [x] `_language-design.md`
- [x] `concept-design-review.md`
