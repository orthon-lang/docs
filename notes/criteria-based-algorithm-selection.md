# Criteria-based Algorithm Selection

**Status:** *Unvalidated idea — needs feasibility validation.*

**Date:** 2026-07-17

## Idea

Don't choose an algorithm, don't choose an algorithm category (`Adaptive`,
`MemoryOptimized`, `Parallel`), but choose an **optimization criterion**.
The compiler (or runtime) selects the algorithm itself, based on:

- optimization criteria (speed, memory, predictability)
- platform (CPU, GPU, WebAssembly, bare metal)
- data context (e.g., data already on GPU → GPU sort)

## Two Levels

### Semantic (part of the language)

```orthon
sort(data)              // guarantees: sortedness, stability
```

This is already an implementation hint. It may be absent on some
platforms.

### Policy (part of Strategy)

```orthon
// ❌ Still names a strategy class
Algorithm Policy = Adaptive

// ✅ Declares optimization intent
//    "minimize memory on this hot path"
//    "maximize throughput, data is on GPU"
```

## Example: GPU sort

If the data being sorted is already on the GPU, the compiler automatically
selects a GPU-optimised sorting algorithm. The programmer does not
write `GpuSort` — they specify the criterion (or nothing, using the
default strategy).

## Connection to Orthon's philosophy

This is a consistent application of the principle "the programmer describes WHAT,
the implementation chooses HOW" — one level deeper than the current Policy model,
where values still name strategy categories rather than pure criteria.

Elsewhere in the language, this principle is already applied. Sorting becomes
another example.

## Open questions

- How to formulate optimisation criteria declaratively?
- How to resolve conflicts between criteria (speed vs memory)?
- Can the programmer still "pin" an algorithm when needed?
- How does this affect performance predictability?
- Does this idea apply only to Algorithm Policy, or more broadly?

## See also

- `docs/how/IMPLEMENTATION_POLICIES.md` § Algorithm Policy
- `docs/how/architecture/ARCHITECTURE.md` § Implementation Strategy
- `docs/how/strategies/IMPLEMENTATION_STRATEGIES.md`
