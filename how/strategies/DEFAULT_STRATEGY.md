# Default Strategy

> General-purpose balance of performance and resource usage.
> Suitable for most applications with no specialised requirements.

## Policy Profile

| Policy | Value | Status |
|--------|-------|--------|
| **Allocation Policy** | `Arena` | Active ([EDR-034](../decision_records/architecture/EDR-034-allocation.md)) |
| **Region-Based Memory (Allocation Sub-Policy)** | `ScopeRegion` | Active ([EDR-035](../decision_records/architecture/EDR-035-region-based-memory-management.md)) |
| **Execution Model Policy** | `AOT` | Active ([EDR-036](../decision_records/architecture/EDR-036-execution-program.md)) |
| **Algorithm Policy** | `Adaptive` | *Pending concept design* |
| **Evaluation Policy** | `Lazy Defaults` | *Pending concept design* |
| **Lifetime Policy** | `ReferenceCounting` | *Pending concept design* |
| **Concurrency Policy** | `ThreadPool` | *Pending concept design* |

## Description

The Default Strategy is the baseline for all Orthon implementations.
It targets general-purpose workloads where no single constraint
(memory, throughput, latency, determinism) dominates.

- **Allocation Policy = Arena** — region-based allocation with bulk
  deallocation provides good average-case performance without the
  overhead of a full garbage collector. Supports both short-lived
  and long-lived allocations through multiple arenas.
- **Region-Based Memory = ScopeRegion** — arena lifetimes inferred
  from lexical scope. The compiler automatically inserts arena
  creation at scope entry and bulk deallocation at scope exit.
  No programmer annotation required for the common case.
- **Execution Model Policy = AOT** — ahead-of-time compilation to
  native code. Programs are compiled once and deployed as native
  binaries. This is the default execution strategy for production
  deployments.
- **Algorithm Policy = Adaptive** — algorithm selection adapts to
  input characteristics (size, order, distribution) at runtime or
  compile time, choosing the best available implementation.
- **Evaluation Policy = Lazy Defaults** — function arguments and
  certain expressions are evaluated lazily by default, deferring
  computation until the result is actually needed.
- **Lifetime Policy = ReferenceCounting** — automatic memory
  management through reference counting, providing predictable
  deallocation without stop-the-world pauses.
- **Concurrency Policy = ThreadPool** — fixed pool of worker threads
  for parallel task execution, balancing throughput with predictable
  resource usage.

## See Also

- [`IMPLEMENTATION_STRATEGIES.md`](./IMPLEMENTATION_STRATEGIES.md)
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`EMBEDDED_STRATEGY.md`](./EMBEDDED_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](./HIGH_PERFORMANCE_STRATEGY.md)
