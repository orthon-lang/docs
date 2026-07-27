# EDR-055: Unpacking — Destructuring Assignment with Pattern Syntax

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's PRIMITIVE_BLOCKS (EDR-016) establishes `pack`/`unpack` as a fundamental primitive — value composition/decomposition with symmetric syntax. The `pack` primitive creates compound values; `unpack` decomposes them.

The research document at `how/concepts/research/important/UNPACKING.md` proposes destructuring assignment as a language feature — extracting values from compound data structures (tuples, records, arrays) into individual bindings using pattern syntax.

The core problem: **binding names to structural positions** should be syntactic, not procedural. Explicit indexing (`list[0]`, `tuple[1]`) is verbose, brittle, and non-obvious.

The alignment with PRIMITIVE_BLOCKS is clear:
- `pack` creates compound values: `let point = (x, y)` — pack
- `unpack` decomposes: `let (x, y) = point` — unpack
- The symmetry principle demands that construction and destruction share syntax

The question: **is destructuring assignment new semantics or syntactic sugar?**

Analysis: Destructuring assignment is a Language pattern that composes several primitives:
- `let (x, y) = point` desugars to `let tmp = point; let x = tmp[0]; let y = tmp[1]` — but this requires the compiler to:
  1. Recognise the pattern syntax in assignment context
  2. Generate the correct field-by-field extraction code
  3. Support nested patterns, rest patterns, rename syntax, ignore patterns

The compiler must resolve destructuring patterns — this is a syntactic transformation, not new runtime semantics. The `pack`/`unpack` primitive already provides the decomposition capability; destructuring assignment is syntax that makes it ergonomic.

The Decision Pipeline classified UNPACKING as **Language**: Destructuring syntax matches pack/unpack symmetry (PRIMITIVE_BLOCKS). Compiler must resolve destructuring patterns. However, the semantics are entirely expressible via existing primitives — destructuring is syntactic sugar over `pack`/`unpack`.

---

### Decision

Adopt destructuring assignment as a language syntax construct in Orthon:

1. **Tuple destructuring** — `let (x, y) = point` — decomposes a tuple into individual bindings.

    ```orthon
    let point = (10, 20)
    let (x, y) = point          # x = 10, y = 20
    let (first, ..rest) = list   # rest pattern
    ```

2. **Record destructuring** — `let {name, age} = person` — decomposes a record by field name.

    ```orthon
    let {name, age} = person
    let {name: n, age: a} = person  # rename bindings
    ```

3. **Nested destructuring** — Compound structures decomposed recursively.

    ```orthon
    let {address: {city, country}} = user
    let ((a, b), c) = nested_tuple
    ```

4. **Ignore pattern** — `_` for unused positions.

    ```orthon
    let (x, _, z) = triple       # second value ignored
    ```

5. **Rest pattern** — `..rest` for remaining elements.

    ```orthon
    let (first, ..rest) = list
    let {name, ..} = person      # only extract name, ignore rest
    ```

6. **Function parameter destructuring** — Destructuring in function signatures.

    ```orthon
    fun distance({x, y}: Point) -> Float
        return sqrt(x * x + y * y)
    
    fun first((x, _): Pair) -> x
    ```

7. **Destructuring in `for` loops** — Loop variable destructuring.

    ```orthon
    for {name, age} in people:
        print("{name} is {age} years old")
    
    for (index, value) in items.enumerate():
        print("Index {index}: {value}")
    ```

**Semantic specification:** All destructuring forms desugar to `pack`/`unpack` primitives. The compiler generates field-by-field extraction code. No new runtime semantics — only syntactic transformation.

```orthon
# Tuple destructuring desugaring:
let (x, y) = point
# → let tmp = point; let x, y = unpack(tmp)

# Record destructuring desugaring:
let {name, age} = person
# → let name = person.name; let age = person.age
```

---

### Consequences

**Positive:**
- Symmetry with PRIMITIVE_BLOCKS' `pack`/`unpack` — construction and destruction use matching syntax.
- Eliminates boilerplate indexing (`point[0]`, `point[1]`).
- Nested patterns reduce nesting depth in code.
- Function parameter destructuring is ergonomic — especially for record parameters.
- `for` loop destructuring pairs naturally with iterator combinators like `.enumerate()`.
- All forms desugar to existing primitives — no new runtime behaviour.

**Negative:**
- Pattern syntax must be specified in SYNTAX.md — adds parsing complexity.
- Interaction with ownership (move vs. borrow on destructuring) must be specified.
- Nested destructuring can obscure the original data structure's shape.
- Function parameter destructuring may conflict with named parameter syntax when both use `{...}`.

---

### Compliance

1. The `what/concepts/UNPACKING.md` specification defines the canonical destructuring semantics.
2. All destructuring forms must desugar to `pack`/`unpack` primitives — no new runtime semantics.
3. Destructuring syntax must mirror pattern-matching syntax (per EDR-025) for consistency.
4. Rest patterns (`..rest`) must collect remaining elements.
5. Ignore patterns (`_`) must suppress binding for unused positions.
6. Rename syntax (`{name: n}`) must bind the field value to the specified name.
7. Ownership semantics follow PATTERN_MATCHING (EDR-025): destructuring from a value moves; destructuring from a reference borrows.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Positional destructuring only (tuples, no records) | Would leave record destructuring to manual field access — asymmetric with the `pack` primitive which supports both. |
| No function parameter destructuring | Would force callers to unpack inside the function body — reduces the declarative value of signatures. |
| Destructuring via explicit `unpack` function call only | Would require manual decomposition — `let (x, y) = unpack(point)` — violating the symmetry principle. Pattern syntax is more declarative. |
| No `for` loop destructuring | Would force manual unpacking inside the loop body — `for item in items: let {name, age} = item`. Ergonomically inferior. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (new language construct — syntax change with desugaring): `LOGICAL_CONSISTENCY_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE`, `LLM_GENERABILITY_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to extract values from compound data without verbose indexing." Destructuring is proven across Rust, Python, JS, and Swift. Serves VISION.md's Comfortable by Design pillar. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Destructuring syntax mirrors pattern-matching syntax (per EDR-025) — consistent across the language. The pack/unpack symmetry is maintained: `(x, y)` creates a tuple, `let (x, y) = ...` decomposes one. All terms have precise definitions. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "Destructuring is syntactic sugar over pack/unpack — removing it loses ergonomics, not expressive power." Verification: every destructuring form desugars to existing primitives. The ergonomic gain (concise extraction) justifies the syntax. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Destructuring composes Level 1 primitives (`pack`/`unpack`, `identifier`, `assignment`) into a Level 2 Language Pattern. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Destructuring is a syntactic transformation — strategy-independent. Field-by-field extraction follows the same semantics regardless of allocation or evaluation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Destructuring lets you unpack compound values using the same syntax used to create them." Remove-one-thing test: removing destructuring would force manual indexing everywhere. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Destructuring is highly LLM-generable — the pattern matches Python, Rust, and JS destructuring. `let (x, y) = tuple` and `let {name, age} = record` are intuitive. Self-correction: incorrect destructuring patterns (missing field, type mismatch) produce compile errors. |

**Gates not applied:** None — all seven gates are required (new language construct).

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
