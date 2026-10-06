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
fun process_dataset(data: Dataset) -> Iterator<BatchResult>
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

A function that uses `emit` returns `Iterator<T>` where `T` is the intermediate value type. The function's final `return` value is accessible via `.final()` on the iterator:

```orthon
fun parse(file: File) -> Iterator<ParsedLine>
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
fun parse(file) -> Iterator<ParsedLine>
    for line in file:
        emit parseLine(line)
    return statistics

# Form 2: emit only, return via .final()
fun parse(file) -> Iterator<ParsedLine>
    for line in file:
        emit parseLine(line)
    emit statistics           # final value as last emit
```

### Nested `emit` Function — Splitting Stream from Terminal Result

> **Disputed — open, not a settled recommendation.** The dual-channel
> pattern shown above (`emit` plus a differently-typed `return` in the same
> function, retrieved via `.final()`) was flagged in a concept-review
> discussion: readers expect `let x = f()` to bind `f`'s `return` value, not
> an `Iterator<T>`, and `.final()` hides a second result channel behind an
> accessor that does not read as returning *the* result. The review
> converged on the pattern below — splitting the stream and the terminal
> result across two functions — as the preferred shape where the choice is
> available. The dual-channel form above remains the ACCEPTED EDR-052
> specification; this subsection documents an additional model alongside
> it, not a replacement. See Open Questions below.

```orthon
type Progress(step: Int, current: Int)

fun show_progress(info: Progress)
    log("Step {info.step}: current result = {info.current}")

fun process_data(batches: Array<Array<Int>>) -> Int
    let mut result = 0
    let mut step = 0
    let mut spikes = 0

    fun track_batch(batch: Array<Int>) -> Iterator<Progress>
        for value in batch:
            result += value
            step += 1
            if value > 25:
                spikes += 1
            emit Progress(step: step, current: result)

    for batch in batches:
        for info in track_batch(batch):
            show_progress(info)

    log("spikes: {spikes}")
    return result

result = process_data([[10, 20], [30], [40, 5, 60]])
print("Total:", result)
```

`track_batch` is nested inside `process_data` specifically because each call must update three
closed-over mutable locals of the outer function at once — `result`, `step`, and `spikes`. A
free-standing top-level function doing the same per-batch work would have to return a composite
value and have every call site in the `for batch in batches:` loop manually unpack that composite
and fold it back into three separate accumulators on every iteration; nesting lets it write
directly into the outer bindings through closure capture (per `PRIMITIVE_BLOCKS.md`'s
mutable-capture rule — `mut` on the captured binding, here `result`/`step`/`spikes`, is what makes
this legal), eliminating that manual fold-back entirely.

`track_batch` uses `emit` rather than `return` because one call can genuinely produce a variable
number of `Progress` values — one per element of `batch`, so 0..N depending on the batch's
length. The general rule: use `emit` only when a single invocation can produce a variable count
of items (including zero); if a function always emits exactly once per call, plain `return` is
correct instead — wrapping one guaranteed value in an `Iterator<T>` only to unwrap it immediately
is unjustified overhead.

This example deliberately does not use `.final()` or combine `emit` with a differently-typed
`return` in the same function. The terminal result (`result`) is owned entirely by the outer
`fun process_data`, returned with an ordinary `return`; the intermediate stream is owned entirely
by the nested `fun track_batch`, which returns `Iterator<Progress>`. The two result channels are
two different functions, not one function multiplexing two kinds of output through one return
path.

### Behavioural Specification

1. If a function uses `emit`, it returns `Iterator<T>`. The type parameter `T` is inferred from `emit` expressions.
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
2. How does intermediate-result `emit` interact with a coroutine-on-yield, if that hypothesis ([COROUTINE_ON_YIELD](../../how/concepts/research/deferrable/COROUTINE_ON_YIELD.md)) is ever accepted?
3. Should there be a way to yield intermediate results without building the full iterator state machine?
4. Is combining `emit` with a differently-typed `return` in the same function (the `.final()`
   pattern, see Model above) sound, or should it be replaced by the split pattern — an outer
   terminal-owning `fun` plus a nested stream-owning `emit` `fun` (see 'Nested `emit` Function —
   Splitting Stream from Terminal Result' above)? Raised but not resolved by the concept-review
   discussion that surfaced it.

## Decision History

- **2026-07-27** — Accepted via EDR-052. Classification: Language (semantic refinement of EDR-021). Clarifies that `emit` serves both lazy sequence production and intermediate-result publication. Adds `.final()` accessor specification.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/LAZY_SEQUENCE_GENERATORS.md` (referenced)
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/SYNTAX.md`
