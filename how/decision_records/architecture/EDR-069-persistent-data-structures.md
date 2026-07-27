# EDR-069: Persistent Data Structures — Structural Sharing Collections

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Module

---

### Context

Orthon's collection model has mutable `List[T]` and conditionally immutable `Tuple`. This leaves a gap for guaranteed-immutable collections needed for hash keys, concurrent access, caching, and versioning. Persistent data structures with structural sharing provide immutable collections where "modification" returns a new version sharing internal nodes with the old.

---

### Decision

Persistent data structures are a **StdLib** concept:
- The StdLib provides persistent collection types: `PersistentList[T]`, `PersistentMap[K, V]`, `PersistentSet[T]`.
- The core defines an `Immutable` marker trait that persistent types implement.
- The compiler can use the `Immutable` marker for optimisations (hash-key usage, copy elision).
- Conversion functions between mutable and persistent variants.
- Structural sharing is an implementation detail — the interface presents value semantics.

---

### Consequences

- **Positive:**
  - Hash-key safe — persistent collections can be used in maps and sets.
  - Thread-safe by construction — no mutation means no data races.
  - Cheap snapshots — structural sharing makes versioning practical.
- **Negative:**
  - Two collection families (mutable and persistent) instead of one.
  - Higher constant factors than mutable arrays.
  - Not compatible with in-place algorithms (sorting, shuffling).

---

### Compliance

- The `Immutable` marker trait must be a zero-method trait (pure type-level guarantee).
- All persistent collection types must implement `Immutable`.
- Conversion functions must be provided.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| No persistent collections | `Tuple` + CoW insufficient for hash-key and concurrent use cases. |
| Persistent only (Clojure) | In-place algorithms impossible without mutable collections. |
| `freeze()` operation | Weaker guarantee — frozen view invalidated by mutation through another reference. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Safe concurrent access without locks — direct programmer need. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Value semantics for collections — consistent with rest of the model. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One `Immutable` marker + library types covers the gap. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | `Immutable` marker uses existing trait system (EDR-019). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Structural sharing is an implementation detail — interface is value semantics. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Well-understood from Clojure, Scala, and functional programming. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | LLMs correctly use immutable collections for thread-safe and key-safe scenarios. |
