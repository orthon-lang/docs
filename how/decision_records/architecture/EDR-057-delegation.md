# EDR-057: Delegation — Composition via Delegation Pattern

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How does a type reuse behaviour from another type without inheritance? Two problems:
1. **Class delegation** — forwarding all interface methods to a contained instance.
2. **Property delegation** — getter/setter behaviour delegated to a helper object.

These are expressible via composition of existing primitives (trait implementation + manual method forwarding). The `@delegate` macro desugars to explicit `impl` blocks using the existing macro system (EDR-029). No new compiler semantics are required.

---

### Decision

Delegation is a **StdLib** concept:
- Class delegation via `@delegate(Trait) to field` macro (registered in the macro registry per EDR-029).
- Property delegation via StdLib delegate protocols (`lazy`, `observable`, `vetoable`, `map`).
- The compiler generates forwarding code for `@delegate` annotations but no new delegation-specific syntax.
- Selective override: explicit method definitions take precedence over delegated methods.
- No implicit promotion — delegation must be declared explicitly.

**Note:** DELEGATION (this concept) is distinct from DELEGATE (the execution policy from EDR-036). The execution `delegate` creates concurrent execution contexts; delegation pattern composes types. These are completely orthogonal concepts that happen to share similar naming.

---

### Consequences

- **Positive:**
  - No new compiler-level semantics needed.
  - Reuses the existing macro system (EDR-029) for code generation.
  - Class delegation replaces boilerplate forwarding methods.
  - Property delegation provides reusable property behaviours.
- **Negative:**
  - `@delegate` depends on compile-time execution (EDR-031).
  - Property delegation requires the delegee to implement `Get`/`Set` protocol.
  - No compiler-level checking that delegation is semantically valid.

---

### Compliance

- The StdLib must provide `lazy`, `observable`, `vetoable`, and `map` delegate implementations.
- `@delegate` macro must generate correct forwarding `impl` blocks.
- Method resolution must give precedence to explicit methods over delegated ones.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Language-level `by` keyword (Kotlin) | Adds syntax and compiler complexity for a pattern expressible via macros + StdLib. |
| Implicit promotion (Go) | Forwarding is invisible at the definition site — violates Explicitness. |
| Manual forwarding only | High boilerplate; defeats the purpose of delegation. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Eliminates boilerplate forwarding — direct programmer value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Forwarding semantics are well-defined. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One macro keyword (`@delegate`) covers class delegation. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on existing macro system (EDR-029). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Delegation semantics independent of macro implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | StdLib-based means no long-term language surface commitment. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | One annotation with clear semantics — easy for LLMs to generate. |
