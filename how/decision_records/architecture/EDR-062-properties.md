# EDR-062: Properties — Getter/Setter Sugar Over Attribute Access

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How does a language expose the data of a type without coupling consumers to its internal representation? Properties unify field storage and computed access behind a uniform `.name` interface. The caller writes `obj.name` regardless of whether it is a stored field or a computed value.

The concept of "a named value with getter/setter" is entirely expressible via existing primitives (attribute access + function calls). The syntactic sugar of implicit getter generation is a convenience, not new semantics. No compiler-level runtime behaviour changes are introduced.

---

### Decision

Properties are a **StdLib** pattern:
- Every field in a type declaration is implicitly a property with a getter and optional setter.
- A property declared without a getter body auto-generates a trivial getter (return backing field).
- Computed properties specify the getter body explicitly.
- Callers use `.name` syntax uniformly — stored and computed properties are indistinguishable at the call site.
- Changing a stored property to a computed one never changes the call site.

---

### Consequences

- **Positive:**
  - Uniform access eliminates refactoring friction.
  - No new compiler semantics — properties desugar to field access + function calls.
  - Computed properties enable encapsulation without boilerplate.
- **Negative:**
  - Properties do not introduce new capabilities — they are sugar over existing primitives.
  - The programmer must still distinguish stored from computed when reading the declaration.

---

### Compliance

- Property declarations must desugar to field declarations + getter/setter function implementations.
- Uniform access must be maintained — the call site must not distinguish stored from computed.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Language-level properties (C#/Kotlin) | Adds compiler complexity for sugar — no new semantics to justify it. |
| Explicit getters/setters (Java) | Breaks uniform access — refactoring field to computation changes the call site. |
| No property concept | Raw field access only — no encapsulation mechanism at call site. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Handles the common case without ceremony. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Property semantics are a direct superset of field semantics. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One concept — named value with optional computed getter/setter. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Properties desugar to existing primitives — no architectural impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Property sugar is independent of any specific runtime implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Proven pattern from C#, Kotlin, Swift — lowest-risk addition. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Obvious syntax — `field: Type { get: ... set: ... }` — easy for LLMs. |
