# EDR-083: Range — Inclusive-Inclusive Range as a First-Class Value

**Status:** Accepted

**Date:** 2026-08-05

**Category:** Architecture

**Scope:** Subsystem (Collection Semantics / Data Model)

---

### Context

INDEXING ([EDR-082](./EDR-082-1-based-indexing.md)) locked the range norm: inclusive-inclusive `1..N` is the *only* range semantic, applied uniformly to index access, slices, and iteration. But the range itself was not yet a defined concept — it remained buried inside ITERATOR_PROTOCOL ([EDR-022](./EDR-022-iterator-protocol.md)) as `0..10` (exclusive) / `0..=10` (inclusive) range expressions, all of which now contradict the accepted norm.

The 2026-08-05 ITERATOR_PROTOCOL review (routing B3) concluded that Range/Slice needs a semantic definition **separate from the Iterator trait** — recorded in `how/concepts/research/essential/RANGE_SLICE.md` (Type A gap). Two parallel hypotheses raised open questions: `RANGE_STEP.md` (step outside the literal) and `SEQUENCE_METHODS.md` (combinators apply directly to range values).

The problem: **how are ranges defined as first-class values, how do they relate to — but stay distinct from — the Iterator protocol, and what is the concrete spelling under the inclusive-inclusive norm?**

---

### Decision

1. **Range is a first-class value type**, semantically separate from the `Iterator` trait. `Range` is a compact descriptor (a `pack` composite: `start`, `end`) that produces a contiguous integer sequence on demand. It is not an iterator and not a collection.
2. **Literal `a..b` is inclusive-inclusive** — `1..N` produces N elements (1, 2, …, N). This is the *only* range semantic (per EDR-082). The `..=` spelling is **eliminated** — a single spelling, no redundant alias.
3. **Named canonical form** `range(a, b)` (Named Before Symbolic): `range(a, b)` ≡ `a..b`. The literal is **Language** (compiler-recognized; participates in `@get` indexing and `for` desugaring); the `Range` type and `range()` constructor are **Standard Library** (LIBRARY_BOUNDARY, per EDR-082 layer classification).
4. **`0..<N` is an FFI-boundary interop utility only** — never in application code. Visibility is determined by the FFI index-translation policy (Milestone 8; INDEXING Open Q4).
5. **`Range` implements `IntoIterator[Int]`** — usable in `for` loops and combinator chains directly, without an explicit `.iter()`. Combinators (`map`, `filter`, …) apply directly to the range value (SEQUENCE_METHODS). Chains stay lazy.
6. **`.step(n)` is a method on `Range`** returning a *strided* `Range` value (still a data value, still `IntoIterator`). `step(0)` is a compile-time error. A negative step iterates in descending direction.
7. **Empty range is a value** — `end < start` (e.g., `1..0`) yields zero elements, not a syntax error.
8. **`enumerate` composition reconciled** — the spelling from EDR-082/INDEXING is corrected to `enumerate(items) ≡ zip(1..len(items), items)` (the `..=` form is superseded by this decision).

---

### Consequences

**Positive:**
- One spelling, one semantic — inclusive-inclusive `1..N` everywhere; no `..=` alias to remember.
- Range is orthogonal to iteration: a value that implements `IntoIterator`, composable with combinators, reusable across index access, slices, and loops.
- The language owns the `+1` length arithmetic: `1..N` is N elements; `range(a, b)` is `b - a + 1` elements.
- Retires the 0-based legacy range text from accepted docs (superseding the range portions of EDR-022/EDR-053).
- Zero-cost: ranges compile to counter loops in all strategies.

**Negative:**
- `..=` removal is a (small) deviation from the freshly-accepted EDR-082 formula spelling — a documented reconciliation, not a semantic reversal.
- Step semantics (`step(0)` error, negative step) add a small surface beyond the core value.
- Retires the familiar `0..N` spelling entirely, including in code that predates the norm (handled by the C-001 sweep, TODO IX-B1).

---

### Compliance

1. `1..N` is the only range semantic in application code; no half-open form; `..=` does not exist.
2. `range(a, b)` ≡ `a..b`; the named form is StdLib, the literal is Language.
3. `Range` implements `IntoIterator[Int]`; combinator chains apply directly and stay lazy.
4. `.step(n)` returns a strided `Range` value; `step(0)` is a compile-time error.
5. Empty range `end < start` is a value with zero elements.
6. `0..<N` appears only at the FFI boundary.
7. `enumerate(items) ≡ zip(1..len(items), items)` (spelling per this decision).
8. `what/concepts/RANGE.md` defines the canonical semantics; ITERATOR_PROTOCOL/ITERATION_LOOP range text delegates here.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Range owned by the Iterator trait (status quo) | Buried the range inside the consumption protocol; forced the 0-based exclusive/inclusive split; cannot express slicing cleanly. |
| Keep `..=` as an alias for `..` | Two spellings for one semantic violate Explicitness and Minimal Core; the EDR-082 formula used `..=` only as a notational slip. |
| Half-open `a..b` with `..=` inclusive (Rust-style) | Contradicts the accepted inclusive-inclusive norm (EDR-082) and re-introduces the `+1` arithmetic the norm removes. |
| Range as a built-in keyword type | Range composes from existing primitives (`pack` composite + `IntoIterator`); only the literal syntax needs Language support. |

### Gate Validation

> Required for all Architecture-category EDRs — a new Core Language semantic requires the full catalogue of 7 gates (`DECISION_VALIDATION.md` § Gate Selection).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | Programmer writes `1..10`, gets 10 elements, no `+1`/`-1`; one spelling, one semantic; direct translation from mathematics. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../gates/methods/SOCRATIC_METHOD.md) | Pass | `Range`, `range(a, b)`, `IntoIterator`, `.step(n)`, empty-range all have precise, non-overlapping definitions; no self-reference; reconciles the `enumerate` formula. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Removing any component breaks a use case: no value → no reuse across index/slice/loop; no `IntoIterator` → no direct combinators; no `.step` → strided iteration impossible. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Literal semantics = Core (Level 2 pattern over `pack`/`call`); `Range` type + methods = StdLib; no layer violation; range no longer depends on the iterator protocol for its identity. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction (zero-cost counter loop vs strategy-agnostic semantics) separated: semantics = which integers in which order; representation (counter loop, unrolled, vectorized) = Strategy. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "a range is a first-class inclusive-inclusive run of integers that can be iterated or sliced." No "but/except"; supersedes EDR-022/EDR-053 range text cleanly. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | `1..N` maps to human/mathematical notation; a single spelling removes the `..` vs `..=` choice an LLM must otherwise track; `range(a, b)` named form is LLM-friendly; `step(0)` is statically detectable. |

**Gates not applied:** none — a new Core Language semantic requires the full catalogue.

**Detailed reasoning:** See `DECISION_LOG.md` § Entry: Range (EDR-083) for the per-gate reasoning trail.

---

### Related Concepts

- `what/concepts/RANGE.md` — Full concept specification
- `what/concepts/INDEXING.md` — 1-based base and the inclusive norm (EDR-082)
- `what/concepts/SLICE.md` — Range applied to a random-access composite (EDR-084)
- `what/concepts/ITERATOR_PROTOCOL.md` — `IntoIterator`/combinators; range text now delegates here (EDR-022)
- `what/concepts/ITERATION_LOOP.md` — index-based iteration uses ranges (EDR-053)
- `how/concepts/research/essential/RANGE_SLICE.md`, `how/concepts/research/important/RANGE_STEP.md`, `how/concepts/research/important/SEQUENCE_METHODS.md` — original research
- FFI (Milestone 8) — `0..<N` interop visibility
