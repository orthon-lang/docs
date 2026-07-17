# High Performance Strategy

> Maximum throughput, parallel hardware utilisation.
> Designed for HPC, data processing, and compute-intensive workloads
> on multi-core and NUMA architectures.

## Policy Profile

| Policy | Value | Status |
|--------|-------|--------|
| **Allocation Policy** | `NUMA-aware` | *Pending concept design* |
| **Algorithm Policy** | `Parallel` | *Pending concept design* |
| **Evaluation Policy** | `Eager` | *Pending concept design* |
| **Lifetime Policy** | `RegionBased` | *Pending concept design* |
| **Concurrency Policy** | `WorkStealing` | *Pending concept design* |

## Description

The High Performance Strategy optimises for raw throughput and
scalability across parallel hardware. It is suitable for batch
processing, scientific computing, and server-side workloads where
maximising resource utilisation is the primary goal.

- **Allocation Policy = NUMA-aware** — memory allocation is
  optimised for Non-Uniform Memory Access architectures. Allocations
  are placed on the NUMA node closest to the accessing thread,
  minimising remote memory access latency.
- **Algorithm Policy = Parallel** — algorithms exploit data
  parallelism and vectorisation. Divide-and-conquer and parallel
  reductions are preferred over sequential implementations when
  input size exceeds a threshold.
- **Evaluation Policy = Eager** — all expressions are evaluated
  immediately. Lazy evaluation is disabled to avoid thunk overhead
  and to enable maximum compiler optimisation.
- **Lifetime Policy = RegionBased** — object lifetimes are managed
  through region-based (arena) allocation with bulk deallocation.
  This provides predictable performance without per-object reference
  counting or garbage collection overhead.
- **Concurrency Policy = WorkStealing** — dynamic work distribution
  where idle threads steal tasks from busy threads' queues. Provides
  optimal load balancing for irregular parallel workloads.

## See Also

- [`IMPLEMENTATION_STRATEGIES.md`](./IMPLEMENTATION_STRATEGIES.md)
- [`IMPLEMENTATION_POLICIES.md`](../IMPLEMENTATION_POLICIES.md)
- [`DEFAULT_STRATEGY.md`](./DEFAULT_STRATEGY.md)
- [`EMBEDDED_STRATEGY.md`](./EMBEDDED_STRATEGY.md)
