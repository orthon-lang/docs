# EDR-002: Decision Validation Gate System

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-001

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

### Consequences

- **Positive:**
  - Catches failures early — before spec, before freeze
  - Each gate prevents a specific, named failure mode
  - Independent gates mean no single bias dominates
- **Negative:**
  - Adds process overhead — 6 gates per proposal
  - Requires discipline to apply consistently
  - Risk of perfunctory gate-checking without the checklist (EDR-004)

### Evolution

- Gates can be **added** when a new failure mode is identified.
- Gates can be **removed** if a failure mode becomes irrelevant.
- The gate system itself is validated through EDR-001 (EDR System).

### Compliance

Verified through EDR-004 (Checklist) and EDR-007 (Concept Design Review).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Single reviewer approval | No systematic coverage; bias dominates |
| Peer review without gates | Unstructured; misses specific failure modes |
| Automated checks only | Cannot verify philosophical or user-value criteria |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-003 (Validation Methods) | Each gate applies one method |
| EDR-004 (Checklist) | Checklist operationalises gates into review form |
| EDR-007 (Concept Design Review) | Gates evaluate the output of the 11-step procedure |
