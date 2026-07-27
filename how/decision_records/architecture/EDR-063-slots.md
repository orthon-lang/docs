# EDR-063: Slots — Compact Field Storage Annotation

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How does a type declare a fixed, known set of attributes — restricting which fields can exist at runtime — for memory efficiency, safety, and programmer intent? In Orthon's property-based model, every declared property is inherently a slot. The concept collapses into "a property with a backing field." No separate declaration mechanism is needed beyond declaring the property itself.

---

### Decision

Slots are a **StdLib** pattern:
- Every property declaration in a type implicitly creates a slot — no separate `__slots__`-like declaration.
- The compiler uses fixed-size layout by default for all declared types.
- Dynamic attribute extension requires an explicit `dynamic` modifier on the type.
- The StdLib provides a `Slot` annotation for explicit storage control when needed.

---

### Consequences

- **Positive:**
  - Declaring a property is sufficient to reserve a slot — no separate mechanism.
  - Fixed-field layout is the default, providing predictable memory layout.
  - Dynamic extension is visible when opted into.
- **Negative:**
  - No way to have dynamic attributes without declaring `dynamic`.
  - Inheritance of slots is implicit (property declarations are inherited).

---

### Compliance

- The compiler must use fixed-size layout for all declared types by default.
- `dynamic` modifier must be explicitly required for attribute extension.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Always dynamic (Python) | Per-instance dict overhead — unacceptable for systems-level use. |
| Opt-in fixed (Python `__slots__`) | Requires separate declaration — redundant when properties already declare slots. |
| Always fixed (Rust) | No dynamic escape hatch — Orthon's progressive disclosure principle requires one. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Predictable memory layout without ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Every declared property implicitly reserves a slot — consistent. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Slots are a consequence of the property model, not a separate feature. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with existing property and type system. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Slot semantics independent of storage implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Minimal surface — slots are implicit in property declarations. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs declare properties; slots follow automatically. |
