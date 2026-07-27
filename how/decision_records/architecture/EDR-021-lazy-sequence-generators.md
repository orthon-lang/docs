# EDR-021: Lazy Sequence Generators — `emit` Keyword for Lazy Production

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs a mechanism for producing lazy sequences — values emitted one at a time on demand — without manual iterator state management. The language's design principles demand lazy-by-default evaluation (Phase 3 D-06), composition without intermediate allocation, and support for infinite sequences.

The research document at `how/concepts/research/essential/LAZY_SEQUENCE_GENERATORS.md` established the hypothesis, derived from imperative crutch analysis of manual iterator implementation in Python, Java, and other languages. Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Manually implementing an iterator requires writing stateful classes or objects — too much boilerplate. The language should provide a first-class mechanism for producing lazy sequences.
2. **Step 2 (Minimal Solution):** Generator functions using an `emit` keyword, compiled to state machines that implement `Iterator[T]`. Three equivalent canonical forms: `emit value`, `return sequence(value)`, `return value ->`. Lazy by default. Infinite sequences valid.
3. **Step 3 (Principle Check):** Aligns with Intent Over Implementation (programmer describes what to produce, compiler handles state machine), Minimal Core (generators replace manual iterator classes), Explicitness (`emit` makes lazy production syntactically visible).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/LAZY_SEQUENCE_GENERATORS.md` with `orthon` code blocks, including generator syntax, composition with iterators, infinite sequences, and completion semantics.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4, Wave 1) classified LAZY_SEQUENCE_GENERATORS as **Language** per D-03: Lazy sequence semantics (`emit`) are a compiler-recognized pattern with special evaluation guarantees (lazy-by-default, per Phase 3 D-06). Not expressible via primitives alone.

---

### Decision

Adopt **generator functions with `emit` keyword** for lazy sequence production in Orthon:

1. **`emit` keyword** — The canonical form for producing values from a generator. `emit value` yields a single value and suspends execution.
2. **All canonical forms equivalent** — `emit value`, `return sequence(value)`, and `return value ->` have identical semantics.
3. **Lazy by default** — Generator bodies execute lazily (per Phase 3 D-06). Values are produced on demand via the iterator protocol. Materialisation is explicit via `.collect()`.
4. **Generators implement `Iterator[T]`** — A generator function returns an `Iterator[T]`, making generators composable with all iterator combinators.
5. **State machine compilation** — Generator bodies compile to stackless state machines (not heap-allocated coroutines).
6. **Infinite sequences valid** — The language does not prevent infinite generators. Termination is controlled by the consumer.
7. **Composition without allocation** — Generator expressions compose via iterator combinators without intermediate collection allocation.

---

### Consequences

**Positive:**
- Eliminates manual iterator implementation boilerplate — one `emit` replaces a stateful class.
- Lazy by default — values are produced on demand, enabling infinite sequences and efficient composition.
- Generators are automatically iterators — full interop with the iterator protocol and combinator chain.
- Stackless state machine compilation — no per-generator heap allocation.
- Three equivalent canonical forms give the programmer syntactic choice.
- Composition without allocation — combinators produce lazy iterators, not eager collections.

**Negative:**
- State machine compilation adds compiler complexity (control-flow graph to state machine transformation).
- Generators cannot easily express certain patterns (recursive generators, mutual generators) without heap allocation.
- The `emit` keyword adds a new syntactic form to the language.
- Debugging generators is harder than debugging eager functions (execution is split across `next()` calls).

---

### Compliance

1. The `what/concepts/LAZY_SEQUENCE_GENERATORS.md` specification defines the canonical semantics.
2. Generator functions must return `Iterator[T]` — the compiler infers the type parameter `T` from `emit` expressions.
3. The `emit` keyword must desugar to iterator protocol `next()` calls in all implementations.
4. All three canonical forms (`emit`, `return sequence()`, `return ->`) must produce identical semantics.
5. Lazy evaluation is the default — no eager materialisation without explicit `.collect()`.
6. Infinite generators must be supported (the language must not impose an artificial "finite-only" constraint).
7. Generator state must be stored in a fixed-size struct (no heap allocation per generator instance in the default strategy).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `yield` keyword (Python, C#) | `yield` is passive and ambiguous (also used in `yield from`). `emit` is an active verb consistent with Orthon's naming conventions. |
| Manual `Iterator` implementation only | Would require programmer to write stateful classes for every lazy sequence. Violates Minimal Core — the boilerplate is not doing semantic work. |
| Eager evaluation by default | Would break infinite sequences. Would allocate intermediate collections for combinator chains. Violates Phase 3 D-06 (lazy by default). |
| Coroutines (full/general) | Generator state machines are a restricted form of coroutines. Full coroutines add complexity (separate stacks, yield-to semantics) not needed for sequence production. |
| List comprehensions only | Eager by definition, no lazy composition. Cannot express infinite sequences. Would require additional syntax for lazy variants. |
| Reactive streams (RxJS, ReactiveX) | Push-based model (observables) is the dual of pull-based iteration. Larger concept surface than needed. Better suited for the Standard Library than the core language. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to produce a sequence of values without writing a stateful class." Every programmer who has implemented an iterator manually knows the boilerplate. The solution serves VISION.md's Comfortable by Design pillar — `emit` makes lazy sequence production as simple as writing a loop. Code example from LAZY_SEQUENCE_GENERATORS.md (`natural_numbers()`) shows a three-line generator replacing a multi-field stateful class. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All constructs have precise definitions. Generator = function with `emit`. `emit` = produce value + suspend. Completion = control flow exits function body. All three canonical forms are provably equivalent (they desugar to the same state machine). No self-referential paradoxes — a generator does not generate itself. The lazy-by-default rule applies consistently: no `emit` call executes until `next()` is called. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The generator model is minimal — removing any component makes lazy sequence production incomplete." Removing `emit` would force manual iterator implementation. Removing lazy evaluation would break infinite sequences. Removing `Iterator[T]` conformance would force a separate protocol for consumption. Removing stackless compilation would force heap allocation. Result: all components are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Generators operate at Level 1–2 boundary (Primitive Operations / Language Patterns) in the Semantic Dependency Architecture. The `emit` keyword composes Level 1 primitives (`function`, `call`, `scope`) into a Level 2 pattern (state machine). Generators implement `Iterator[T]` (Level 2 trait), which is consumed by `for` loops (Level 2). No layer violations — generators do not depend on the Standard Library for their core semantics. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: generators require a state-machine transformation (seems strategy-dependent), yet generator semantics must be strategy-independent. Separation: the *semantic definition* of a generator is "a function that produces values on demand" — the state machine transformation is a Strategy choice. Stackless state machine (Default), stackful coroutine with separate stack (High Performance), or CPS transformation (Embedded) are all valid. Lazy evaluation guarantee is identical across strategies. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "A generator is a function that produces values one at a time using `emit`, and the compiler handles all the state management." Explainable without "and", "but", "except". Remove-one-thing test: removing generators would force manual iterator implementation everywhere. The concept matches established patterns (Python `yield`, C# `yield return`, Rust's generator RFC). Evolution path: new `emit` forms, async generators, and bidirectional generators can be added without changing the core model. No conceptual debt. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `emit value` has a single, unambiguous meaning — produce a value and suspend. No context-dependent syntax. Schema round-trip: generator signature `fn name() -> Iterator[T]` is fully expressible in the type system. Hallucination surface: low — the pattern matches Python `yield` and Rust generators, which LLMs generate reliably. Self-correction: missing `Iterator[T]` return type is a compile error, incorrect `emit` placement outside a generator is a compile error, type mismatches between `emit` values and return type are detected. Common LLM mistakes (forgetting `emit`, using `return` instead of `emit`) produce clear compile errors. |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/LAZY_SEQUENCE_GENERATORS.md` — Full concept specification
- `what/concepts/ITERATOR_PROTOCOL.md` — The consumption side of the sequence model. Generators implement `Iterator[T]`.
- `what/SEMANTIC_MODEL.md` — Ownership and Mutation (generator state transitions)
- `what/GLOSSARY.md` — Lazy Sequence, Generator
- `how/concepts/research/essential/LAZY_SEQUENCE_GENERATORS.md` — Original research document
- `how/concepts/research/important/GENERATORS.md` — Extended generator analysis
- `how/concepts/research/essential/EXECUTION_PROGRAM.md` — Sequence semantics in execution programs
- `how/DESIGN_PRINCIPLES.md` — Minimal Core, Explicitness, Intent Over Implementation
- Phase 3 D-06: Lazy by default for `emit`
- Phase 3 D-07: `@` for Metadata Protocol

### Supersedes

*None* — this is a new decision, not a replacement.
