# EDR-032: Composable Collection Operations

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Standard Library

---

### Context

Phase 4 must decide how Orthon provides declarative collection transformations (map, filter, reduce, find, etc.). The research document [`COMPOSABLE_COLLECTION_OPS.md`](../../how/concepts/research/essential/COMPOSABLE_COLLECTION_OPS.md) proposes these as a standard library concern, building on the [`ITERATOR_PROTOCOL`](EDR-022-iterator-protocol.md).

The Decision Pipeline Q2 asks: *Is this a language problem or a library problem?*

| Operation | Language dependency | StdLib classification |
|---|---|---|
| `.map()` | Requires `Iterator[T]` trait (EDR-022 — Language) | **StdLib** — method on `Iterator[T]` |
| `.filter()` | Requires `Iterator[T]` trait (EDR-022 — Language) | **StdLib** — method on `Iterator[T]` |
| `.fold()` / `.reduce()` | Requires `Iterator[T]` trait (EDR-022 — Language) | **StdLib** — method on `Iterator[T]` |
| `.collect()` | Requires `Iterator[T]` trait (EDR-022 — Language) | **StdLib** — method on `Iterator[T]` |
| Loop fusion (optimisation) | None — performance only | **Implementation Strategy** |

Each operation is a composition of `Iterator[T].next()` calls. The compiler does not need special knowledge of specific combinator names — it only needs to know the `Iterator[T]` trait and the `for` loop desugaring (already decided in EDR-022).

### Decision

Classify **composable collection operations as Standard Library**, not core language:

1. **StdLib, not language** — `.map()`, `.filter()`, `.reduce()`, `.fold()`, `.find()`, `.any()`, `.all()`, `.count()`, `.collect()`, `.to_list()`, `.to_set()`, `.take()`, `.skip()`, `.take_while()`, `.skip_while()`, `.chain()`, `.zip()`, `.enumerate()` are methods on `Iterator[T]` defined in the Standard Library.
2. **Lazy by default** — All combinators return lazy `Iterator` values. Materialisation is explicit (`.collect()`, `.to_list()`, etc.). Per Phase 3 D-06.
3. **No comprehension syntax** — Comprehensions are not added as language syntax in v0.1. They may be added later as syntactic sugar over combinator chains.
4. **Loop fusion is Implementation Strategy** — Whether multiple combinator passes are fused into a single loop is an optimisation decision, not language semantics.
5. **`@` prefix for protocol methods** — Protocol method calls on iterators use the `@` prefix per Phase 3 D-07.

### Consequences

**Positive:**
- Zero language-level additions — the `Iterator[T]` trait (EDR-022) provides everything needed.
- StdLib combinators can evolve independently of language specification.
- Each combinator is a simple function composition — easy to specify, implement, and verify.
- Loop fusion as Implementation Strategy keeps semantics clean — combinators mean what they say, optimisation is separate.
- New combinators can be added without language version changes.

**Negative:**
- No special syntax for common patterns (e.g., comprehensions). Programmers write `.map()` and `.filter()` chains.
- Without comprehension syntax, deeply nested transformations are less concise.
- Eager programmers may forget to call `.collect()`, expecting materialised results from lazy iterators.

### Compliance

1. All combinators must be implementable as methods on `Iterator[T]` without compiler special-casing.
2. Each combinator must have a documented semantic specification (what it produces, not how it optimises).
3. Loop fusion must be documented as an Implementation Strategy concern, not a language guarantee.
4. StdLib combinator definitions must live in `docs/what/stdlib/` and reference this EDR.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| **Language-level comprehensions** | Adds syntax to the language. Comprehensions can be added in v0.2+ as sugar over combinator chains — no need to decide now. |
| **Language-level map/filter keywords** | Violates Minimal Core — these are function compositions, not irreducible operations. |
| **Eager by default** | Would violate Phase 3 D-06 (lazy by default) and require explicit `.lazy()` to defer. |
| **Built-in parallel combinators** | Parallellism model depends on CONCURRENCY_MODEL (EDR-033). Deferred to v0.2+. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Combinators solve the real problem of imperative loop boilerplate (see crutch analysis in research doc). |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | All combinators are compositions of `Iterator[T].next()`. No new semantic axioms needed. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | StdLib classification is the simplest possible answer — no language changes needed. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Combinators operate entirely within the Standard Library layer, depending on `Iterator[T]` (Language layer). |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Combinator semantics are strategy-independent. Loop fusion (optimisation) is explicitly delegated to implementation. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | StdLib classification means combinators can be added, deprecated, or changed without language version changes. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Combinators have simple, well-defined signatures. The Iterator combinator pattern is well-established across languages (Rust, Swift, Kotlin). |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-032 for per-gate reasoning trail.
