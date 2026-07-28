# EDR-073: Declarative Constructs — Documentation of Declarative Patterns

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

What makes a language construct "declarative" — and which common programming tasks should Orthon provide declarative constructs for? A construct is declarative when the programmer specifies *what* the result should be, and the language or StdLib determines *how* to achieve it.

This is a meta-concept that catalogues existing declarative patterns: collection transformations (`.filter()`, `.map()`, `.sorted()`), resource management (`using`), sorting (`.sorted(by:)`), and derived structural methods (`@derive`). All are already provided by existing concepts. No new constructs are introduced.

---

### Decision

Declarative Constructs is a **StdLib** (documentation-only) concept:
- The concept documents which patterns Orthon considers declarative.
- Each declarative construct has a specified desugaring to imperative primitives.
- No new language syntax — the language already provides the primitives for declarative patterns.
- The concept serves as reference: "if you need to do X, use the declarative form Y."

---

### Consequences

- **Positive:**
  - No new language surface — documents existing capabilities.
  - Provides clear guidance on preferred coding style.
  - Every declarative construct has a well-defined imperative desugaring.
- **Negative:**
  - Not a feature — impossible to "implement" as a single unit.
  - Declarative patterns are distributed across multiple existing concepts.

---

### Compliance

- Every documented declarative construct must have a specified imperative desugaring.
- The concept must reference the source concepts for each declarative pattern.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| New declarative syntax (comprehensions) | New surface — would require syntax design in Phase 5. Deferred to v0.2. |
| Framework-based declarative (LINQ) | External dependency — violates self-contained principle. |
| No documentation | Programmers may not realize the declarative form exists. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Documentation of best practices — direct programmer value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | All constructs are existing concepts — no inconsistency risk. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One document cataloguing existing patterns. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | No new architecture — pure documentation. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Documentation-only — no implementation dependency. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Must be kept in sync with evolving concepts — lightweight maintenance. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Documentation tells LLMs the preferred declarative form for each task. |
