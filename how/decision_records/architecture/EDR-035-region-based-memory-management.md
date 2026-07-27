# EDR-035: Region-Based Memory Management as Allocation Sub-Policy

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Memory Management)

---

### Context

Region-based memory management (arenas) is a specific allocation strategy where memory is allocated in linear buffers (regions) and freed in bulk when the region scope exits. The research document [`REGION_BASED_MEMORY_MANAGEMENT.md`](../../concepts/research/essential/REGION_BASED_MEMORY_MANAGEMENT.md) explores this model in depth, covering arena structure, data types, indirection via arena indices, mutation, closures, module arenas, concurrency, and lifetime analysis.

**Per D-04:** Region-based memory management is classified as **Policy** — it is a sub-policy within the Allocation Policy (EDR-034), not a Language construct. It does not add new language semantics; it is a specific implementation choice about HOW allocation is realised within the Arena allocation strategy.

**Relationship to EDR-034:** EDR-034 defines the Allocation Policy with `Arena` as one of five values. This EDR defines the sub-policy values within the `Arena` allocation strategy — how arenas are managed, scoped, and inferred.

---

### Decision

Region-Based Memory Management is a **sub-policy within the Allocation Policy**, not a Language construct. It defines three policy values for how arena lifetimes are determined:

| Value | Description | Use Case |
|-------|-------------|----------|
| `ScopeRegion` | Arena lifetime inferred from lexical scope — arena created at scope entry, freed at scope exit. Compiler inserts arena operations automatically. | Default for most code — no programmer annotation needed. |
| `ExplicitRegion` | Programmer explicitly declares arena lifetimes via annotations (`@arena`). Compiler validates. | Performance-critical code where scope inference is insufficient. |
| `NoRegion` | No region-based allocation — falls back to another Allocation Policy value (Heap, GC, Static). | Targets that do not support arena allocation. |

The sub-policy is only meaningful when the Allocation Policy (EDR-034) is set to `Arena`. When Allocation Policy is `Heap`, `GC`, `Static`, or `Linear`, the Region-Based sub-policy is implicitly `NoRegion`.

---

### Why Policy, Not Language

1. **No new language semantics.** Region-based allocation is an implementation technique — the programmer declares data structures, and the arena mechanism is an invisible optimization. The semantic model (ownership, lifetime, mutation) is unaffected.
2. **Language semantics are independent.** Core concepts (equality, pattern matching, traits, error handling) behave identically regardless of arena strategy. The same program compiles under ScopeRegion, ExplicitRegion, or NoRegion — only performance changes.
3. **Sub-policy of Allocation.** This is a refinement of EDR-034's `Arena` value, not a separate language concern. The Policy system naturally accommodates sub-policies within a policy value.

---

### Consequences

- **Positive:**
  - Arena allocation is fully invisible to the programmer by default (ScopeRegion).
  - Explicit arena annotations available for performance tuning when needed.
  - Full inheritance of Allocation Policy's implementation independence.
  - Region inference without lifetime annotations avoids Rust-style annotation burden.
- **Negative:**
  - Sub-policy adds complexity to the Strategy system (nested policy values).
  - ScopeRegion inference may produce suboptimal arena boundaries in complex control flow.
  - Module arena concept (long-lived module-level arenas) requires additional specification.
  - Interaction with ownership transfer across arena boundaries (deep copy on move) has performance implications.

---

### Compliance

The DEFAULT_STRATEGY.md specifies `Allocation = Arena` with `Region = ScopeRegion`. Implementation Strategies must document their Region sub-policy value when Allocation Policy is `Arena`. The Core Language must not assume arena-based allocation.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Region inference as Language feature (explicit lifetime annotations) | Violates Minimal Core — arena lifetimes are implementation detail. Default ScopeRegion handles 90% of cases. |
| Full automatic region inference (ML Kit-style) | Research-grade complexity without proportional benefit. Lexical scope inference provides sufficient coverage. |
| No region sub-policy — single arena model | Too restrictive — different code patterns benefit from different arena scoping (function scope vs. manual region vs. none). |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Sub-policy — indirect value through allocation performance. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Sub-policy refinement is consistent with Policy architecture. Arena allocation naturally decomposes into lifetime-scoping strategies. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Three values cover the natural spectrum: automatic, explicit, disabled. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Sub-policy pattern extends the Strategy system without adding new architectural concepts. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Region semantics (scope-bound lifetimes, bulk deallocation) are implementation-independent. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding inference strategies is a new sub-policy value, not a system change. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Policy concept — LLM never writes arena management code. |

**Gates not applied:** USER_VALUE_GATE (sub-policy); LLM_GENERABILITY_GATE (policy concept).
