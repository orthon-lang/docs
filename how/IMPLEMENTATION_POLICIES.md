# Implementation Policies

> A **Policy** is a declarative constraint or preference within a
> single domain-specific area of implementation. Each Policy makes
> decisions independently within its area of responsibility. Policies
> are not part of the language — they are implementation choices.

## Relationship to Strategy

A **Strategy** is a named set of Policies. Policies are the building
blocks; a Strategy assembles them into a coherent profile.

```
Language Specification
        │
        ▼
Implementation Strategy  ← named set of Policies
        │
        ├── Allocation Policy
        ├── Algorithm Policy
        ├── Evaluation Policy
        ├── Lifetime Policy
        ├── Concurrency Policy
        └── ...
        │
        ▼
Implementation
```

## The Declarative Principle

**A Policy is declarative, not procedural.**

A Policy states *what* is preferred, not *how* to achieve it.

| ❌ Procedural (avoid) | ✅ Declarative (preferred) |
|---|---|
| `if speed then TimSort else MergeSort` | `Algorithm Policy = Adaptive` |
| `if low_memory then malloc else mmap` | `Allocation Policy = Arena` |

This keeps Policies orthogonal, composable, and independent of the
concrete implementation beneath them.

## Policy Catalogue

Each Policy type corresponds to a separable language concept. A Policy
type is added when its corresponding concept is designed (see
[`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) —
`IMPLEMENTATION_INDEPENDENCE_GATE`).

### Allocation Policy

**Concept:** [`concepts/ALLOCATION.md`](../what/concepts/ALLOCATION.md)

Controls memory acquisition and deallocation strategy.

| Value | Description |
|-------|-------------|
| `Heap` | General-purpose heap allocator (malloc/free) |
| `Arena` | Region-based allocation, bulk deallocation |
| `Linear` | Sequential bump allocation, no deallocation |
| `GC` | Tracing garbage collection |
| `Static` | Compile-time fixed allocation, no runtime allocator |

### Algorithm Policy

**Status:** *Pending concept design.*

Controls the choice of algorithm implementations (sorting, searching,
hashing, etc.) where multiple correct implementations exist.

| Possible Values | Description |
|-----------------|-------------|
| `Adaptive` | Selects algorithm based on input characteristics |
| `MemoryOptimized` | Prefers algorithms with minimal memory footprint |
| `Parallel` | Prefers algorithms that exploit parallelism |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Evaluation Policy

**Status:** *Pending concept design.*

Controls when and how expressions are evaluated.

| Possible Values | Description |
|-----------------|-------------|
| `Lazy Defaults` | Default arguments and some expressions evaluated lazily |
| `Eager` | All expressions evaluated immediately |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Lifetime Policy

**Status:** *Pending concept design.*

Controls how object lifetimes are managed and when resources are
released.

| Value | Description |
|-------|-------------|
| `ReferenceCounting` | Reference-counted automatic memory management |
| `GarbageCollected` | Tracing garbage collection, deferred reclamation |
| `Manual` | Explicit allocation and deallocation by the programmer |
| `RegionBased` | Region- or arena-based bulk lifetime management |

*Specific values will be formalised when the corresponding language
concept is designed.*

### Concurrency Policy

**Status:** *Pending concept design.*

Controls how concurrent tasks are scheduled and executed.

| Possible Values | Description |
|-----------------|-------------|
| `ThreadPool` | Fixed pool of worker threads |
| `WorkStealing` | Dynamic work distribution across threads |
| `EventLoop` | Single-threaded event-driven concurrency |
| `None` | Single-threaded execution, no concurrency |

*Specific values will be formalised when the corresponding language
concept is designed.*

## Extending the Catalogue

New Policy types are added as new language concepts are designed. Each
new Policy must:

1. Correspond to a separable language concept (documented in
   `docs/what/concepts/`).
2. Pass the [`IMPLEMENTATION_INDEPENDENCE_GATE`](gates/DECISION_VALIDATION.md#gate-catalogue):
   the concept must be definable without reference to the Policy.
3. Define declarative values — not procedures.
4. Be added to this catalogue and cross-referenced from the
   concept document.

## See Also

- [`ARCHITECTURE.md`](architecture/ARCHITECTURE.md) § Implementation Strategy
- [`IMPLEMENTATION_STRATEGIES.md`](strategies/IMPLEMENTATION_STRATEGIES.md)
- [`DEFAULT_STRATEGY.md`](strategies/DEFAULT_STRATEGY.md)
- [`EMBEDDED_STRATEGY.md`](strategies/EMBEDDED_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](strategies/HIGH_PERFORMANCE_STRATEGY.md)
