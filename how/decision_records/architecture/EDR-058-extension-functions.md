# EDR-058: Extension Functions — Method-Call Syntax on External Types

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How does a programmer add behaviour to an existing type without modifying its source, extending it via inheritance, or wrapping it in an adapter? Extension functions allow a function defined outside a type to be called with method-call syntax on values of that type.

The compiler must recognize receiver-call syntax on types from other modules and resolve dispatch based on the static type. This requires name resolution rules, precedence handling (member wins), and import control — all compiler-level services.

---

### Decision

Extension functions are a **Language** construct:
- Defined outside the receiver type but called with receiver syntax.
- No access to private members of the receiver type.
- Static dispatch based on the static type of the receiver.
- Explicit import required for extension functions from other packages.
- Member function takes precedence over extension function of the same signature.
- Extension properties supported (computed only, no backing fields).

---

### Consequences

- **Positive:**
  - Enables adding methods to any type without modifying source.
  - Preserves encapsulation (no access to private members).
  - Static dispatch avoids runtime overhead.
  - Import control prevents namespace pollution.
- **Negative:**
  - Adds name resolution complexity (extension functions vs members, cross-package resolution).
  - Ambiguity possible when two imports define the same extension function.
  - Does not participate in trait/interface satisfaction.

---

### Compliance

- Compiler must resolve extension function calls to the correct definition based on receiver static type.
- Extension function import must be explicit.
- Member function must shadow extension function of the same signature.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Trait-based (Rust) | Requires trait declaration per extension group — more ceremony than needed for one-off extensions. |
| Monkey-patching (Python) | No access control, fragile, unpredictable — unacceptable for a language with static guarantees. |
| No extension functions | Utility function syntax is less natural for call chains. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Natural call syntax on any type — direct programmer value. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Static dispatch, member precedence, explicit import — consistent rules. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept: function defined outside type, called with receiver syntax. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with name resolution and import system. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Semantics independent of dispatch implementation (static or vtable). |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Kotlin/C#/Swift — proven pattern. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Obvious syntax — `fun Type.method()` — LLMs generate correctly from natural language descriptions. |
