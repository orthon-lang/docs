# 1-Based Indexing

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-08-05
>
> **🚦 Pipeline status (2026-08-05):** Ran through the Concept Pipeline
> ([`how/CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md), stages 2–7).
> Decision Pipeline → **ACCEPT** (Language/Core). Validation gates →
> **3 Pass / 4 Flag**. Convergence check → **FAIL** — **NOT CONVERGED**:
> blockers B1–B4 must be resolved before an EDR (EDR-082) can be filed.
> **B1 resolved (2026-08-05)** — indexing is a Level 2 pattern over `a@get(i)`;
> B2–B4 remain open. Full reasoning trail:
> [`how/gates/DECISION_LOG.md`](../../../gates/DECISION_LOG.md)
> § Entry: 1-Based Indexing. See Decision History below.
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

Should Orthon collections index their elements starting from 0 or from 1?

Two contradictory traditions exist:

1. **0-based indexing** (C, C++, Java, Python, JavaScript, Rust, Go) — the
   first element is at index 0. Rooted in C's pointer arithmetic: `arr[i]` is
   syntactic sugar for `*(base + i * sizeof(element))`. When `i = 0`, the
   offset is zero — no runtime subtraction needed.

2. **1-based indexing** (Lua, Julia, MATLAB, Fortran, R) — the first element
   is at index 1. Rooted in human and mathematical convention: the first item
   in a sequence is item number 1, not item number 0.

The core problem: **0-based indexing creates a persistent cognitive gap
between how humans count (1, 2, 3, …) and how machines address memory
(offset 0, offset 1, …).** This gap is the source of off-by-one errors —
one of the most common bug categories across all programming languages —
and forces domain experts (mathematicians, scientists, business analysts)
to mentally translate between their natural notation and the machine's.

Orthon's vision explicitly targets *both* human and LLM audiences. The
question is whether Orthon should perpetuate the C-legacy default or
decide in favor of human-natural indexing, as several successful
languages (Lua, Julia) have done before.

## Principles

1. **Principle of Least Astonishment (POLA)** — The language should behave
   as a reasonable person expects. A non-programmer expects `items[1]` to
   return the first item. See [`DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md)
   § POLA.
2. **Intent Over Implementation** — The programmer describes *what* should
   happen, not *how* memory is laid out. Indexing is a logical operation,
   not address arithmetic. See [`DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md)
   § Intent Over Implementation.
3. **Data First** — Indexing operates on Data (the primary abstraction), not
   on raw memory. The abstraction should match the domain, not the hardware.
4. **Minimal Core** — Do not introduce configurable index bases (as in
   Pascal/Ada) — that adds language complexity for marginal benefit.

## Examples

### Languages with 1-based indexing

| Language | Domain | Rationale for 1-based |
|----------|--------|----------------------|
| **Lua** | Embedding, scripting | Target audience: non-professional programmers. Arrays (tables) start at 1 because that is what the users expect. |
| **Julia** | Scientific computing | Target audience: scientists and mathematicians. Formulas copy directly from paper to code without index translation. |
| **MATLAB** | Numerical computing | Mathematical convention: vectors are $x_1, x_2, \ldots, x_n$. |
| **Fortran** | Scientific/HPC | Scientific computing tradition: arrays model mathematical vectors. |
| **R** | Statistics | Statistical convention: data frames, observations numbered from 1. |

### Languages with configurable index bases

| Language | Mechanism | Notes |
|----------|-----------|-------|
| **Pascal** | `array [5..10] of Integer` | Arbitrary lower bound at declaration site |
| **Ada** | `type Vector is array (1..N) of Float` | Arbitrary lower bound, strongly typed |

These are instructive but violate Orthon's Minimal Core principle —
configurable index bases add language complexity for marginal benefit.

### 0-based indexing in domain code: the adaptation tax

Consider a mathematical formula translated to 0-based code:

```python
# Mathematical: sum of first k elements of vector x₁, x₂, …, xₙ
# 0-based version — mental translation required
total = sum(x[0:k])  # "k elements" = x[0] through x[k-1]; x[k] excluded
```

```julia
# 1-based version — direct translation from paper
total = sum(x[1:k])  # "first k elements" = x[1] through x[k] inclusive
```

The 0-based version requires the programmer to remember that the $k$-th
element is at index $k-1$ and that the slice `[0:k)` excludes the endpoint.
The 1-based version reads exactly as the formula states.

### Off-by-one errors: the persistent cost

Off-by-one errors (OBOE) are among the most common bugs in programming.
Studies consistently rank them in the top error categories across languages.
Examples:

```python
# 0-based: classic off-by-one
for i in range(len(items)):
    items[i+1]  # IndexError on last iteration
```

```julia
# 1-based: the equivalent loop is natural
for i in 1:length(items)
    items[i]  # last index == length(items)
```

When `len(items) == 5`, the 0-based programmer must remember that valid
indices are `0..4`, not `0..5`. The 1-based programmer knows that valid
indices are `1..5` — the last index equals the length.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Collection Indexing Policy | Determines whether collection indices start at 0 or 1 |
| FFI Boundary Policy | Governs index translation at language boundaries (C interop) |
| Range Semantics Policy | Determines whether range literals are inclusive-inclusive or half-open |

## Implications for Orthon

### Proposal: 1-based indexing as the default

Orthon should adopt 1-based indexing for all built-in collection types.
The first element of a sequence is at index 1. The last element is at
index `len(sequence)`.

**Canonical forms:**

```orthon
items[1]           -- first element
items[len(items)]  -- last element
items[1..k]        -- first k elements (inclusive-inclusive)
```

**len semantics:** `len(sequence)` returns the number of elements. With
1-based indexing, the last valid index equals `len(sequence)` — a natural
and intuitive property.

**Range literals:** `1..N` is inclusive-inclusive, producing N elements.
An alternative half-open form `0..<N` (producing N elements starting at 0)
is available for FFI interop contexts but is not the default.

**enumerate semantics:** `items.enumerate()` produces pairs `(1, first)`,
`(2, second)`, `(3, third)`, … — matching the natural counting that
`enumerate` implies.

### Impact on Semantic Model

The six semantic dimensions defined in
[`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) (EDR-013) are
affected as follows:

| Dimension | Impact | Explanation |
|-----------|--------|-------------|
| **Identity** | Medium | Indexing defines *how* an element is identified within a collection. 1-based identifies elements by ordinal position (first, second, …); 0-based identifies by memory offset (0 bytes forward, 1×size bytes forward, …). The semantic model's Identity dimension does not prescribe a base, but 1-based aligns with the principle that identity is logical, not physical. |
| **Evaluation** | Medium | Indexing is an evaluation operation — `items[i]` evaluates to the $i$-th element. The Evaluation dimension guarantees that this operation is deterministic and side-effect-free. The choice of base affects which index values are valid (1..N vs. 0..N-1) but does not change the evaluation model. |
| **Ownership** | None | Indexing reads a reference to an owned element; the base does not affect ownership semantics. |
| **Mutation** | None | Mutation through indexed access (`items[i] = value`) requires exclusive access per the semantic invariants; the base does not affect this. |
| **Visibility** | None | Indexing does not cross visibility boundaries. |
| **Lifetime** | None | Indexing does not affect lifetime semantics. |

### Impact on Primitive Blocks

The primitive set defined in
[`PRIMITIVE_BLOCKS.md`](../../../what/PRIMITIVE_BLOCKS.md) (EDR-016) is
**unchanged** — no new primitive is introduced. Indexing is specified as a
**Level 2 Language Pattern** (resolved 2026-08-05, blocker B1):

- **Desugaring:** `a[i]` ≡ `a@get(i)` — syntactic sugar over a call to the
  `@get(i)` protocol method, decomposing to the existing `function` + `call`
  primitives. This follows the Metadata Protocol pattern (`@`-prefixed,
  language-recognized protocol access, as with `list@len()`, `obj@fields`,
  `collection@iter()`) — *not* the `.`-prefixed `attribute access` primitive,
  which is reserved for named-member access.
- **`identifier`** and **`literal`** — index values are literals or
  computed expressions.

**Composition formula:** `a[i]` → `a@get(i)` → `call(function(@get), a, i)`.
The choice of 0 vs. 1 is a *semantic parameter of the `@get` protocol
contract*: the language commits that the first element is at `@get(1)` and
the last at `@get(len(a))`.

**Scope of positional access:** `@get(i)` is not collection-specific. As the
one-element form of `unpack` (positional extraction), it applies to every
random-access composite produced by `pack` — tuples, strings, `Span`, and
ranges. Random-access capability is declared via an `Indexable`-like trait, so
non-random-access representations (`Sequence`, `Set`) simply do not implement
it, and `a[i]` on them is a compile-time diagnostic, not a runtime failure.

### Layer Classification

Per the classification criteria in
[`LIBRARY_BOUNDARY.md`](../../../what/LIBRARY_BOUNDARY.md):

| Level | Scope | Rationale |
|-------|-------|-----------|
| **Core Language** | Indexing syntax (`a[i]`), range literal semantics (`1..N`), `len` semantics | The indexing base is a language-level decision — it affects the meaning of every `a[i]` expression and every `for` loop. It cannot be changed by a library. |
| **Standard Library** | `enumerate()`, range types, collection APIs, modulo-arithmetic utilities | Collection iteration and utility functions adapt to the chosen base but do not define it. |
| **FFI Boundary** | Explicit index translation layer for C interop | Every call across the C FFI boundary must translate indices: Orthon 1..N → C 0..N-1 on the way out, C 0..N-1 → Orthon 1..N on the way in. This is a defined, auditable boundary, not an ad-hoc convention. |
| **Not framework** | N/A | Indexing base is too fundamental for framework-level control. |

The indexing base is a **Language** category decision — it is a semantic
parameter of the `@get` protocol contract, not something a library could
change. The `a[i]` mechanism itself is Level 2 sugar over `@get(i)`; the base
is the Core-language commitment on that protocol.

## Tradeoffs

### Advantages of 1-based indexing

1. **Cognitive alignment with human counting.** The first element is at
   index 1. The last element is at index `len`. No mental translation.
   This serves Orthon's goal of being "comfortable by construction"
   ([`VISION.md`](../../../why/VISION.md)).

2. **Direct translation from mathematics and business domains.**
   $x_1$ in a paper is `x[1]` in code. Day 1 of the month is `month[1]`.
   Domain experts write what they mean.

3. **Elimination of off-by-one errors at the language level.**
   `for i in 1..len(items)` naturally iterates over all elements.
   No `range(len(items))` + `items[i]` pattern with its off-by-one risk.

4. **LLM Generability.** LLMs are trained on both 0-based and 1-based
   code (Lua, Julia, MATLAB, R). However, 1-based code has fewer
   off-by-one patterns for the LLM to track. See
   [`how/gates/DECISION_VALIDATION.md`](../../gates/DECISION_VALIDATION.md)
   § LLM_GENERABILITY_GATE.

5. **Alignment with Orthon's design philosophy.** Orthon is not a
   systems language. It does not expose raw memory. Indexing is a
   logical operation on Data — the C-legacy of pointer arithmetic
   has no place in Orthon's abstraction model.

### Disadvantages of 1-based indexing

1. **FFI friction.** Every interaction with C libraries, system APIs,
   and network protocols requires +1/−1 translation at the boundary.
   This is a defined, auditable boundary (see FFI Boundary above) but
   it is a real cost that every FFI consumer pays.

   *Mitigation:* The FFI boundary is already a translation layer
   (memory layout, calling convention, type mapping). Index translation
   is one more rule in an already-non-trivial boundary. Julia
   demonstrates that this is manageable for a language targeting
   non-systems domains.

2. **Programmer habit disruption.** The majority of programmers are
   trained on C-family languages with 0-based indexing. Switching to
   1-based requires unlearning.

   *Mitigation:* Orthon already differs from C-family languages in
   fundamental ways (ownership model, expression-orientation, trait
   system). The indexing base is one more deliberate difference, not
   an isolated oddity.

3. **Modulo arithmetic complexity.** Ring buffers and circular queues
   with 1-based indexing require `((i - 1) % N) + 1` instead of
   `i % N`.

   *Mitigation:* The standard library provides `wrap_index(i, N)` or
   similar. Modulo-heavy code is rare outside systems programming and
   specialized data structures; when it appears, the library mitigates
   the ergonomic cost.

4. **Documentation and educational cost.** Every tutorial and reference
   must explicitly explain the 1-based choice and why it differs from
   the C-family norm.

   *Mitigation:* This is a one-time cost per learner. The explanation
   is simple: "Orthon counts like you do."

### Summary

| Criterion | 0-based | 1-based | Orthon fit |
|-----------|---------|---------|------------|
| Human cognitive alignment | Weak | Strong | ✅ 1-based |
| Mathematical/domain translation | Requires adaptation | Direct | ✅ 1-based |
| Off-by-one error surface | Large | Minimal | ✅ 1-based |
| C FFI ergonomics | Zero-cost | Translation required | ⚠️ Tradeoff accepted |
| Modulo arithmetic | Natural | Awkward | ⚠️ Mitigated by stdlib |
| Programmer familiarity | Majority default | Minority | ⚠️ Tradeoff accepted |
| LLM generability | High (trained on both) | High (trained on both) | ≈ Equal |
| Abstraction purity | Leaks hardware model | Matches logical model | ✅ 1-based |

## Related Concepts

- **ITERATION_LOOP** ([`ITERATION_LOOP.md`](../important/ITERATION_LOOP.md),
  EDR-053) — The `for item in sequence` loop. Currently uses `0..len(array)`
  for index-based iteration in its GLOSSARY entry. If 1-based indexing is
  adopted, the canonical index-based form becomes `1..len(array)`.

- **ITERATOR_PROTOCOL**
  ([`ITERATOR_PROTOCOL.md`](../../research/essential/ITERATOR_PROTOCOL.md),
  EDR-022) — `.enumerate()` semantics. With 1-based indexing, `enumerate`
  produces `(1, first), (2, second), …`.

- **COMPOSABLE_COLLECTION_OPS**
  ([`COMPOSABLE_COLLECTION_OPS.md`](../../research/essential/COMPOSABLE_COLLECTION_OPS.md),
  EDR-023) — `map`/`filter`/`reduce` on collections. Indexing base does not
  affect these operations directly, but slice semantics (`items[1..k]`)
  interact with the range literal design.

- **RANGE** (no separate research document yet) — Range literal syntax and
  semantics. With 1-based indexing, the natural default is inclusive-inclusive
  (`1..N` produces N elements). A half-open form (`0..<N`) is available for
  interop.

- **FFI** (Milestone 8, [`ROADMAP.md`](../../../when/ROADMAP.md) § M8) —
  Foreign Function Interface. Index translation layer required at the
  C interop boundary.

## Alternative Approaches

### Alternative A: 0-based indexing (C-family default)

The path of least resistance: follow the C tradition. All index-based
code looks familiar to programmers from C, C++, Java, Python, JavaScript,
Rust, and Go.

**Rejection rationale:** Perpetuates the cognitive gap between human
counting and machine addressing. Orthon's design pillars (Data First,
Intent Over Implementation, comfort by construction) argue against
preserving a hardware abstraction in a language that is not a systems
language.

### Alternative B: Configurable index base (Pascal/Ada model)

Allow the programmer to declare the index base per collection:
`array [1..N]` vs. `array [0..N-1]`.

**Rejection rationale:** Violates Minimal Core and Orthogonality.
Every function that accepts a collection must handle arbitrary index
bases or assert a specific one. Two collections with different bases
cannot be indexed uniformly. The complexity cost outweighs the
flexibility benefit.

### Alternative C: Abstract indexing (no exposed integer indices)

Iteration only through iterators and `enumerate`; no direct integer
indexing. Elements are accessed via `first`, `last`, `nth(i)` methods.

**Rejection rationale:** Overly restrictive. Integer indexing is a
fundamental operation for algorithms (binary search, dynamic programming,
matrix operations). Removing it makes the language less expressive
without eliminating the underlying design question — `nth(i)` still
needs to decide whether `i` starts at 0 or 1.

## Open Questions

1. **Range syntax details (Phase 5).** Should the default range literal be
   `1..N` (inclusive-inclusive) or `1..<N+1` (half-open)? The semantic
   commitment (1-based) is this document's concern; the concrete syntax
   is deferred to Phase 5 (Syntax Design).

2. **`enumerate` default.** Should `enumerate()` start at 1 by default, or
   should it accept an optional start parameter (`enumerate(from: 0)`)?
   Julia's `enumerate` starts at 1; Python's starts at 0 with an optional
   `start` parameter.

3. **Modulo arithmetic in the standard library.** What utilities should the
   standard library provide for index wrapping (`wrap_index`), ring buffer
   indexing, and similar patterns that are less natural with 1-based
   indexing?

4. **FFI index translation.** Should the FFI boundary perform automatic
   index translation for known collection types, or should it require
   explicit `to_c_index` / `from_c_index` calls? Julia leaves this to
   the programmer; Orthon could provide a more structured approach.

5. **Interaction with `SPAN`.** The `SPAN` concept
   ([`SPAN.md`](../important/SPAN.md)) defines a lifetime-tracked view
   over memory. Does `Span` use 1-based or 0-based indexing? If `Span`
   is primarily an FFI/interop type, 0-based may be more natural for
   that specific type.

6. **GLOSSARY and existing examples.** The current
   [`GLOSSARY.md`](../../../what/GLOSSARY.md) entry for `for` loop uses
   `0..len(array)` as the example. If 1-based indexing is adopted, all
   existing code examples in documentation must be audited and updated.

## Decision History

**2026-08-05 — Pipeline run (Concept Pipeline, stages 2–7).**
Ran through the full pipeline per [`how/CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md).
Detailed reasoning trail recorded in
[`how/gates/DECISION_LOG.md`](../../../gates/DECISION_LOG.md) § Entry: 1-Based Indexing.

- **Decision Pipeline (10 Qs):** **ACCEPT** as a Language (Core) decision. The
  index base is unavoidable (even abstract indexing must choose a base for
  `nth(i)`) and is not a library concern.
- **Primitive Decomposition Check: FLAG (B1).** `a[i]` is *not* covered by the
  existing `attribute access` primitive (`PRIMITIVE_BLOCKS.md` § 3.2.4 defines
  it as named-member access via `.`). Must be re-specified as a Level 2 pattern
  over `nth(i)`/`get(i)` (recommended) or a new Level 1 primitive.
- **Layer Classification:** Core Language — **Language** category
  (`LIBRARY_BOUNDARY.md`). Range, `enumerate`, and FFI translation belong to
  their own concepts (RANGE, ITERATOR_PROTOCOL, FFI).
- **Validation gates:** `USER_VALUE_GATE` Pass · `LOGICAL_CONSISTENCY_GATE`
  Flag · `CONCEPTUAL_SIMPLICITY_GATE` Pass · `ARCHITECTURAL_INTEGRITY_GATE`
  Flag · `IMPLEMENTATION_INDEPENDENCE_GATE` Pass · `LONG_TERM_MAINTAINABILITY_GATE`
  Flag · `LLM_GENERABILITY_GATE` Pass.
- **Convergence check: FAIL** — not ready for EDR-082.

**Verdict: NOT CONVERGED.** Blockers before EDR-082:
- **B1** — ✅ **RESOLVED (2026-08-05).** `a[i]` is a Level 2 pattern over the
  `@get(i)` protocol method (Metadata Protocol, `@`-prefix), decomposing to
  `function` + `call`; no new primitive. Positional access generalizes to all
  random-access `pack` composites (tuples, strings, `Span`, ranges) via an
  `Indexable`-like trait. The 1-based base is a semantic parameter of the
  `@get` contract. See the corrected «Impact on Primitive Blocks» section.
- **B2** — open: SPAN interaction (single-base rule vs. two-base language).
- **B3** — open: `enumerate` default start (1, matching the collection base).
- **B4** — open: retroactive amendment to ITERATION_LOOP (EDR-053) and
  ITERATOR_PROTOCOL (EDR-022) range/enumerate conventions.

Advisory (B5): add Collection Indexing Policy to `IMPLEMENTATION_POLICIES.md`,
consider an LLM Toolchain requirement for the 1-based base in the schema, and
plan the GLOSSARY/examples audit.

No decision accepted yet — the B1 resolution is a design refinement, not the
acceptance EDR.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md` (Evaluation dimension clarification)
- [ ] `what/PRIMITIVE_BLOCKS.md` (attribute access semantics)
- [ ] `what/GLOSSARY.md` (update `for` loop example, add indexing entry)
- [ ] `what/SYNTAX.md`
- [ ] `what/LIBRARY_BOUNDARY.md` (add INDEXING entry under Language)
- [ ] `ITERATION_LOOP.md`
- [ ] `ITERATOR_PROTOCOL.md` (enumerate semantics)
- [ ] `COMPOSABLE_COLLECTION_OPS.md`
- [ ] `SPAN.md` (indexing base for memory views)
- [ ] `how/architecture/ARCHITECTURE.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
