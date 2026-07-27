# EDR-056: Contracts — Compiler-Enforced Pre/Postconditions

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

How does a language provide verifiable guarantees about a function's behaviour — both to human readers and to LLMs generating code? Three motivations drive the need for first-class contracts:

1. **LLM generation accuracy** — A typed signature gives structure but not intent. Contracts give both, reducing hallucination of nonsensical calls.
2. **Compiler-verified intent** — Contracts checked at compile time (where possible) detect violations at the earliest possible moment.
3. **Test synthesis** — Contracts are executable specifications from which the compiler can generate test cases.

The `requires`, `ensures`, and `invariant` keywords require syntactic integration into function signatures and compiler-level enforcement — not expressible via library composition.

---

### Decision

Contracts are a **Language** construct. The keywords `requires`, `ensures`, and `invariant` are part of the function signature syntax. The compiler:
- Verifies contracts statically where possible.
- Degrades to runtime assertions for dynamic conditions.
- Enforces contract expression purity (no side effects in contract expressions).
- Elides contracts in release builds unless `--enforce-contracts` is passed.
- Supports contract inheritance via Liskov substitution rules.

---

### Consequences

- **Positive:**
  - LLMs can infer function intent from contracts, reducing generation errors.
  - Compiler catches contract violations at compile time where possible.
  - Contracts compose: caller's `ensures` must satisfy callee's `requires`.
  - Higher-order contracts (contracts on function arguments) deferred to v0.2.
- **Negative:**
  - Adds syntactic surface area to function declarations.
  - Contract purity enforcement adds compiler complexity.
  - Runtime contract checking has a cost in debug builds.

---

### Compliance

- Every function with a contract must have its `requires`/`ensures` clauses checked by the compiler.
- Contract expressions must pass purity analysis.
- Contract inheritance must follow Liskov substitution.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Library-based contracts | Requires syntactic integration — contracts need to be part of the function signature, not callable wrappers. |
| Documentation-only | No verification — contracts drift from implementation with no detection mechanism. |
| Type-level encoding (phantom types) | Cannot express relational properties between inputs and outputs (e.g., `result * result ≈ x`). |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes intent; compiler verifies it. Directly addresses LLM generation accuracy. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Contracts follow consistent Liskov inheritance rules. Purity enforcement is well-scoped. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Three keywords (`requires`, `ensures`, `invariant`) cover the essential contract types. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Contracts are part of the function signature — they compose with the existing declaration syntax. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Contract semantics are independent of any specific checker implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Contracts document intent at the declaration site, improving long-term maintainability. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Contracts reduce LLM error rates by providing semantic intent alongside type structure. |

**Gates not applied:** None — all seven gates are applicable to architectural decisions.
