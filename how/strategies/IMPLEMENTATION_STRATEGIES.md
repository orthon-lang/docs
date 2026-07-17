# Implementation Strategies

> A **Strategy** is a named set of **Policies**. Each Policy is a
> declarative choice within a single domain-specific area.
> New strategies can be added as they emerge without changing the
> architecture document.

## Conceptual Model

```
Language Semantics (what the program means)
        │
        ▼
Implementation Strategy  ← named set of Policies
        │
        ├── Allocation Policy    → Arena
        ├── Algorithm Policy     → Adaptive
        ├── Evaluation Policy    → Lazy Defaults
        ├── Lifetime Policy      → ReferenceCounting
        ├── Concurrency Policy   → ThreadPool
        └── ...
        │
        ▼
Implementation (compiler, runtime, platform)
```

A Strategy is **declarative, not procedural**. It describes *what*
set of Policies to apply, not *how* to choose them:

| ❌ Procedural (avoid) | ✅ Declarative (preferred) |
|---|---|
| `if speed then strategy X else strategy Y` | `Strategy = Default { Allocation: Arena, ... }` |

This keeps Strategies orthogonal — replacing one Policy within a
Strategy does not affect other Policies, and Strategies can be
composed and compared by their Policy values alone.

## Policy Types

Each Policy type corresponds to a separable language concept.
See [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
for the full catalogue.

| Policy Type | Concept | Values |
|---|---|---|
| **Allocation Policy** | `concepts/ALLOCATION.md` | Heap, Arena, Linear, GC, Static |
| **Algorithm Policy** | *Pending* | Adaptive, MemoryOptimized, Parallel |
| **Evaluation Policy** | *Pending* | Lazy Defaults, Eager |
| **Lifetime Policy** | *Pending* | ReferenceCounting, GarbageCollected, Manual, RegionBased |
| **Concurrency Policy** | *Pending* | ThreadPool, WorkStealing, EventLoop, None |

## Scope of the Pattern

The following table catalogues every mechanism where the
interface/implementation separation (defined in
[`ARCHITECTURE.md`](../architecture/ARCHITECTURE.md#scope-of-the-pattern))
applies, and maps each to its Policy type.

| Aspect | Interface (Language / StdLib) | Policy Type | Implementation Values |
|---|---|---|---|
| **Memory allocation** | Declarative allocation semantics | Allocation Policy | Heap, arena, linear, or GC-backed allocator |
| **Expression evaluation** | Expression semantics and composition | Evaluation Policy | AST walker, bytecode VM, JIT, or interpreter |
| **Algorithm selection** | What the program expresses | Algorithm Policy | Sorting, searching, hashing strategy implementations |
| **Object representation** | Data model and structural contracts | *(future)* | Struct-of-fields, SOA, tagged unions, or pointer-based layout |
| **Concurrency model** | Task and communication semantics | Concurrency Policy | Thread pool, work-stealing, event loop, or hardware dispatch |
| **Error handling** | Result and error propagation model | *(future)* | Stack unwinding, error codes, or return-value branching |

> **Note:** Object Representation and Error Handling will be mapped
> to dedicated Policy types when their corresponding language concepts
> are designed. Until then, they remain implementation-level concerns.

## Strategy Profiles

Concrete Strategies assemble Policies into named profiles. Each
profile optimises for a different set of constraints.

- [`DEFAULT_STRATEGY.md`](./DEFAULT_STRATEGY.md) — general-purpose balance
  of performance and resource usage
- [`EMBEDDED_STRATEGY.md`](./EMBEDDED_STRATEGY.md) — minimal resource
  footprint, deterministic behaviour
- [`HIGH_PERFORMANCE_STRATEGY.md`](./HIGH_PERFORMANCE_STRATEGY.md) —
  maximum throughput, parallel hardware

New profiles can be added as implementation needs arise.

## See Also

- [`ARCHITECTURE.md`](../architecture/ARCHITECTURE.md) § Implementation Strategy
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`DECISION_VALIDATION.md`](../gates/DECISION_VALIDATION.md) — `IMPLEMENTATION_INDEPENDENCE_GATE`