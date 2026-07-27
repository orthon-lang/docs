# EDR-066: Sorting — Stable Sort Algorithm as StdLib

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Sorting data by keys is a fundamental operation. The key design decision is stability: when sorting data multiple times by different keys, should the relative order of equal elements be preserved? Stability guarantees compose; instability forces the programmer to sort by all keys simultaneously.

Sort algorithms are library implementations. The stability guarantee is a policy choice that lives in the StdLib's algorithm selection.

---

### Decision

Sorting is a **StdLib** concept:
- The default sort algorithm is Timsort (stable, hybrid merge+insertion sort).
- Stability is guaranteed by default for all sorting operations.
- An explicit `sort_unstable` variant is available for performance-critical use.
- The sort algorithm is a configurable Algorithm Policy (IMPLEMENTATION_POLICIES.md).

---

### Consequences

- **Positive:**
  - Stability composes — multiple sort passes produce predictable results.
  - Opt-in instability for performance.
- **Negative:**
  - Stable sort has a small constant factor overhead vs unstable.
  - Timsort is not the fastest for all data patterns.

---

### Compliance

- The default `sort` method must be stable.
- `sort_unstable` must preserve no ordering guarantees for equal elements.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Unstable by default | Silent correctness bugs on multi-pass sorting. |
| Stability implementation-defined | Non-portable behaviour — violates Deterministic Behavior principle. |
| No sort in StdLib | Sorting is a fundamental collection operation — omission would be widely noted. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Stable sort by default — least surprise. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Stability composes predictably. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One default algorithm, one opt-in variant. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | StdLib method on collection types — no architectural impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Sort policy can change without affecting language semantics. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Timsort is proven across Python, Java, and Android. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `list.sort()` is the most LLM-obvious API. |
