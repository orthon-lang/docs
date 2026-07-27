# `emit` as Intermediate Result — Incremental Computation with Final Value

> **✅ ACCEPTED — [EDR-052](../how/decision_records/architecture/EDR-052-emit-as-intermediate-result.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`GENERATORS.md`](GENERATORS.md),
> [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Intermediate Result

---

## Issue (Why)

Traditional functions return a single result. If a computation produces multiple values over time, languages introduce separate constructs (generators, streams, callbacks). But many computations have both intermediate and final results:

- **Data processing pipeline** — Parsing a file produces line-by-line results and a final summary.
- **Long-running computation** — An optimisation algorithm publishes the current best solution incrementally and returns the final optimum.
- **Streaming ETL** — Processing batches of data produces per-batch reports and a final aggregate.

The core problem: **a function should be able to publish intermediate results during execution while maintaining a final return value** — without introducing separate stream or callback abstractions.

## Principles

1. **`emit` serves both models** — Lazy sequence production (EDR-021) and intermediate-result publication share the same `emit` keyword. The difference is conceptual, not mechanical.

2. **Intermediate ≠ lazy** — In the intermediate-result model, the computation drives itself rather than waiting for `next()` calls. Values are produced as they become available.

3. **`return` signals completion** — The final `return` value of an emitting function is accessible separately from the intermediate `emit` values.

4. **Zero new syntax** — The `emit` keyword already exists (EDR-021). This document specifies its use for intermediate results.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Evaluation Policy | Intermediate results are evaluated eagerly (computation-driven), unlike lazy sequences (consumer-driven) |
| Iterator Protocol Policy | `.final()` accessor on the iterator provides the function's return value |
| Result Collection Policy | Governs whether intermediate results are buffered or streamed |

## Model (What)

### Intermediate Result Pattern

A function that uses both `emit` and `return`:

```orthon
fun process_dataset(data: Dataset) -> Iterator[BatchResult]
    for batch in data.batches():
        let result = analyse(batch)
        emit result                     # publish intermediate result
    return compute_summary(data)       # final summary after all work

# Consumer
let results = process_dataset(my_data)
for r in results:                       # consumes intermediate emits
    display(r)
let summary = results.final()           # gets the final return value
```

### Semantic Model

A function that uses `emit` returns `Iterator[T]` where `T` is the intermediate value type. The function's final `return` value is accessible via `.final()` on the iterator:

```orthon
fun parse(file: File) -> Iterator[ParsedLine]
    for line in file:
        emit parseLine(line)
    return total_lines(file)

let it = parse("data.txt")
for parsed in it:
    process(parsed)
let count = it.final()      # total_lines result
```

### Equivalent Forms

```orthon
# Form 1: emit + return (canonical)
fun parse(file) -> Iterator[ParsedLine]
    for line in file:
        emit parseLine(line)
    return statistics

# Form 2: emit only, return via .final()
fun parse(file) -> Iterator[ParsedLine]
    for line in file:
        emit parseLine(line)
    emit statistics           # final value as last emit
```

### Behavioural Specification

1. If a function uses `emit`, it returns `Iterator[T]`. The type parameter `T` is inferred from `emit` expressions.
2. If the function has a final `return value`, the value is stored in the iterator's state and accessible via `.final()`.
3. If the function does NOT use `emit`, the return value is the normal function result (no iterator wrapping).
4. Without a final `return`, the iterator is exhausted when the function body completes (no `.final()` value).

## Default Strategy

The compiler generates a state machine that:
1. Executes the function body, producing values via `emit` (intermediate results are pushed to a buffer or consumed directly)
2. Stores the final `return` value in a per-iterator slot
3. `.final()` returns the stored value

## Alternative Strategies

| Strategy | Description |
|---|---|
| No `.final()` support | Final return value discarded; iterator produces intermediate values only. Simpler state machine. |
| Push-based intermediate results | Intermediate values are pushed to a callback/stream rather than pulled via `next()`. Better for computation-driven models. |
| Eager with intermediate buffer | All intermediate and final values buffered eagerly. Simpler but breaks laziness. |

## Open Questions

1. Should `.final()` be callable only after the iterator is fully consumed, or at any time?
2. How does intermediate-result `emit` interact with bidirectional generators (EDR-050)?
3. Should there be a way to yield intermediate results without building the full iterator state machine?

## Decision History

- **2026-07-27** — Accepted via EDR-052. Classification: Language (semantic refinement of EDR-021). Clarifies that `emit` serves both lazy sequence production and intermediate-result publication. Adds `.final()` accessor specification.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/LAZY_SEQUENCE_GENERATORS.md` (referenced)
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/SYNTAX.md`
