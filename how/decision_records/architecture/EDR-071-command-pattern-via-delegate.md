# EDR-071: Command Pattern via Delegate — Obsoleted by First-Class Functions

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

The GoF Command pattern encapsulates a request as an object. In languages with first-class functions (delegates, closures), the pattern is entirely obsoleted — any callable value serves the same purpose with less ceremony. Orthon's delegate model (EDR-036) provides first-class functions, making the Command pattern redundant.

The concept exists only to document this obsolescence. No new constructs are needed.

---

### Decision

Command Pattern via Delegate is a **StdLib** (documentation-only) concept:
- The delegate model (EDR-036) obsoletes the Command pattern.
- The StdLib documents the isomorphism between OOP patterns and delegate forms.
- No Command-specific types, macros, or annotations are introduced.
- The documentation notes that PATTERN_MATCHING_DISPATCH (EDR-026) provides alternative command routing.

---

### Consequences

- **Positive:**
  - No new language surface — the concept documents what already exists.
  - Eliminates class explosion for trivial command implementations.
  - Single concept (delegate) replaces multiple OOP patterns.
- **Negative:**
  - Developers coming from OOP backgrounds may look for a Command interface that doesn't exist.
  - Documentation must explicitly address this gap.

---

### Compliance

- The StdLib must document the delegate-as-command pattern.
- No `Command` interface or trait should be introduced.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Traditional Command interface | Requires class per command — ceremony without benefit. |
| Enum-based command dispatch | Useful for routing but not a substitute for general command pattern. |
| Annotation-based command | `@Command` annotation — adds surface for something already expressible. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Developer writes a lambda, not a class — direct ergonomic gain. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Delegate model subsumes all command arities. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Fewer concepts — one primitive (delegate) replaces multiple patterns. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | No new architecture — the concept documents existing capabilities. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Documentation-only — no implementation dependency. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | No surface area — no maintenance burden. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs naturally generate lambdas/closures — no need to teach a Command abstraction. |
