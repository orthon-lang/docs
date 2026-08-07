# EDR-086: Generics Syntax Revision — Angle-Bracket Parameters and `as` Bounds

**Status:** Accepted

**Date:** 2026-08-07

**Category:** Architecture

**Scope:** Platform

---

### Context

EDR-024 adopted trait-bounded parametric polymorphism but committed to a
specific syntax: inline `[T: Trait]` shorthand and `where T: TraitA + TraitB`
clauses. Human review of the GENERICS concept surfaced four problems with
that syntax:

1. **`[ ]` is already taken.** Orthon uses `[T]` for array/slice types and
   `[dyn Trait]` for trait-object arrays (see TRAITS.md, GENERICS.md). A
   generic parameter list `[T: Iterator]` reads as an array type, which
   damages orthogonality and learnability.
2. **`:` is overloaded.** It already denotes parameter type annotation
   (`value: T`), so `[T: Iterator]` and `where T: Trait` use the same
   symbol for two different meanings.
3. **Bound delimiter roles must be explicit.** The `where` clause must
   distinguish bounds on one parameter from constraints on different
   parameters, with grouping that an LLM cannot silently misparse.
4. **The variance example presumed a class hierarchy.** `List[Cat]` vs.
   `List[Animal]` assumes `Cat <: Animal`, but Orthon has no class
   inheritance and traits do not create subtypes. Variance examples must
   be grounded in Orthon's real subtyping sources (unions, literal types,
   widening).

### Decision

Revise the generics syntax and the variance documentation:

1. **Angle-bracket parameters.** Generic type parameters use `<>`:
   `type Pair<T, U>`, `fun identity<T>(T value)`. `[ ]` is freed for
   indexing, array types, and slices.
2. **Bound-first inline shorthand.** A single inline bound is written
   bound-first with `as`: `<Iterator as T>`, read "T is (at least) an
   Iterator" — T may have more capabilities than the bound requires.
3. **`where` clauses via `as`.** Bounds in `where` clauses use `as`:
   `where T as Hash + Eq`. `+` combines multiple bounds on one parameter
   (conjunction — the type must satisfy all of them). `,` separates
   constraints on different parameters: `where T as Hash, U as Ord`.
4. **No method-level shadowing.** A method must not re-declare a type
   parameter of its enclosing type; a duplicate name is a compile error.
5. **Variance documentation.** Variance is computed by position-based
   inference from trait method signatures (unchanged semantics from
   EDR-024). Examples use Orthon's real subtyping sources — unions
   (EDR-045), literal types (EDR-043), and widening — instead of class
   hierarchies.
6. **Declaration kinds and parameter order.** Generics examples use the
   `fun`/`proc`/`new` declaration kinds and type-first parameter syntax
   (`T item`) already established in the Semantic Model / GLOSSARY.
7. **Explicit variance annotations** remain deferred to v0.2+ (unchanged
   from EDR-024).

### Consequences

**Positive:**
- `[ ]` is unambiguous: indexing, arrays, and slices only.
- `:` is no longer overloaded in the bound position; `as` reads as a
  natural-language binding ("Iterator as T").
- `<>` aligns with the comptime model (GLOSSARY: `comptime T: type`
  parameters replace `<T>` syntax) and with type-instantiation syntax.
- Variance examples now reflect the actual type system (unions, literals,
  widening) instead of a non-existent inheritance hierarchy.
- The INVALID (shadowing) case is documented explicitly for LLM
  generability.

**Negative:**
- `<>` requires parser disambiguation from the comparison operators
  `<`/`>`; this is a well-understood problem (Rust, Swift) but adds parser
  complexity.
- The change ripples across every generics example in concepts, GLOSSARY,
  and CORE_CONCEPTS; examples must be migrated.
- Full syntax remains subject to Phase 5 finalisation.

### Compliance

1. No generic example may use `[T: Trait]` or `where T: Trait`; all bounds
   use `<>` with `as`.
2. `+` combines bounds on one parameter; `,` separates parameters — the
   two roles must not be conflated.
3. Method-level re-declaration of a class type parameter is a compile error.
4. Variance examples must be grounded in union, literal, or widening
   subtyping — no class-hierarchy examples.
5. Variance remains deterministically computable from trait method
   signatures (EDR-024, Compliance #4).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep `[T: Trait]` (status quo) | Collides with array/slice/indexing syntax; `:` overloaded with type annotations |
| `where T as Hash & Eq` (`&` for bounds) | `&` is the reference/borrow marker (`&T`, `&mut T`) and is reserved for intersection types (EDR-045) — a third meaning would overload it |
| `where T as Hash, Eq` (`,` for bounds) | Ambiguous with parameter separation: `where T as Hash, U as Ord` cannot be distinguished from two bounds on one parameter |
| Param-first `<T as Iterator>` | Considered; bound-first chosen because it reads as a natural-language contract ("Iterator as T") and keeps the trait name prominent for LLM generability |
| Variance examples using a class hierarchy | Rejected — Orthon has no inheritance; the examples would document non-existent subtyping |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Programmer writes `fun first<Iterator as T>(T items)`; the bound reads as a sentence and `[]` stays reserved for arrays/indexing. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | `+` (bounds on one parameter) and `,` (different parameters) are disjoint roles; `as` is unambiguous vs. `:` annotations; shadowing is an explicit error. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Each symbol has one role: `<>` parameters, `as` binding, `+` conjunction, `,` separation — minimal overload. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | The revision does not change the semantic model of generics (bounds, dispatch, variance); it refines the surface only. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Angle-bracket parsing is strategy-independent; monomorphisation/boxing remain Strategy choices as in EDR-024. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "Parameters in `<>`, bounds via `as`, `+` combines, `,` separates." Conservative and stable. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | The bound reads as natural language ("Iterator as T"); the INVALID shadowing case is documented; `+`/`,` roles are explicit. |

**Gates not applied:** None — the revision touches an accepted language
construct and requires all seven gates.

**Detailed reasoning:** The per-gate reasoning trail is captured in the
Gate Validation table above and in the GENERICS concept review that
produced this revision (2026-08-07).

### Related Concepts

- `what/concepts/GENERICS.md` — Revised specification
- `how/decision_records/architecture/EDR-024-generics.md` — Superseded syntax aspects
- `what/concepts/TRAITS.md` (EDR-019) — Trait system foundation
- `what/concepts/UNION_INTERSECTION_TYPES.md` (EDR-045) — Union subtyping basis for variance examples

### Supersedes

Syntax aspects of [EDR-024](EDR-024-generics.md) — the parameter-delimiter
choice, the inline bound syntax, the `where`-clause syntax, and the basis
for variance examples. The semantic decisions of EDR-024 (trait-bounded
polymorphism, static dispatch by default, invariance by default, no type
erasure) remain in force.
