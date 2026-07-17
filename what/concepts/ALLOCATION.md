# Allocation

## Issue (Why)

What problem does this concept solve?

## Principles

Which principles must not be violated?

## Model (What)

What does the language concept look like?

## Default Strategy

By default, allocation follows the **Allocation Policy** specified in
the active Implementation Strategy. For the Default Strategy, this is
`Arena` — region-based allocation with bulk deallocation.

See [`DEFAULT_STRATEGY.md`](../../how/strategies/DEFAULT_STRATEGY.md)
and [`IMPLEMENTATION_POLICIES.md`](../../how/IMPLEMENTATION_POLICIES.md)
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

See [`IMPLEMENTATION_POLICIES.md`](../../how/IMPLEMENTATION_POLICIES.md)
§ Allocation Policy for the full catalogue.

## Open Questions

What remains unresolved?

## Decision History

Why were alternatives rejected?
