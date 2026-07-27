# EDR-060: Smart Cast — Flow-Sensitive Type Narrowing

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

---

### Context

After checking that a value is of a specific type, the programmer should not need to cast it again — the compiler should know. Smart casting tracks type-narrowing information through control flow constructs (`if`, `when`, `&&`, `||`) and automatically narrows the type of a variable within the relevant scope.

This requires the compiler to perform flow-sensitive type analysis — tracking type information across control flow edges. This is a compiler-level analysis not expressible via primitive composition.

---

### Decision

Smart cast is a **Language** construct:
- After `if value is Type`, the compiler narrows `value` to `Type` in the true branch.
- After `value isnt None`, the compiler unwraps `Option[T]` to `T`.
- Each `when` branch narrows the scrutinee's type.
- Smart cast only applies to effectively-immutable variables.
- `value as Type` provides an explicit cast escape hatch.
- Conservative — if the compiler cannot prove safety, it stays at the wider type.

---

### Consequences

- **Positive:**
  - Eliminates redundant explicit casts after type checks.
  - Composes with pattern matching — smart cast handles non-pattern scenarios.
  - Conservative by default — no unsound narrowing.
- **Negative:**
  - Does not apply to mutable variables (even if provably unmodified).
  - Does not propagate through function call boundaries.
  - Smart cast is partially subsumed by PATTERN_MATCHING (EDR-025) — many narrowing scenarios are already covered by pattern matching.

---

### Compliance

- Compiler must perform flow-sensitive type tracking for `if`, `when`, `&&`, `||`, `return`, and `throw`.
- Narrowing must reset on variable reassignment.
- Explicit cast `as Type` must compile to an unchecked type assertion.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| No smart cast | Manual casting after every check is verbose (Java pre-16). |
| Always explicit narrow | Every narrowing requires a cast keyword — ergonomic loss without clarity gain. |
| Aggressive narrowing | Applies to mutable variables — unsafe without escape analysis. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Eliminates redundant casts — direct ergonomic improvement. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Narrowing follows visible control flow — programmer can predict when it applies. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One rule: after type check, type narrows in the checked scope. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Composes with pattern matching (EDR-025) — smart cast handles non-pattern narrowing. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Flow-sensitive analysis is independent of specific type system design. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Kotlin and TypeScript — proven pattern. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs correctly generate type checks expecting narrowing — smart cast makes generated code work. |
