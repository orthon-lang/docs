# Embedded Strategy

> Minimal resource footprint, deterministic behaviour.
> Designed for resource-constrained targets such as microcontrollers,
> firmware, and real-time systems.

## Policy Profile

| Policy | Value | Status |
|--------|-------|--------|
| **Allocation Policy** | `Static` | Active |
| **Algorithm Policy** | `MemoryOptimized` | *Pending concept design* |
| **Evaluation Policy** | `Eager` | *Pending concept design* |
| **Lifetime Policy** | `Manual` | *Pending concept design* |
| **Concurrency Policy** | `None` | *Pending concept design* |

## Description

The Embedded Strategy optimises for minimal and predictable resource
usage, suitable for environments where memory is scarce, deadlines
must be met, or runtime overhead must be minimised.

- **Allocation Policy = Static** — all memory is allocated at compile
  time. No runtime allocator, no heap fragmentation, no allocation
  failures at runtime. Memory usage is fully determined before the
  program starts.
- **Algorithm Policy = MemoryOptimized** — algorithms are selected
  for minimal memory footprint, even at the cost of throughput.
  In-place algorithms are preferred over those requiring auxiliary
  storage.
- **Evaluation Policy = Eager** — all expressions are evaluated
  immediately. No lazy evaluation, no thunks, no deferred computation.
  Control flow and resource usage are fully predictable.
- **Lifetime Policy = Manual** — the programmer explicitly manages
  object lifetimes. No automatic reference counting or garbage
  collection overhead. Suitable for bare-metal and RTOS environments.
- **Concurrency Policy = None** — single-threaded execution. No
  thread pool, no work stealing, no concurrency primitives. Ideal
  for single-core embedded targets.

## See Also

- [`IMPLEMENTATION_STRATEGIES.md`](./IMPLEMENTATION_STRATEGIES.md)
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`DEFAULT_STRATEGY.md`](./DEFAULT_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](./HIGH_PERFORMANCE_STRATEGY.md)
