# EDR-065: Named and Optional Parameters — Function Call Ergonomics

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How should a language make function and constructor calls easier to read and maintain? Named arguments allow parameters to be specified by name in any order. Optional parameters with defaults reduce the need for multiple overloads.

Named and optional parameters desugar to positional parameters + default values. The parameter binding and default value generation are desugaring operations — no new runtime semantics.

---

### Decision

Named and optional parameters are a **StdLib** convention:
- Call sites may specify parameters by name, improving readability.
- Call sites may omit parameters that have declared default values.
- Named arguments may appear in any order.
- Positional and named arguments may be mixed (positional first, then named).
- Default values are evaluated at call time (lazily).
- Named/optional parameters interact with overload resolution (named arguments disambiguate overloads).

---

### Consequences

- **Positive:**
  - Reduces overload explosion — optional parameters replace multiple overloads.
  - Named arguments make call sites self-documenting.
  - Adding a new optional parameter does not break existing callers.
- **Negative:**
  - Default value evaluation timing must be specified (call-time).
  - Overload resolution with named arguments adds compiler complexity.

---

### Compliance

- The compiler must resolve named arguments to parameters.
- Overload resolution must consider named argument matching.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Positional only | Overload explosion — many variants for the same function. |
| Named-only (Smalltalk) | Verbose for simple calls where order is natural. |
| Builder pattern | No language support — every builder is hand-written. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Self-documenting call sites — direct readability improvement. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Named resolution has well-defined rules — no ambiguity. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept — named binding — covers both features. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Desugars to positional parameters — no architectural impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Semantics independent of argument-passing implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven from Python, C#, Kotlin, Swift — lowest-risk feature. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Named arguments are the most LLM-friendly call syntax — explicit mapping from name to value. |
