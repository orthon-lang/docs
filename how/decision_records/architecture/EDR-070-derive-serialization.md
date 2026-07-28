# EDR-070: Derive Serialization — `@derive(Serialize)` via Comptime Macro

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Manually converting objects to/from JSON requires recursive code that misses fields, mishandles cyclic references, and breaks type safety. Orthon should eliminate manual serialization code by providing automatic serialization/deserialization for value types through trait derivation.

The `@derive` mechanism (EDR-029) already provides compile-time code generation via comptime macros. Serialization is a natural extension: a `@derive(Serialize)` annotation invokes a registered macro that generates `to_json`/`from_json` implementations.

---

### Decision

Derive Serialization is a **StdLib** concept:
- `@derive(Serialize, Deserialize)` on a struct generates serialization implementations.
- Uses the existing `@derive` macro mechanism (EDR-029) — no new compiler semantics.
- The StdLib provides format backends (JSON, binary) and customization annotations (`@json(rename_all = "snake_case")`).
- Deserialization returns `Result<T, SerializeError>` per EDR-020.

---

### Consequences

- **Positive:**
  - Eliminates manual serialization boilerplate.
  - Reuses the existing macro system — no new compiler infrastructure.
  - Deserialization type validation at parse time.
- **Negative:**
  - Depends on compile-time execution (EDR-031) for macro evaluation.
  - Binary format support may be deferred beyond v0.1.

---

### Compliance

- The StdLib must provide `Serialize`/`Deserialize` traits and derive macros.
- `@derive(Serialize)` must generate correct `to_json`/`from_json` implementations.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Manual serialization | Error-prone — the problem this solves. |
| Runtime reflection | Bypasses compile-time type safety. |
| External code generation | Build-step drift from source — fragile. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | `@derive(Serialize)` — one annotation eliminates all serialization boilerplate. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Serialization follows structural type shape — deterministic mapping. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One annotation — `@derive` — covers all format backends. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Builds on existing `@derive` macro system (EDR-029). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Serialization trait is independent of format backend implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Derive-based serialization is proven in Rust (serde) and Swift (Codable). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `@derive(Serialize)` is the most LLM-obvious annotation — one declarative line eliminates an entire category of generation errors. |
