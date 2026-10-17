# EDR-065: Named and Optional Parameters — Function Call Ergonomics

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

**Amended by:** [EDR-096](./EDR-096-redesigned-unpacking-syntax.md) (call-site named-argument spelling).

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

---

### Amendments

**2026-10-17 — Call-site spelling amended by [EDR-096](./EDR-096-redesigned-unpacking-syntax.md) (D-06, D-12).**
Call-site named arguments are written `value as name` (for example `connect("example.com", true as use_ssl)`), replacing `name: value` at call sites. Parameter declarations keep `name: Type`, because the colon stays reserved for type annotations. A label overrides position when binding arguments, and duplicate labels are errors. Defaults are unchanged and belong to the whole parameter, not to a position inside a pattern. Whether variadic parameters and call-site spreading exist in v0.1 is the open item DEF-a in EDR-096 and Open Questions item 3 of `what/concepts/NAMED_AND_OPTIONAL_PARAMETERS.md`. Status stays Accepted; the original body above is retained for history.
