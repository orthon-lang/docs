# EDR-053: Iteration Loop — `for`/`while` with Protocol-Based Iteration

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's iterator protocol (EDR-022) defines `Iterator[T]` trait and `IntoIterator[T]` trait as the consumption side of the sequence model. The `for` loop desugars to the iterator protocol. The iterator protocol specification (in `what/concepts/ITERATOR_PROTOCOL.md`) describes the desugaring:

```orthon
for item in collection:
    process(item)

# Desugars to:
let mut it = collection@iter()
loop:
    match it@next():
        Some(item) -> process(item)
        None       -> break
```

The research document at `how/concepts/research/important/ITERATION_LOOP.md` proposes formalising the loop constructs (`for` and `while`) as part of Orthon's language specification. The document establishes:

- **One loop construct for iteration** — `for item in sequence` (no C-style `for (;;)`)
- **`while` for condition-based looping** — `while condition` (separate from iteration)
- **Sequence-based iteration** — the loop operates on sequences, not indices
- **Composable with sequence operations** — transformations happen before the loop
- **No hidden allocation** — iteration streams values without buffering

The question: **does the iteration loop add new semantics beyond ITERATOR_PROTOCOL (EDR-022)?**

Analysis: EDR-022 defines the iterator protocol and describes `for` loop desugaring. However, the actual `for` and `while` loop constructs require:
- Compiler-level loop syntax (`for`, `while` keywords)
- Loop desugaring from `for` to iterator protocol calls
- `break` and `continue` semantics
- `while` condition evaluation semantics
- Destructuring in loop variables (`for (a, b) in pairs`)

These are compiler-level syntactic constructs. While the `for` loop desugars to iterator protocol calls, the loop syntax itself requires compiler support. The `while` loop is a separate construct for condition-based looping (not iteration).

The Decision Pipeline classified ITERATION_LOOP as **Language**: `for`/`while` loop constructs require compiler-level syntax and desugaring. `for` desugars to ITERATOR_PROTOCOL (EDR-022). `while` is a separate condition-based construct.

---

### Decision

Adopt the following loop model for Orthon:

1. **`for item in sequence`** — The only iteration loop construct. Operates on any type implementing `IntoIterator[T]`. Desugars to the iterator protocol per EDR-022.

    ```orthon
    for item in items:
        process(item)
    
    for i in 0..len(array):       # index-based via range
        process(array[i])
    ```

2. **No C-style `for (;;)`** — Index-based iteration is expressed via range literals (`0..n`, `0..=n`) which produce iterators.

3. **`while condition`** — Condition-based looping, separate from iteration.

    ```orthon
    while queue.not_empty():
        process(queue.dequeue())
    ```

4. **`break` and `continue`** — Available in both `for` and `while`. `break` exits the loop. `continue` skips to the next iteration. Optional `break value` (Rust-style) for `loop` (infinite loop with result).

5. **Destructuring in loop variables** — `for (a, b) in pairs` decomposes each element.

    ```orthon
    for {name, age} in people:
        print("{name} is {age} years old")
    ```

6. **`loop` keyword for infinite loops** — An explicit `loop { ... }` construct for infinite loops, with optional `break value` for expression-oriented looping.

    ```orthon
    let result = loop:
        let item = queue.receive()
        if is_valid(item) then break item
    ```

7. **No `else` clause on `for`** — Python-style `for...else` is not adopted. Post-loop handling is explicit.

**Key relationship to EDR-022:** The `for` loop desugaring is the language-level bridge between the loop syntax and the iterator protocol. EDR-022 defines the protocol; this EDR defines the syntax and desugaring.

---

### Consequences

**Positive:**
- One iteration construct (`for ... in`) — simple, consistent, LLM-friendly.
- No C-style `for (;;)` — eliminates off-by-one errors and boilerplate index management.
- Range syntax (`0..n`) provides index-based iteration without C-style syntax.
- `while` is separate from iteration — clear semantic distinction (condition vs. sequence).
- `loop` with `break value` supports expression-oriented infinite loops.
- Destructuring in loop variables reduces boilerplate for compound elements.

**Negative:**
- `for (;;)` style cannot be expressed directly — some programmers may miss it (though `loop` + explicit condition covers the same patterns).
- Iteration over non-standard patterns (skip by 2, custom step) requires sequence operations before the loop.
- Range syntax must be defined (StdLib or built-in) — `0..n` is syntactic sugar requiring compiler-level range-to-iterator conversion.

---

### Compliance

1. The `what/concepts/ITERATION_LOOP.md` specification defines the canonical loop semantics.
2. `for ... in` must desugar to `IntoIterator::iter()` + `loop` calling `next()` (per EDR-022).
3. `while` must NOT require an iterable — it operates on a boolean condition.
4. `break` and `continue` must have defined semantics in both `for` and `while`.
5. `loop { ... }` must execute forever unless `break` is called.
6. `break value` must be supported in `loop` constructs (expression-oriented).
7. Range syntax (`0..n`) must produce an `Iterator[Int]` value.
8. Destructuring in loop variables must follow the same pattern syntax as PATTERN_MATCHING (EDR-025).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| C-style `for (;;)` included | Would add a second iteration construct with independent semantics (index, condition, step). Creates two ways to iterate, violating Minimal Core. Range syntax covers all index-based patterns. |
| While as general iteration only | Would lose the sequence-consumption ergonomics of `for ... in`. `for` is declarative (what to iterate); `while` is imperative (loop until condition). |
| No `loop` keyword | `loop { ... }` with `break value` is cleaner than `while true { ... }` for expression-oriented loops. Rust's pattern is proven. |
| For-each with optional index | The `enumerate()` combinator (EDR-032) already provides index access. Adding index syntax to `for` would create two ways to get the index. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (new language construct — loop syntax with desugaring): All seven gates.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I need to iterate over a sequence of values." `for ... in` is the most intuitive and widely-used iteration construct. Serves VISION.md's Comfortable by Design pillar — familiar syntax with safe semantics. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All constructs have precise definitions consistent with EDR-022. `for` desugars deterministically. `break`/`continue` have defined semantics across both `for` and `while`. No self-referential paradoxes. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The loop model is minimal — one iteration construct, one condition construct." Removing `for` would force `while` + manual iterator management. Removing `while` would force `loop` + conditional `break` for condition-based looping. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | `for` desugars to the iterator protocol (EDR-022) within the same Core Language layer. `while` is a primitive control flow construct. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Loop semantics are strategy-independent — iteration follows the `next()` protocol regardless of allocation or evaluation strategy. `for` desugaring is a syntactic transformation applicable in any implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "`for item in sequence` iterates over the sequence; `while condition` loops until false." The model matches Python, Rust, Swift — proven across decades. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | `for ... in` and `while` are among the most LLM-generable constructs. The loop desugaring to iterator protocol matches Rust. Destructuring in loop variables matches Python and Rust patterns. Self-correction: non-iterable source in `for` is a compile error; `while` with non-boolean condition is a type error. |

**Gates not applied:** None — all seven gates are required.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
