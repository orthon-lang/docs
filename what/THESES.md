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

### AST Macros — expander contract, not phase sugar

**Status:** DRAFT
**Source:** [`what/concepts/AST_MACROS.md`](concepts/AST_MACROS.md),
[EDR-029](../how/decision_records/architecture/EDR-029-ast-macros.md),
[`what/concepts/COMPILE_TIME_EXECUTION.md`](concepts/COMPILE_TIME_EXECUTION.md) § Relationship to Existing Mechanisms / § Synthesis,
[`how/gates/DECISION_LOG.md`](../how/gates/DECISION_LOG.md) (EDR-029 pipeline Q5/Q7),
review lock 2026-08-23 (Q3: `@macro` vs `bake` / `@`)

**Thesis:** `@macro` is a **compiler-recognized AST expander contract**,
not syntactic sugar over the phase marker (`bake` / comptime) and not
reducible to Metadata Protocol (`@`). It marks a function whose job is
**AST → AST code generation** (typed AST in, typed AST out, splice into
the program under hygiene and single-pass expansion). The phase engine
runs the body at compile time; `@` may supply structure; neither
substitutes for expansion.

> Elaboration: four duties stay distinct —
> (1) **types/generics** via `<>` (EDR-086);
> (2) **metadata/reflection** via `@` (read structure: `@typeInfo`, …);
> (3) **phase** via an explicit marker at the call site (candidate
> `bake`) for value-level compile-time evaluation;
> (4) **AST codegen/expansion** via `@macro` / `@derive` (declaration-
> driven expand pass: parse → expand → type-check expanded AST).
>
> `bake f(x)` embeds a **value**. A `@macro` / `@derive` invocation
> splices **AST nodes** (`ImplBlock`, `Expr`, …). Decision Log for
> EDR-029: the macro mechanism adds **new semantics** (Q5); only
> `@derive` is sugar over `@macro` (Q7). Typed AST signatures are part
> of the contract, not the whole of it — also required: expansion
> participation, splice, hygiene-by-default, single-pass ordering, and
> post-expansion verification.
>
> What it does NOT claim: it does not accept `bake` as a keyword; it
> does not amend EDR-031 Principle 5; it does not decide the unhygienic
> escape sigil (`#` vs alternatives — open, Q4); it does not require
> call-site `bake` on macro invocations (expansion finds `@macro` /
> `@derive` without a phase keyword). It refines the three-axis comptime
> thesis: `@macro` is not merely “function-level comptime sugar” — it is
> the specialised **codegen/expansion** surface that builds *on* the
> phase engine.

### Traits and ADTs — data and behaviour are separated; both Expression Problem axes stay open

**Status:** VERIFIED
**Source:** [EDR-019](../how/decision_records/architecture/EDR-019-traits.md),
[EDR-039 §4](../how/decision_records/architecture/EDR-039-algebraic-data-types.md),
[`what/concepts/TRAITS.md`](concepts/TRAITS.md),
[`what/concepts/ALGEBRAIC_DATA_TYPES.md`](concepts/ALGEBRAIC_DATA_TYPES.md) § Behaviour,
[`LANGUAGE_PROBLEMS.md`](LANGUAGE_PROBLEMS.md) § The Expression Problem

**Thesis:** Orthon separates **data from behaviour** at the type level:
`type` declares a data shape only (product records, ADT variants), and
behaviour attaches externally — through traits (EDR-019) and separate
`impl` blocks (EDR-039 §4). ADT variants never carry inherent methods.

> Elaboration: two canonical shapes follow. Path A — one closed ADT
> (`type Shape = Circle(...) | Rectangle(...)`) with a single
> `impl Area for Shape` dispatching via `match`; adding a variant breaks
> exhaustiveness (compile-time error). Path B — independent product
> types (`type Circle(radius: Float)`, `type Rectangle(...)`) each with
> its own `impl Area for Circle` / `impl Area for Rectangle`;
> polymorphism via generics (`<Area as T>`, static, default) or `dyn`
> (opt-in vtable). In both paths the `impl` block is external to the
> `type` declaration — the Java/C# class-with-methods model is rejected.
> This is Orthon's resolution of the Expression Problem: Path A
> optimises adding operations, Path B optimises adding types, and Path B
> inverts responsibility for defining operations so both extension axes
> stay open without editing existing code.
> What it does NOT claim: it does not permit methods inside a `type`
> declaration; it does not pick a single mechanism — both paths are
> permitted; trait inheritance stays rejected (EDR-019) — composition
> via bounds `where T as A + B`, not hierarchy.

### Binding keywords — `let` (shadowing) and `var` (mutability)

**Status:** DRAFT
**Source:** [`../how/syntax/REBINDING_SYNTAX.md`](../how/syntax/REBINDING_SYNTAX.md)
(Locked Decisions, 2026-08-19),
[`../how/syntax/SHADOWING_SYNTAX.md`](../how/syntax/SHADOWING_SYNTAX.md),
[`concepts/DECLARATION_BY_ASSIGNMENT.md`](concepts/DECLARATION_BY_ASSIGNMENT.md) (EDR-074)

**Thesis:** `let` marks **shadowing (rebinding)** — a new binding over a
name already in scope; `var` marks **explicit mutability**. The
immutable default needs no keyword (`x = 1`); `val` is rejected as
redundant and misleading.

> Elaboration: the semantic core is EDR-074 — no implicit shadowing
> (any name reuse must be syntactically visible, Principle 5), immutable
> by default, mutation requires an explicit marker. The keyword names
> are Phase 5 syntax candidates, not accepted syntax: `let` is the
> candidate for same-scope rebinding; `var` is a mutability marker, not
> a declaration keyword; `val` was rejected (2026-08-19) — corpus-wide
> `val` means immutable (Kotlin, Scala), which conflicts with LLM
> Generability and is redundant with the keyword-free immutable default.
> What it does NOT claim: it does not settle the Phase 5 keyword names
> (Decision Queue Open, Human sign-off Pending); it does not decide
> whether the same-scope pipeline idiom (`data = parse(data)`) is
> `let`-rebinding or `var`-reassignment (REBINDING_SYNTAX Axis 1 open);
> it does not claim doc convergence — concept-level sources
> (MUTABILITY.md, DECLARATION_BY_ASSIGNMENT.md, GLOSSARY.md) still use
> `mut` for the mutable marker.
