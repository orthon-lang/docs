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

## Contracts (proposed by EDR-098)

> Proposed by [EDR-098](../decision_records/architecture/EDR-098-trait-contract-unification.md); not accepted. It adds no new default: the rule of [`CONTRACTS.md`](../../what/concepts/CONTRACTS.md) applies unchanged.

The strategy does not change when contracts are checked: proof at compile time where possible, runtime assertions in debug builds, elision in release builds unless `--enforce-contracts` is passed. It changes the **diagnostic payload** when a check runs: the minimal payload is a diagnostic code (which identifies the kind of violation and the guilty side) and the source location. The clause text and the values of the names in it are not formatted or stored, because formatting and allocation are not affordable by default on this strategy.

## See Also

- [`IMPLEMENTATION_STRATEGIES.md`](./IMPLEMENTATION_STRATEGIES.md)
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`DEFAULT_STRATEGY.md`](./DEFAULT_STRATEGY.md)
- [`HIGH_PERFORMANCE_STRATEGY.md`](./HIGH_PERFORMANCE_STRATEGY.md)
