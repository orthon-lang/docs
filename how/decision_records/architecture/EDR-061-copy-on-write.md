# EDR-061: Copy-on-Write — Memory Optimisation for Value Semantics

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Memory Model)

---

### Context

How does a language manage memory safely without forcing all programmers through a borrow checker learning curve? Copy-on-write (CoW) is a memory optimisation technique where assignment shares data and mutation triggers a clone only when the data is shared.

CoW is an implementation technique for value-semantics collections. The programmer does not write CoW-specific code — they write value-semantics code and the compiler/runtime optimises via CoW. This is a memory optimisation strategy, not a new semantic concept.

---

### Decision

Copy-on-write is classified as **StdLib / Implementation Strategy**:
- CoW is the DEFAULT_STRATEGY's chosen mechanism for implementing value semantics on standard collections.
- The SEMANTIC_MODEL specifies value semantics by default; CoW is how the runtime achieves this efficiently.
- `shared` keyword makes reference semantics explicit — values under `shared` use RC-based sharing, not CoW.
- CoW eliminates the need for a borrow checker in common patterns.
- The language surface does not expose CoW — it is an invisible optimisation.

---

### Consequences

- **Positive:**
  - Programmers write value-semantics code without thinking about memory management.
  - Avoids borrow checker learning curve for common patterns.
  - Deterministic destruction (no GC pauses).
  - `shared` provides explicit escape hatch for shared mutable state.
- **Negative:**
  - CoW has runtime cost on shared mutation — the clone operation cannot be elided.
  - Cycle handling in `shared` requires backup GC or cycle detection.
  - Some patterns (linked lists, graphs) require `shared` and its RC overhead.

---

### Compliance

- The DEFAULT_STRATEGY must specify CoW parameters (when cloning triggers, reference counting discipline).
- The SEMANTIC_MODEL must specify value semantics as the default.
- `shared` must make identity sharing syntactically visible.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Borrow checker (Rust) | Higher cognitive load than CoN's target audience requires. |
| Tracing GC (Java, Go) | Non-deterministic destruction violates Orthon's design principles. |
| Pure value copying | Every assignment triggers deep copy — unacceptable performance for collections. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Write value-semantics code; CoW handles performance transparently. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | CoW preserves value semantics — assignment is a logical copy. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One optimisation technique covers the default collection strategy. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | CoW is an implementation choice within the Strategy system. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Value semantics are specified; CoW is one possible implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | CoW is a well-understood technique (Swift, Clojure, many systems). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | CoW is transparent — LLMs never write CoW-specific code. |
