# EDR-067: Declarative Multi-Key Sort — Syntactic Sugar Over Sorting

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Sorting by multiple fields requires manually writing comparators with comparison chains. Orthon should eliminate manual comparator chains by providing a declarative `sorted(by:)` API with key paths.

This is syntactic sugar over the SORTING concept — the `sorted(by:)` method accepts a list of key paths and constructs a lexicographic comparator. No new language semantics.

---

### Decision

Declarative multi-key sort is a **StdLib** API:
- `collection.sorted(by: .field)` for single-key sorting.
- `collection.sorted(by: [.field_a, .field_b])` for multi-key.
- `collection.sorted(by: .field, order: descending)` for direction control.
- All forms construct lexicographic comparators under the hood.
- Builds on the SORTING concept (EDR-066).

---

### Consequences

- **Positive:**
  - Eliminates manual comparator boilerplate.
  - Self-documenting — the sort key is visible in the call.
  - Zero new language semantics — purely a library API.
- **Negative:**
  - Key path syntax (`.field`) requires compiler support for type-safe key paths.

---

### Compliance

- The StdLib must provide `sorted(by:)` with positional and key path arguments.
- All forms must desugar to the SORTING primitive.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Manual comparator chains | Verbose and fragile — the problem this concept solves. |
| Builder pattern | `sort_by(.a).then_by(.b)` — more concepts for same result. |
| Implicit sort by struct field order | Non-explicit — violates Orthon's explicitness principle. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | `sorted(by: .field)` is the natural reading. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Lexicographic comparison of key paths is deterministic. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One API with variadic key paths — no overload explosion. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Pure StdLib — builds on SORTING. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Comparator construction independent of sort algorithm. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Low surface area — one API method. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `sorted(by: .field)` is the most LLM-obvious declarative API shape. |
