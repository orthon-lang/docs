# EDR-034: Allocation as Policy — Core Allocation Model

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Memory allocation is a fundamental implementation concern that affects performance predictability, memory safety, and overall program design. The core language must define *what* allocation means semantically (data is created in memory), but must not prescribe *how* that allocation is realized — the mechanism belongs to the Implementation Strategy.

The research document [`ALLOCATION.md`](../../concepts/research/essential/ALLOCATION.md) establishes that allocation semantics are core, but the *mechanism* — heap, arena, GC, linear, static — is an Implementation Policy choice. This EDR formalises that separation and records the Allocation Policy values.

**Per D-04:** Allocation is classified as **Policy**, not Language. The programmer declares data structures; the Implementation Strategy decides allocation. Allocation does not add new semantics expressible via primitives — it is an implementation choice about HOW primitives (`literal`, `pack`, `identifier`, `reference`) are materialised in memory.

---

### Decision

Allocation is a **Policy-level concern**, not a Language construct. The Allocation Policy defines *how* data is allocated, rather than adding new language semantics.

The Allocation Policy defines five mutually exclusive values:

| Value | Description | Use Case |
|-------|-------------|----------|
| `Heap` | General-purpose heap allocator (malloc/free) | General-purpose when no single constraint dominates |
| `Arena` | Region-based allocation, bulk deallocation | Default — predictable performance |
| `Linear` | Sequential bump allocation, no deallocation | Tight loops, short-lived temporary data |
| `GC` | Tracing garbage collection | High-productivity, throughput-oriented targets |
| `Static` | Compile-time fixed allocation, no runtime allocator | Embedded, real-time, safety-critical |

The Allocation Policy is part of the Implementation Strategy system, not the Core Language. Each Implementation Strategy selects one Allocation Policy value; the Language specification is independent of which value is chosen.

---

### Why Policy, Not Language

Per D-04's Policy classification rules:

1. **No new semantics expressible via primitives.** Allocation is the realisation of existing primitives (`pack` for composite data, `literal` for inline values, `reference` for indirection) — it does not add new semantic operations.
2. **Language semantics are independent.** The same program is valid under Heap, Arena, GC, Linear, or Static allocation. Core features (equality, pattern matching, traits, error handling) are unaffected by which Allocation Policy is active.
3. **Implementation Strategy concern.** Allocation is a *how* decision, not a *what* decision. The programmer declares data; the strategy decides materialization.

---

### Consequences

- **Positive:**
  - Allocation mechanism is fully decoupled from language semantics — programs are portable across allocation strategies.
  - Programmer never writes `malloc`/`new` — allocation is implicit from data declarations.
  - Five well-defined policy values cover the full spectrum from embedded (Static) to high-productivity (GC).
  - Default (Arena) provides predictable performance without GC pauses.
- **Negative:**
  - Allocation hint syntax (e.g., `@arena`, `@heap`) is deferred — no way to hint allocation at call sites in v0.1.
  - Performance-critical code may need strategy profiles, not just one policy.
  - Interaction with Lifetime Policy (scope-based vs. RC vs. GC) adds complexity.

---

### Compliance

Every concept document that declares an Allocation Policy footprint must reference this EDR. Implementation Strategies must select exactly one Allocation Policy value. The Core Language specification must not assume any specific allocation mechanism.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Allocation as Language construct (`new` keyword, placement new) | Violates Minimal Core — allocation mechanism is an implementation detail, not language syntax. Programmer should declare data, not allocation sites. |
| GC as default | Unpredictable pauses violate Predictability principle. Arena provides better baseline. |
| No allocation policy — single mechanism | Violates Implementation Independence — a language targeting embedded to cloud must support multiple allocation strategies. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Policy concept — user value is indirect through strategy selection. Not a direct language feature. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Policy classification is consistent with D-04: no new semantics, independent of language core. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Five values cover known allocation strategies. Each is well-understood. No overlap. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Fits cleanly in Strategy system — one policy type with orthogonal values. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Allocation is fully independent of any specific implementation. Core semantics work under all values. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding new allocation strategies (e.g., NUMA-aware) simply adds a new policy value. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Policy concept — LLM never writes allocation code. Strategy selection is a build-time concern. |

**Gates not applied:** USER_VALUE_GATE (policy, not user-facing feature); LLM_GENERABILITY_GATE (LLM never writes allocation strategy code).

**Policy classification note:** This EDR applies reduced validation per D-04's Policy pipeline — USER_VALUE_GATE is skipped because Policy concepts provide indirect user value through strategy selection, not direct language semantics.
