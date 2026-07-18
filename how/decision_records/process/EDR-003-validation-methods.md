# EDR-003: Validation Methods

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

**Supersedes:** TDR-002

---

### Context

Each validation gate needs a structured reasoning method — a
disciplined technique that guides the evaluator through the
validation, rather than relying on unstructured opinion.

Without explicit methods, gate evaluation becomes subjective:
evaluators apply their own mental habits, which may miss the
specific failure mode the gate is designed to catch.

### Decision

Adopt six distinct reasoning methods, each chosen for its fit with
a specific gate's perspective:

| Method | Gate | Cognitive Mode | What it prevents |
|--------|------|---------------|------------------|
| Working Backwards | `USER_VALUE_GATE` | Empathic | Building features nobody asked for |
| Socratic Method | `LOGICAL_CONSISTENCY_GATE` | Critical | Contradictions hidden by imprecise language |
| Scientific Method | `CONCEPTUAL_SIMPLICITY_GATE` | Empirical | Over-engineering disguised as completeness |
| Logical Analysis | `ARCHITECTURAL_INTEGRITY_GATE` | Deductive | Layering violations that seem harmless |
| TRIZ | `IMPLEMENTATION_INDEPENDENCE_GATE` | Inventive | Strategy compromises accepted too early |
| Einstein's Method | `LONG_TERM_MAINTAINABILITY_GATE` | Reductive | Complexity that becomes permanent debt |

Each method has its own document under `gates/methods/` with the
full technique, application steps, and the gate it serves.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Unstructured validation | High | Each evaluator applies their own mental model |
| Method-gate mismatch | Medium | Right question, wrong technique to answer it |
| Cognitive bias dominance | High | Single cognitive mode dominates all evaluations |

### Consequences

- **Positive:**
  - Each gate has a proven, teachable technique
  - Methods cover complementary cognitive modes — no single bias
  - Methods are reusable across multiple proposals
- **Negative:**
  - Learning curve — evaluators must understand 6 distinct methods
  - Risk of mechanical application without understanding the spirit

### Evolution

- Methods can be **replaced** if a better technique is discovered for a gate.
- Methods can be **added** if new gates are introduced.

### Compliance

Verified through EDR-004 (Checklist) which prompts method application per gate.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Single universal method | No single method covers all cognitive modes |
| Ad-hoc per-proposal method selection | Inconsistent; no guarantee of coverage |
| Checklist-only (no methods) | Checklist says *what* to check, not *how* to think |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-002 (Gate System) | Methods are the engines that power the gates |
| EDR-004 (Checklist) | Checklist prompts method application |
