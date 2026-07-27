# EDR-050: Generators — Bidirectional Yield and Generator Expressions

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's lazy sequence model was established in [EDR-021](./EDR-021-lazy-sequence-generators.md): generator functions using the `emit` keyword, compiled to stackless state machines implementing `Iterator[T]`. LAZY_SEQUENCE_GENERATORS defines the **core `emit` semantics** for lazy production.

The research document at `how/concepts/research/important/GENERATORS.md` proposes extending the generator model with:
- **Bidirectional generators** — `yield` optionally accepts values back from the consumer (two-way generators, like Python's `generator.send(value)`)
- **Generator expressions** — inline generator syntax: `(x * x for x in 1..10 if x % 2 == 0)`
- **Iterator combinators** — `.take()`, `.filter()`, `.map()`, `.collect()` on generators
- **Async generators** — `async yield` — generators that can also await at yield points

The question: **do these extensions add new semantics beyond LAZY_SEQUENCE_GENERATORS (EDR-021)?**

Analysis:
- **Bidirectional generators** (`yield expr` where `expr` receives a value from the caller) add new two-way communication semantics not present in the one-way `emit` model. The function can receive values from the consumer during iteration.
- **Generator expressions** are syntactic sugar — they desugar to generator functions.
- **Iterator combinators** are StdLib (per EDR-032) — they are method implementations on `Iterator[T]`.
- **Async generators** depend on the async model (EDR-047) — they combine `async` with `emit`. Deferred to async streams specification.

The Decision Pipeline classified GENERATORS as **Language**: Bidirectional yield adds producer-consumer communication semantics (send values back to generator) not present in EDR-021's one-way `emit` model. Generator expressions require compiler-level desugaring.

---

### Decision

Adopt the following generator extensions for Orthon:

1. **`yield` as bidirectional `emit`** — The `yield` keyword provides two-way communication: producing a value and optionally receiving a value from the consumer. `yield expr` emits `expr` to the consumer and receives a value back; `yield` with no expression is equivalent to `emit` (one-way).

    ```orthon
    fun interactive() -> Iterator[String]
        let prefix = yield "ready"
        emit prefix ++ ": working"
    
    let gen = interactive()
    let msg = gen@next()        # "ready"
    gen@send("user")            # send "user" back to generator
    ```

2. **Generator expressions** — Parenthesised inline generator syntax for simple sequences. Desugars to generator functions.

    ```orthon
    let squares = (x * x for x in 1..10)
    let evens = (x for x in 1..100 if x % 2 == 0)
    ```

3. **Generator expression semantics** — Lazy by default (per EDR-021). `for x in source if condition` desugars to `source.filter(\|x\| condition)`. `for x in source` desugars to a loop emitting `x`.

4. **`yield from` (delegation)** — Delegate generation to a sub-generator:

    ```orthon
    fun combined() -> Iterator[Int]
        yield from fib(10)
        yield from fib(20)
    ```

5. **Bidirectional generator trait** — A `BidirectionalGenerator[T, U]` trait where `next()` returns `T` and `send(value: U)` sends `U` back to the generator.

**Relationship to EDR-021:** EDR-021 establishes the core `emit` model. This EDR extends it with bidirectional communication (`yield`) and syntactic sugar (generator expressions). The `emit` keyword remains the canonical one-way form.

---

### Consequences

**Positive:**
- Bidirectional `yield` enables coroutine-style two-way communication without adding a separate channel or callback mechanism.
- Generator expressions provide concise syntax for simple lazy sequences, matching familiar patterns from Python.
- `yield from` enables natural generator composition and delegation.
- Full backward compatibility with EDR-021's `emit` model — `yield` without expression is equivalent to `emit`.
- Iterator combinators remain StdLib per EDR-032 — no language additions for transformation chains.

**Negative:**
- `yield` introduces a second keyword for sequence production alongside `emit` — two keywords for closely related concepts.
- Bidirectional generators add complexity to the state machine transformation (must track resume values from consumer).
- Generator expressions add syntax parsing complexity (parenthesised form with optional `if` filter).
- Async generators depend on both the async model (EDR-047) and the generator model — deferred to avoid cross-dependency.

---

### Compliance

1. The `what/concepts/GENERATORS.md` specification defines the canonical semantics.
2. `yield` without expression must be semantically equivalent to `emit` (one-way production, no consumer interaction).
3. `yield expr` (where `expr` receives a value from the consumer) must track the received value in the state machine.
4. Generator expressions must desugar to generator functions with the same semantics — no special runtime behaviour.
5. `yield from` must delegate to the sub-generator's iterator protocol.
6. `BidirectionalGenerator[T, U]` must be a trait that extends `Iterator[T]` with a `send(value: U)` method.
7. All generator extensions must be implementable as stackless state machine transformations (default strategy).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| One-way `emit` only (no bidirectional) | Already settled in EDR-021. Bidirectional yield adds two-way communication, useful for interactive generators and coroutine patterns. |
| Generator expressions as StdLib only | Would require combinator chains for every expression — less ergonomic than inline syntax. The parenthesised form is well-understood (Python, Rust). |
| No `yield from` | Would force manual iteration over sub-generators — `for x in sub: emit x` — adding boilerplate without semantic value. |
| Async generators now | Depends on both EDR-047 (async) and EDR-021 (emit). Deferred to avoid cross-dependency — will be specified when both models are stable. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (new language construct — generator expressions + bidirectional yield add syntax and semantics).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to produce sequences lazily, and sometimes communicate back to the producer." Generator expressions solve the "concise inline sequence" problem. Bidirectional `yield` solves the "interactive generator" problem. Both are proven patterns (Python, Lua). |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | `yield` without expression ≡ `emit` (defined equivalence). `yield expr` adds consumer-to-producer communication (defined semantics). Generator expressions desugar to generator functions deterministically. No self-referential paradoxes. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "Bidirectional yield adds one new capability (consumer-to-producer communication) not expressible via EDR-021's one-way `emit`." Verification: two-way communication requires either channels, callbacks, or bidirectional yield. Channels add more complexity; callbacks invert control. Bidirectional yield is the simplest model. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Generator expressions and yield extend LAZY_SEQUENCE_GENERATORS (EDR-021) within the same Core Language layer. No layer violations. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Bidirectional semantics are strategy-independent: the state machine includes a storage slot for the consumer's sent value, regardless of allocation or evaluation strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "A generator can optionally receive values from its consumer using yield." Remove-one-thing test: removing bidirectional yield loses two-way communication capability — channels are a heavier alternative. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Generator expressions match Python comprehensions — highly LLM-generable. `yield` patterns match Python generators. `yield from` matches Python's `yield from`. Bidirectional yield (`gen.send()`) is a well-known Python pattern. Self-correction: type mismatches in bidirectional yield produce compile errors. |

**Gates not applied:** None — all seven gates are required.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
