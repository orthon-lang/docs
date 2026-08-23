# Theses — Distilled Understanding Anchors

> A compact, reviewable store of distilled theses about Orthon's language
> semantics. Each thesis is a single, verifiable claim that restores
> understanding of a concept at a glance.
>
> Theses are **not specification**. Canonical truth lives in the linked
> sources (concept docs, EDRs, GLOSSARY). A thesis is a memory anchor
> that points back to that truth.

---

## Format

Each thesis follows this shape:

```markdown
### <Concept> — <short label>

**Status:** DRAFT | VERIFIED | SUPERSEDED | REJECTED
**Source:** <canonical document or EDR>

**Thesis:** <one verifiable claim>

> <optional elaboration, counterpoints, what it does NOT claim>
```

**Verification rule.** A thesis is only as good as its source. A thesis
marked `VERIFIED` has been checked against its canonical source (concept
doc, EDR, GLOSSARY). `DRAFT` theses are unverified and must not be relied
on to restore understanding. Incorrect theses are kept as `REJECTED`
entries — they serve as anti-memory so the error is not repeated.

**Routing.** Once verified, a thesis may be promoted into its canonical
destination (GLOSSARY term, concept doc, or EDR), then marked
`SUPERSEDED` here with a pointer to the new home.

---

## Theses

### T (non-optional) — contract of guarantee

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** A non-optional type `T` is a **contract of guarantee**: a
function returning `T` always produces a value of type `T`, and can
never return `None` — the compiler forbids assigning `None` to a
non-optional `T`.

> Corollary: `T` and `Option<T>` are distinct, incompatible types.
> The `!` operator produces `T` at its usage site (explicit "I know this
> is non-null" escape hatch) — it does not narrow the original variable.

### Option&lt;T&gt; — contract of handling

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** `Option<T>` is a **contract of handling**: a function
returning `Option<T>` *may* produce `None`, and the compiler forces the
caller to handle both `Some(T)` and `None` — pattern matching on
`Option` must be exhaustive.

> Corollary: after a `match`/guard establishes a value is `Some(T)`, the
> compiler narrows the type to `T` in that branch (flow-sensitive
> narrowing, EDR-028).

### None — a distinct type, not assignable to non-optional T

**Status:** VERIFIED
**Source:** [EDR-018](../how/decision_records/architecture/EDR-018-null-safety.md),
[`what/concepts/NULL_SAFETY.md`](concepts/NULL_SAFETY.md)

**Thesis:** `None` is a value of its own distinct type `None`, not of
type `T` and not of type `Option<T>`. Assigning `None` to a non-optional
`T` is a compile-time error; it is assignable only where `Option<T>` is
expected.

> Corollary: `None` is the `None` variant of `Option<T>` — a value of
> type `None` coerces into `Option<T>` because `None` is one of its two
> variants. There is no `null` sentinel that silently inhabits `T`.

### Option with Error — two orthogonal axes, combined by nesting

**Status:** VERIFIED
**Source:** [EDR-020](../how/decision_records/architecture/EDR-020-error-handling.md),
[EDR-023](../how/decision_records/architecture/EDR-023-error-union.md),
[EDR-028](../how/decision_records/architecture/EDR-028-type-level-null-safety.md),
[`what/concepts/ERROR_HANDLING.md`](concepts/ERROR_HANDLING.md) § Interaction with Option,
[`what/concepts/ERROR_UNION.md`](concepts/ERROR_UNION.md)

**Thesis:** `Option<T>` and `Result<T, E>` / Error Union (`!T`) are
**two orthogonal axes**, deliberately kept separate in Orthon: `Option`
answers "is the value present?" (absence), `Result`/`!T` answers "did
the operation fail?" (failure with diagnosis). Combining both concerns
is done by **nesting** (`Result<Option<T>, E>`, `Option<Result<T, E>>`),
not by merging them into a single type.

> Corollary: the two axes have distinct operators — `?.`/`??` on
> `Option` (absence is normal), `?` on `Result`/`!T` (failure must be
> propagated or handled). Orthogonality is why Orthon separates
> narrowable `Option` from non-narrowable `Result` (EDR-028).

### Compile-Time Execution — three orthogonal axes

**Status:** DRAFT
**Source:** [`what/concepts/COMPILE_TIME_EXECUTION.md`](concepts/COMPILE_TIME_EXECUTION.md) § Synthesis,
[EDR-031](../how/decision_records/architecture/EDR-031-compile-time-execution.md),
[EDR-086](../how/decision_records/architecture/EDR-086-generics-syntax-revision.md)

**Thesis:** Compile-time execution is needed — it moves work from the
runtime to the compiler. But "comptime" is not one homogeneous
mechanism: it names three orthogonal things by nature, each with its own
surface form — generics/type inference via `<>` (EDR-086),
metadata/reflection via `@` (Metadata Protocol), and the phase axis
(when code runs) via one explicit phase marker whose granularity
(parameter/block/function) is still open.

> Elaboration: axes 1 and 2 are already settled and never use the word
> "comptime" — `<>` and `@` mark compile-time-ness implicitly. Only the
> phase axis needs an explicit marker, and only for work the other two
> do not cover. The `comptime T: type` parameter form (EDR-031) is where
> the type axis was expressed through the phase axis — the conflation
> that causes the syntax tension. Granularity is not settled: a block is
> the irreducible form (compile-time constants, assertions, local
> metaprogramming); a function marker is sugar whose specialised case
> already exists as `@macro` (EDR-029); a parameter marker narrows to
> reflection helpers once generics live in `<>`.
> What it does NOT claim: it does not decide the granularity (OQ 4), the
> block syntax/terminology (OQ 6), or the phase keyword — those are
> Phase 5 decisions.

### Compile-Time Execution — phase marker `bake` at the call site

**Status:** DRAFT
**Source:** [`what/concepts/COMPILE_TIME_EXECUTION.md`](concepts/COMPILE_TIME_EXECUTION.md) § Phase marker position & keyword (Exploration note 2026-08-23),
[`.planning/notes/2026-08-23-comptime-phase-marker-bake.md`](../.planning/notes/2026-08-23-comptime-phase-marker-bake.md)

**Thesis:** The phase axis's explicit marker is applied at the **call
site**, not the declaration — functions are colourless — and its leading
keyword candidate is **`bake`**: `TABLE = bake generate_table(256)`
evaluates the expression during compilation and embeds the result.

> Elaboration: `bake` was chosen over `consteval` (C++ constexpr/
> consteval baggage) and `comptime`; symbols `#`/`@`/`<>` are blocked by
> Semantic Purity and loaded meanings (`#` = line comment + unhygienic
> macro escape, EDR-029; `@` = Metadata Protocol; `<>` = generics,
> EDR-086). `const TABLE = expr` was rejected — the expression-level
> marker is required for the general case (asserts, args), so a separate
> `const` binding concept is not introduced (resolves
> `DECLARATION_BY_ASSIGNMENT.md` OQ3). Comptime safety (no IO/FS/network)
> is checked transitively at the call site.
> What it does NOT claim: `bake` is not an accepted keyword — marker
> position and keyword are DRAFT candidates feeding Phase 5. EDR-031
> Principle 5 ("marker at the definition site") still stands and must be
> amended via an EDR with Human Sign-off before this thesis can be
> VERIFIED. It does not resolve the block form (OQ 6) or granularity
> (OQ 4).
