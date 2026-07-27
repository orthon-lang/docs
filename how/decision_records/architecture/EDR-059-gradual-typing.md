# EDR-059: Gradual Typing — Optional Type Annotations with Selective Type Checking

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

How does a language support both rapid prototyping (dynamic typing, no annotation overhead) and production safety (static checking, compile-time guarantees)? Languages typically force a permanent choice between the two modes. Gradual typing allows the programmer to start dynamic and add types incrementally as the codebase matures.

The ability to selectively enable/disable type checking at module boundaries requires compiler-level infrastructure: type inference that works on unannotated code, boundary checks between typed and untyped code, and a global consistency pass.

---

### Decision

Gradual typing is a **Language** construct:
- Types may always be omitted — inference provides type information even for unannotated code.
- Type annotations on function signatures and public APIs act as compiler-checked boundaries.
- The boundary between typed and untyped code is verified by the compiler (boundary checks).
- No separate declaration files — type information lives alongside code.
- REPL and one-off scripts operate fully dynamically.
- Schema Provider exposes typing level per module for LLM context.

---

### Consequences

- **Positive:**
  - Same language scales from prototyping to production.
  - LLM-generated code can start with minimal annotations.
  - Boundary checks provide safety where it matters most — public APIs.
  - No separate type declaration files to maintain.
- **Negative:**
  - Compiler complexity — must support both fully statically-typed and gradually-typed compilation modes.
  - Boundary check overhead at typed/untyped interfaces.
  - Some type errors may only be caught at runtime in untyped code.

---

### Compliance

- Compiler must support type inference for unannotated code.
- Boundary checks must be inserted at typed/untyped function call interfaces.
- The global consistency pass must run as an optional lint.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Fully static (no gradual typing) | Forces annotation overhead during prototyping — unacceptable for REPL-first design. |
| External type checker (TypeScript/`mypy`) | Type information lives in separate files — violates self-describing principle. |
| Dynamic only | No compile-time safety — unacceptable for production use. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Start dynamic, add types as code matures — direct match with programmer workflow. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Boundary check model is consistent: typed/untyped interaction is well-defined. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept — optional annotations — covers the entire spectrum. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Gradual typing composes with type inference and module system. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Semantics independent of specific inference algorithm. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Flag | Gradual typing adds compiler complexity; well-understood from TypeScript ecosystem. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Critical for LLM adoption — LLM can generate minimal-annotation code and let the compiler catch structural errors. |
