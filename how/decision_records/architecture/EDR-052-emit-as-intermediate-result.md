# EDR-052: `emit` as Intermediate Result — Partial Results During Long-Running Computation

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's lazy sequence model (EDR-021) defines the `emit` keyword for producing lazy sequences — values emitted one at a time on demand. The `emit` semantic is: "produce a value and suspend execution until the consumer requests the next value."

The research document at `how/concepts/research/important/EMIT_AS_INTERMEDIATE_RESULT.md` proposes a **broader semantic for `emit`**: intermediate results during a long-running computation that has both incremental outputs and a final result. The key insight:

```orthon
fun parse(file: File) -> Statistics
    for line in file:
        emit parseLine(line)     # intermediate: publish as you go
    return statistics             # final: summary after all work
```

The question: **does this add new semantics beyond LAZY_SEQUENCE_GENERATORS (EDR-021)?**

Analysis: EDR-021 defines `emit` as the production side of the pull-based iterator protocol — values are produced lazily on demand via `next()`. The "intermediate result" model (function publishes partial results and returns a final value) is semantically equivalent: a generator that produces intermediate values via `emit` and terminates with a final value via `return`.

However, the conceptual framing is important: a function with `emit` returns `Iterator[T]` per EDR-021. The "intermediate result" framing emphasises that:
1. A function may have both intermediate (`emit`) and final (`return`) results.
2. The function is not "a generator" — it's "a computation with incremental output."
3. The caller can consume intermediate results while the computation continues (if the computation drives itself rather than waiting for `next()`).

This is a **semantic refinement** of EDR-021, not a new language construct. The `emit` keyword already supports this model technically. The refinement is in the specification language and conceptual model.

The Decision Pipeline classified EMIT_AS_INTERMEDIATE_RESULT as **Language** (semantic refinement of EDR-021): The `emit` keyword already exists. This EDR clarifies that `emit` serves both lazy sequence production AND intermediate result publication. The specification documents both uses.

---

### Decision

Refine the `emit` semantics to explicitly recognise two use cases:

1. **Lazy sequence production** (EDR-021) — Values produced on demand via iterator protocol. Consumer-driven; producer waits for `next()`.

2. **Intermediate result publication** — Values published as they become available during a computation. The computation has a final `return` value in addition to intermediate `emit` values. The computation drives itself rather than waiting for consumer requests.

```orthon
# Intermediate result model
fun process_dataset(data: Dataset) -> Summary
    for batch in data.batches():
        let result = analyse(batch)
        emit result                     # publish incremental result
    return compute_summary(data)       # final summary

# Consumer can consume both intermediate and final results
let results = process_dataset(my_data)
for r in results:                       # consumes intermediate emits
    display(r)
let summary = results.final()           # gets the final return value
```

**Key semantic specification:**

A function that uses `emit` returns `Iterator[T]` where `T` is the intermediate value type. The function's final `return` value is accessible via a `.final()` method on the iterator. If the function does not use `emit`, the return value is the normal function result (no iterator wrapping).

```orthon
# Equivalent formulations:
fun parse(file) -> Iterator[ParsedLine]
    for line in file:
        emit parseLine(line)
    return statistics     # final value accessible via .final()

# Same semantics, alternative form using emit + return:
fun parse(file) -> Iterator[ParsedLine]
    for line in file:
        emit parseLine(line)
    return statistics
```

**This is a specification refinement** of EDR-021, not a new construct. EDR-021's `emit` already supports this pattern technically — this EDR documents the dual use explicitly and adds the `.final()` accessor.

---

### Consequences

**Positive:**
- Unifies lazy sequences and incremental computation under a single `emit` mechanism — no separate concept needed.
- The `.final()` accessor gives access to the computation's final result, which generators in other languages lack (Python's `StopIteration.value` is ad-hoc).
- Conceptual model is intuitive: "emit intermediate results, return the final answer."
- Zero new syntax — `emit` and `return` already exist.
- The model supports both pull-based (consumer drives) and push-based (computation drives) patterns within the same `emit` mechanism.

**Negative:**
- The distinction between "lazy sequence" and "intermediate result" is conceptual — the compiler treats both the same way.
- `.final()` requires storing the return value in the state machine (already part of EDR-021's state machine).
- Functions that emit intermediate results must return `Iterator[T]` even if the consumer only cares about the final result.

---

### Compliance

1. Any function using `emit` must return `Iterator[T]` (per EDR-021).
2. The final `return` value of an emitting function must be accessible via `.final()` on the iterator.
3. A function using `emit` with a final `return` must store the return value in the state machine for later retrieval.
4. The intermediate result model does not introduce new syntax or keywords — only specification language.
5. Both use cases (lazy sequence, intermediate result) must have identical semantics at the compiler level.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Separate `stream` construct for intermediate results | Would add a second mechanism alongside `emit`. The `emit` keyword already supports both patterns. A second construct would violate Minimal Core. |
| Callback-based intermediate results | Would invert control and require the consumer to register callbacks. `emit` provides a cleaner pull-based model with automatic backpressure. |
| No explicit intermediate result model | Would leave the dual use of `emit` undocumented. Explicitly specifying both use cases improves specification clarity and programmer understanding. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (semantic refinement): `LOGICAL_CONSISTENCY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All terms are precisely defined and consistent with EDR-021. The intermediate result model is a conceptual refinement, not a semantic change. No self-referential paradoxes. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | The refinement operates entirely within EDR-021's existing model. No layer violations. `emit` remains a Core Language construct. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Adding the `.final()` accessor and documentation does not increase language complexity. The model is already present in EDR-021 — this EDR makes it explicit. |
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to publish results incrementally during a long computation and then return a final answer." This is a natural pattern in data processing, parsing, and streaming ETL. |

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — The refinement does not change implementation requirements. `LONG_TERM_MAINTAINABILITY_GATE` — The `.final()` accessor is a minor extension of the existing state machine pattern. `LLM_GENERABILITY_GATE` — The pattern is intuitive; LLMs familiar with generators will understand `emit` + `return` + `.final()`.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
