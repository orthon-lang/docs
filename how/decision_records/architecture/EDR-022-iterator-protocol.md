# EDR-022: Iterator Protocol — Trait-Based Lazy Consumption

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs a lazy, composable, memory-efficient mechanism for consuming sequences. The language already has generators (see EDR-021) for *producing* sequences lazily. The iterator protocol defines the *consumption* side — the trait that makes `for` loops, combinator chains, and collection iteration work uniformly.

The research document at `how/concepts/research/essential/ITERATOR_PROTOCOL.md` established the conceptual model, analysing the spectrum from manual loops through iterator interfaces to stream APIs. Five steps of the Concept Design Review were completed:

1. **Step 1 (Idea/Problem):** Orthon needs a uniform way to consume sequences lazily — whether from generators, collections, ranges, or I/O streams — without forcing the programmer to manage iterator state manually.
2. **Step 2 (Minimal Solution):** A trait-based `Iterator[T]` protocol with `next() -> Option[T]`, `for` loop desugaring, `IntoIterator[T]` for collection integration, and standard combinators as default method implementations (StdLib). Range expressions with exclusive/inclusive/step syntax.
3. **Step 3 (Principle Check):** Aligns with Minimal Core (one protocol covers all iteration), Orthogonality (Iterator is consumption, generators are production), Composition (combinators chain without intermediate allocation).
4. **Step 4 (Examples):** All canonical forms documented in `what/concepts/ITERATOR_PROTOCOL.md` with `orthon` code blocks, including `for` loop desugaring, combinator chains, `IntoIterator`, range expressions, and `@` protocol method access.
5. **Step 5 (EDR):** This document.

The Decision Pipeline (processed in Phase 4, Wave 1) classified ITERATOR_PROTOCOL as **Language** per D-03: Protocol definition (`next() -> Option[T]`) is a compiler-level concept (trait with special `for` loop desugaring). Combinators should be StdLib.

---

### Decision

Adopt the **trait-based `Iterator[T]` protocol** for Orthon sequence consumption:

1. **`Iterator[T]` trait** — `fn next(self) -> Option[T]`. Single-pass, lazy consumption protocol. Any type implementing this trait can be consumed by `for` loops and combinator chains.
2. **`for` loop desugaring** — `for item in iter` desugars to `IntoIterator::iter()` + loop calling `next()`. Compiler enforces that the operand implements `IntoIterator[T]`.
3. **`IntoIterator[T]` trait** — `fn iter(self) -> Iterator[T]`. Enables collections, ranges, and I/O streams to participate in `for` loops. `Iterator[T]` implements `IntoIterator[T]` (returning `self`).
4. **`@` for protocol method access** — Per Phase 3 D-07, `iterator@next()` distinguishes protocol access from attribute access.
5. **Standard combinators as StdLib** — `map`, `filter`, `take`, `skip`, `flat_map`, `zip`, `enumerate`, `collect`, `fold`, `for_each`, `count`, `all`, `any` are default method implementations on `Iterator[T]` living in the Standard Library.
6. **Range expressions** — *Superseded by EDR-083 (see Amendments).* Ranges are now defined by the RANGE concept: inclusive-inclusive `1..N`; `Range` implements `IntoIterator`. Zero-cost — compile to counter loops.
7. **Dependency on COMPOSABLE_COLLECTION_OPS** — The `IntoIterator[T]` trait and iterator combinators provide the foundation for composable collection operations. COMPOSABLE_COLLECTION_OPS (Plan 04-03) builds on this foundation with collection-specific operations (sort, unique, join, group_by).

---

### Consequences

**Positive:**
- Single, uniform protocol for all sequence consumption — generators, collections, ranges, I/O streams.
- Lazy by default — combinator chains produce no intermediate allocations.
- Zero-cost — monomorphisation eliminates combinator overhead; ranges compile to counter loops.
- Single-pass semantics — no buffering or backtracking required by implementations.
- `IntoIterator` separates "can be iterated" from "is an iterator" — clean semantic distinction.
- `@` prefix makes protocol access syntactically visible per Semantic Purity.
- Combinators as StdLib keeps the core language minimal while enabling rich expression.

**Negative:**
- Single-pass semantics means some algorithms (peek, lookahead, multi-pass) require buffering.
- No built-in parallel iteration — parallel combinators require separate concepts.
- Combinator method count on `Iterator` grows over time as new combinators are added to StdLib.
- `@` prefix is an additional syntactic convention that programmers must learn.

---

### Compliance

1. The `what/concepts/ITERATOR_PROTOCOL.md` specification defines the canonical semantics.
2. Every implementation must support the `Iterator[T]` trait and `IntoIterator[T]` trait.
3. `for` loop desugaring must call `IntoIterator::iter()` and produce a `next()` loop.
4. All standard combinators must return lazy iterators — no intermediate allocation.
5. Range expressions must compile to zero-cost counter loops (no heap allocation). *(Range syntax/semantics per EDR-083.)*
6. Protocol method access uses `@` prefix — `iterator@next()`, not `iterator.next()`.
7. COMPOSABLE_COLLECTION_OPS (Plan 04-03) may add collection-specific operations on top of `IntoIterator`.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keyword-based iteration (Python `__iter__`/`__next__` dunder methods) | Duck-typing approach with no static dispatch. Cannot be optimised via monomorphisation. Violates Declarative With Static Guarantees. |
| Separate `Stream` and `Iterator` types (Java, C#) | Creates an unnecessary type barrier — an iterator IS a stream (produces values over time). One protocol is simpler and more orthogonal. |
| Eager iteration only (no lazy protocol) | Would force collection materialisation for all transformations. Cannot express infinite sequences. Violates Minimal Core — requires separate lazy mechanism. |
| No `IntoIterator` — `for` accepts `Iterator` only | Would force callers to call `.iter()` explicitly. Reduces ergonomics for collection iteration. |
| `Iterator` as a built-in type (not a trait) | Would prevent user-defined types from implementing the protocol. Violates Extensibility. |
| Push-based iteration (Observable pattern) | The dual of pull-based iteration. More complex (backpressure, subscription management). Better suited for asynchronous/streaming contexts in the Standard Library. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to iterate over a sequence and transform it without writing temporary collections." Every programmer who has chained `for` loops with intermediate arrays knows the pain. The solution serves VISION.md's Comfortable by Design pillar — the iterator protocol makes lazy, zero-cost iteration natural. Code example from ITERATOR_PROTOCOL.md (`users.filter().map().take().collect()`) shows a four-operation chain without a single intermediate allocation. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All constructs have precise, non-overlapping definitions. `Iterator` = consumption protocol. `IntoIterator` = convert-to-iterator. `for` = desugared `next()` loop. Combinators = lazy method implementations. Range = counter-based iterator. `@` = protocol access. No self-referential paradoxes — an iterator does not iterate itself (though `Iterator[T]` implements `IntoIterator[T]` by returning `self`). Single-pass semantics are consistently enforced. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The iterator protocol is minimal — removing any component makes sequence consumption incomplete." Removing `Iterator[T]` would leave no protocol for consumption. Removing `IntoIterator[T]` would force every `for` loop to explicitly get an iterator. Removing combinatorial laziness would force eager materialisation. Removing `@` protocol access would conflate attribute and protocol access. Removing range expressions would force manual counter loops. Result: all components are necessary. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | The iterator protocol operates at Level 2 (Language Patterns) in the Semantic Dependency Architecture — it composes Level 1 primitives (`function` for `next()`, `call` for invocation, `loop` for consumption) into a higher-level iteration pattern. `for` loop desugaring is compiler-level but composes primitives. `IntoIterator[T]` is a trait interface (per EDR-019). Combinators are Level 2 patterns in the Standard Library. No layer violations — the protocol does not depend on StdLib for its core semantics. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: iterator combinators require monomorphisation for zero-cost (seems strategy-dependent), yet iteration semantics must be strategy-independent. Separation: the *semantic definition* of an iterator is "a value with a `next()` method that returns elements until exhausted" — monomorphisation is a Strategy choice. Monomorphised tight loop (Default), interpreted dispatch (Development), or vtable-based iteration (Dynamic) are all valid. Semantics (which elements are produced in which order) are identical. Range iteration is a counter loop in all strategies. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "An iterator produces values one at a time via `next()`, and combinators transform those values without allocating collections." Explainable without "and", "but", "except". Remove-one-thing test: removing the iterator protocol would force the language back to manual loops for all sequence consumption — no lazy composition, no combinator chains. The concept matches established patterns (Rust `Iterator`, Java `Iterator` with streams). Evolution path: new combinators can be added to StdLib; `DoubleEndedIterator` can be added as a separate trait; parallel combinators can extend the protocol. No conceptual debt. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `Iterator[T]`, `next()`, `IntoIterator[T]`, `for item in`, ranges (`0..10`), combinator chains, and `@next()` each have a single, unambiguous meaning. No context-dependent syntax. Schema round-trip: fully expressible in the trait system — `Iterator` is a trait with a single required method. Hallucination surface: low — the pattern matches Rust's `Iterator` trait, which LLMs generate reliably. Self-correction: missing `next()` implementation is a compile error, wrong type parameter produces a type error, non-iterable type in `for` loop is a compile error. Common LLM mistakes (forgetting `@` for protocol access, using iterator after consumption) are statically detectable. |

**Gates not applied:** None — all seven gates are required for an architecture-level decision establishing a new language construct.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.

### Related Concepts

- `what/concepts/ITERATOR_PROTOCOL.md` — Full concept specification
- `what/concepts/LAZY_SEQUENCE_GENERATORS.md` — The production side of the sequence model. Generators return `Iterator[T]`.
- `what/concepts/TRAITS.md` — The trait system that `Iterator` and `IntoIterator` depend on
- `what/SEMANTIC_MODEL.md` — Ownership and Mutation (borrowing vs. consuming iteration)
- `what/GLOSSARY.md` — Iterator Protocol, IntoIterator
- `how/concepts/research/essential/ITERATOR_PROTOCOL.md` — Original research document
- `how/concepts/research/essential/DATA_MODEL.md` — Sequences as a data representation
- `how/concepts/research/important/ITERATION_LOOP.md` — Loop constructs and `for` desugaring
- `how/concepts/research/essential/EXECUTION_PROGRAM.md` — Sequences in execution programs
- COMPOSABLE_COLLECTION_OPS (Plan 04-03) — Extension of the iterator protocol with collection-specific operations (sort, unique, join, group_by). The iterator protocol's `IntoIterator[T]` and combinator pattern provide the dependency foundation for COMPOSABLE_COLLECTION_OPS. The iterable/iterator separation enables collection operations to be expressed as iterator combinator chains, with materialisation only at `.collect()`.
- `how/DESIGN_PRINCIPLES.md` — Minimal Core, Orthogonality, Explicitness, Semantic Purity
- Phase 3 D-07: `@` for Metadata Protocol

### Supersedes

*None* — this is a new decision, not a replacement.

---

### Amendments

**2026-08-05 — Range semantics superseded by [EDR-083](./EDR-083-range.md).**
Decision item 6 and Compliance item 5 are superseded: range syntax and semantics are
defined by the RANGE concept as inclusive-inclusive `1..N`. The spellings `0..10`,
`0..=10`, and `0..10:step(2)` are retired. `Range` implements `IntoIterator` and enters
combinator chains directly. ITERATOR_PROTOCOL remains authoritative for the
`Iterator[T]`/`IntoIterator[T]` traits, `for` desugaring, and StdLib combinators.
`.enumerate()` base is pinned to 1 per EDR-082 (`enumerate(items) ≡ zip(1..len(items), items)`).
