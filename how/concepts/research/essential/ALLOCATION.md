# Allocation

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

Memory allocation is a fundamental concern that affects performance
predictability, memory safety, and overall program design. The core
problem: **how and when is memory allocated for program data**, and
how does the language ensure that allocation is safe, predictable, and
independent of the programmer's manual management?

In systems languages, manual allocation (C's `malloc`/`free`) gives
control but causes use-after-free, double-free, and memory leaks —
~70% of critical security vulnerabilities in C programs. Managed
languages (Java, Go) use garbage collection for safety but at the cost
of unpredictable pauses and memory overhead. The language needs an
allocation model where the programmer declares *what* to allocate,
not *how* — the Implementation Strategy decides the mechanism.

## Principles

1. **Minimal Core** — Allocation semantics are defined by the Core
   Language, but the *mechanism* (arena, heap, GC, static) is an
   Implementation Policy. The programmer does not specify allocation
   sites.
2. **Explicitness** — Operations that may allocate (e.g., creating a
   collection, boxing a value) must be syntactically visible. The
   programmer should know when allocation happens.
3. **Strategy independence** — Core language semantics must not depend
   on any specific Allocation Policy value. The same program must be
   valid under arena, heap, GC, or static allocation.
4. **Predictability** — The default allocation strategy must provide
   predictable performance characteristics. No hidden GC pauses in
   the default profile.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Allocation Policy | Defines *how* data is allocated (Arena, Heap, GC, Static, Linear) |
| Lifetime Policy | Determines *when* allocated memory is freed (scope-based, reference-counted, traced) |
| Representation Policy | Affects whether a value is heap-allocated or inline (e.g., boxing of primitives) |

## Model (What)

The programmer declares data structures and the language determines
allocation. The programmer does **not** write `malloc` or `new` — data
declarations implicitly define allocation requirements.

```orthon
# Data declaration — allocation is implicit
point = (x: 10, y: 20)     # Arena-allocated in Default Strategy
data = List(1, 2, 3)        # Allocation follows active Policy
```

## Default Strategy

By default, allocation follows the **Allocation Policy** specified in
the active Implementation Strategy. For the Default Strategy, this is
`Arena` — region-based allocation with bulk deallocation.

See [`DEFAULT_STRATEGY.md`](../../strategies/DEFAULT_STRATEGY.md)
and [`IMPLEMENTATION_POLICIES.md`](../../IMPLEMENTATION_POLICIES.md)
§ Allocation Policy.

## Alternative Strategies

Alternative allocation strategies are selected by changing the
**Allocation Policy** in a different Implementation Strategy profile.

| Profile | Allocation Policy | Description |
|---------|-------------------|-------------|
| `DEFAULT_STRATEGY.md` | `Arena` | Region-based, bulk deallocation |
| `EMBEDDED_STRATEGY.md` | `Static` | Compile-time, no runtime allocator |
| `HIGH_PERFORMANCE_STRATEGY.md` | `NUMA-aware` | Topology-optimised for multi-socket systems |

Possible Policy values for allocation: `Heap`, `Arena`, `Linear`,
`GC`, `Static`. Each represents a different point in the trade-off
space between flexibility, performance, and determinism.

See [`IMPLEMENTATION_POLICIES.md`](../../IMPLEMENTATION_POLICIES.md)
§ Allocation Policy for the full catalogue.

## Open Questions

1. Should Orthon provide a way to annotate allocation hints (e.g.,
   `@arena`, `@heap`) for performance tuning, or should allocation
   be fully delegated to the Implementation Strategy?
2. How does allocation interact with the span/memory-view concept?
   Can a Span point to arena-allocated memory that outlives the arena?
3. Should there be a `no_alloc` mode where any allocation is a compile
   error? (Useful for embedded and real-time contexts.)

## Decision History

- **Allocation as Policy, not syntax:** Adopted. The programmer declares
  data, and allocation is decided by the Implementation Strategy.
  Alternatives considered: explicit `new`/`malloc` syntax (rejected:
  violates Minimal Core — allocation mechanism is an implementation detail),
  C++-style placement new (rejected: couples allocation to type system).
- **Arena as default:** Adopted for predictability. Alternatives considered:
  GC as default (rejected: unpredictable pauses violate Predictability
  principle), heap as default (rejected: fragmentation risk).
- **Deferred to Phase 3:** Allocation hint syntax, `no_alloc` mode
  specification, interaction with Span and lifetime tracking.
