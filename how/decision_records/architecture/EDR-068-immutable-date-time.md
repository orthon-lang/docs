# EDR-068: Immutable Date/Time — Value-Semantics Date/Time Types

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Legacy date/time APIs are mutable, thread-unsafe, and have surprising behaviour. Orthon should eliminate date/time-related bugs by providing immutable, thread-safe date/time types. The date/time domain is well-understood and purely a library design problem.

---

### Decision

Immutable Date/Time is a **StdLib** concept:
- The StdLib provides immutable, thread-safe date/time types: `Instant`, `Date`, `DateTime`, `ZonedDateTime`.
- All types are value types with value semantics.
- Operations return new values — no mutation of existing instances.
- Parsing returns `Result<T>` — no exceptions for parse failures.
- Formatters are immutable objects.

---

### Consequences

- **Positive:**
  - Eliminates thread-safety bugs in date/time code.
  - Immutability makes reasoning about date arithmetic straightforward.
  - Result-based parsing integrates with the error handling model.
- **Negative:**
  - Date/time types are not part of the language — they are library types.
  - Calendar system support beyond Gregorian may be deferred.

---

### Compliance

- All date/time types must be immutable.
- Parsing must return `Result<T>` per EDR-020.
- Formatters must be immutable.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Mutable date/time (Java legacy) | Thread-unsafe, surprising mutation — the problem we're solving. |
| Single God object | Constructor overload explosion — violates Single Responsibility. |
| Third-party library only | No standardisation — inconsistent API across ecosystem. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Predictable date arithmetic — no surprises. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Value semantics for time — `yesterday = today - 1`. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One set of immutable types covers all date/time needs. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pure library — no language impact. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Date/time implementation independent of any runtime choice. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood domain — proven from Java `java.time` and Noda Time. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs correctly generate immutable date/time operations. |
