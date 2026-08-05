# EDR-082: 1-Based Indexing — Ordinal Collection Indexing

**Status:** Accepted

**Date:** 2026-08-05

**Category:** Architecture

**Scope:** Subsystem (Collection Semantics / Data Model)

---

### Context

How do Orthon collections identify elements by position? Two contradictory
traditions exist:

1. **0-based indexing** (C, C++, Java, Python, JavaScript, Rust, Go) — the
   first element is at index 0. Rooted in C pointer arithmetic: `arr[i]` is
   sugar for `*(base + i * sizeof(T))`.
2. **1-based indexing** (Lua, Julia, MATLAB, Fortran, R) — the first element
   is at index 1. Rooted in human and mathematical convention.

The core problem: 0-based indexing creates a persistent cognitive gap between
how humans count (1, 2, 3, …) and how machines address memory (offset 0, …).
This gap is the source of off-by-one errors — one of the most common bug
categories — and forces domain experts (mathematicians, scientists, business
analysts) to mentally translate their natural notation into the machine's.

Orthon targets *both* humans and LLMs (VISION.md § LLM Readiness; the LLM
Generability Gate). The index base is an **unavoidable Language decision** —
even abstract indexing (Alternative C) must choose a base for `nth(i)`. The
Decision Pipeline returned **ACCEPT** as a Language (Core) decision.

Adopting 1-based **retroactively modifies accepted concepts** that were
written with 0-based assumptions (ITERATION_LOOP EDR-053, ITERATOR_PROTOCOL
EDR-022, SPAN EDR-064) — handled as a documented cross-concept amendment
(C-001), not silent drift.

**Research & pipeline trail:** [`INDEXING_ONE_BASED.md`](../../concepts/research/important/INDEXING_ONE_BASED.md)
(concept research; blockers B1–B4 and advisory B5 resolved 2026-08-05);
[`DECISION_LOG.md`](../../gates/DECISION_LOG.md) § Entry: 1-Based Indexing
(per-gate reasoning).

---

### Decision

1. **Index base = 1.** All built-in collection types index from 1: the first
   element is at index 1, the last at index `len(collection)`. This is a
   *semantic parameter of the `@get` protocol contract* — the language
   commits first element at `@get(1)`, last at `@get(len(a))`.

2. **`a[i]` is a Level 2 language pattern**, not a new primitive:
   `a[i]` ≡ `a@get(i)` — syntactic sugar over a call to the `@get(i)`
   protocol method (Metadata Protocol, `@`-prefix), decomposing to the
   existing `function` + `call` primitives. `.`-prefixed attribute access is
   reserved for user-defined named-member access. Positional access is the
   one-element form of `unpack`, applying to every random-access `pack`
   composite (tuples, strings, `Span`, ranges) via an `Indexable`-like trait.
   Non-random-access representations (`Sequence`, `Set`) do not implement it
   — `a[i]` on them is a compile-time diagnostic.

3. **Range norm: inclusive-inclusive `1..N`** — the *only* range semantic,
   applied uniformly to index access, slices (`items[1..k]` = first k
   elements), and iteration. The language owns the `+1` length arithmetic:
   `len(slice)` returns the element count; the programmer never writes
   `j - i + 1`. An empty slice is a value with `end < start` (e.g.,
   `items[1..0]`), not a syntax error. A half-open `0..<N` exists only as an
   FFI-boundary interop utility; it is never the default and never appears in
   application code.

4. **`enumerate` defaults to 1**, matching the collection base. It is a
   plain Standard Library method on `Iterator[T]` (EDR-022/EDR-032), *not a
   keyword*: `enumerate(items) ≡ zip(1..=len(items), items)`. No start
   parameter — an offset is expressed by an explicit preliminary range
   (`zip(offset..len(items), items)`).

5. **Single-base rule: `Span` is 1-based** like every collection
   (`span[1]` = first). Span is a Language type (EDR-064) and a random-access
   `pack` composite under `@get`. Its FFI role does not create a second base —
   raw C buffers enter through the FFI index-translation layer. The exact
   C-facing constructor surface is deferred to the FFI concept (Milestone 8).

6. **No configurable index base.** Pascal/Ada-style arbitrary lower bounds are
   rejected — they violate Minimal Core and Orthogonality.

7. **Collection Indexing Policy = `OneBased`** in `IMPLEMENTATION_POLICIES.md`.
   The LLM Toolchain schema must encode the 1-based base so generation
   defaults to 1-based.

---

### Consequences

- **Positive:**
  - Eliminates the off-by-one error class at the language level:
    `for i in 1..len(items)` naturally covers all elements; last index == len.
  - Direct translation from mathematics and business notation: $x_1$ is
    `x[1]`; day 1 of the month is `month[1]`.
  - One counting convention for index production (`enumerate`), consumption
    (`@get`), slicing, and iteration — strengthens LLM generability.
  - FFI index translation is localized to a defined, auditable boundary.
- **Negative:**
  - FFI friction: +1/−1 at every C interop boundary (mitigated: the FFI
    boundary already translates memory layout, calling convention, and type
    mapping — index translation is one more rule there).
  - Programmer habit disruption: the majority of programmers are trained on
    0-based (mitigated: Orthon already differs from C-family in ownership,
    expression-orientation, and traits — one more deliberate difference).
  - Modulo arithmetic is less natural for ring buffers (mitigated: stdlib
    `wrap_index`).
  - Retroactive amendment of accepted concepts (C-001) and a documentation /
    educational cost across all examples.

---

### Compliance

- `a[i]` must desugar to `a@get(i)` (Metadata Protocol); no new primitive.
- `@get` contract: first element at `@get(1)`, last at `@get(len(a))`.
- Every range literal is inclusive-inclusive `1..N`; no half-open form in
  application code; `0..<N` only at the FFI boundary.
- `enumerate` base = 1; `enumerate`/`zip` are StdLib methods on `Iterator[T]`.
- `Span` uses the single base (1); raw C buffers translate at the FFI
  boundary.
- Collection Indexing Policy = `OneBased` (IMPLEMENTATION_POLICIES.md).
- LLM Toolchain schema encodes the 1-based base.
- Cross-concept amendments (C-001) applied to ITERATION_LOOP (EDR-053),
  ITERATOR_PROTOCOL (EDR-022), SPAN (EDR-064), and GLOSSARY examples.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| **0-based indexing (C-family)** | Perpetuates the cognitive gap between human counting and machine addressing; leaks the hardware model into a non-systems language. |
| **Configurable index base (Pascal/Ada)** | Violates Minimal Core and Orthogonality — two collections with different bases cannot be indexed uniformly; complexity cost outweighs flexibility. |
| **Abstract indexing (no integer indices)** | Overly restrictive — integer indexing is fundamental for algorithms (binary search, dynamic programming, matrix operations); `nth(i)` still must choose a base. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Concrete benefit: off-by-one elimination, direct domain translation, comfort by construction. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass (Flag → resolved) | No paradox after B2–B4: one base, one range norm, `enumerate` base pinned, Span single-base. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One commitment ("Orthon counts like you do"); no new primitive; no configurable base. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass (Flag → resolved) | Level 2 pattern over `@get`; retroactive amendment documented as C-001 with legitimate dependency direction. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Base is strategy-agnostic — all strategies implement the same mapping. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass (Flag → resolved) | Pre-freeze commitment; reversal risk accepted with documented mitigation (FFI translation layer). |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass (Flag → resolved) | Flag: LLMs are predominantly trained on 0-based code — mitigated by B5-2 (schema encodes the 1-based base). |

**Gates not applied:** none — a new Core Language semantic requires the full
catalogue of 7 gates (`DECISION_VALIDATION.md` § Gate Selection).

**Detailed reasoning:** See `DECISION_LOG.md` § Entry: 1-Based Indexing for
the per-gate reasoning trail and the B1–B5 resolution notes.
