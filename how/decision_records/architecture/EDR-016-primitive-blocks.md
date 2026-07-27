# EDR-016: Primitive Blocks — Minimal Orthogonal Set of Primitive Operations

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Phase 2 (EDR-013) established the six Semantic Dimensions — Identity,
Ownership, Mutation, Evaluation, Visibility, Lifetime — as Orthon's
Core-Language semantic contract. Phase 3 identifies the **atomic
operations (primitives)** that serve those dimensions — the irreducible
operations from which all derived features (Phase 4) are composed.

Ten essential-tier research documents existed as raw material for the
primitive set synthesis, each independently exploring one facet of what
primitives Orthon needs:

| # | Source Document | Disposition |
|---|-----------------|-------------|
| 1 | `FOUNDATIONAL_ABSTRACTIONS.md` | **Modified** — Data/Data Modifiers hypothesis adopted as organising taxonomy (D-02), not as the primitive set itself |
| 2 | `EXCLUSIVE_DECLARATIONS.md` | **Modified** — fun/proc/new adopted as tags on the `function` primitive (D-04), not separate primitives |
| 3 | `STRUCT_AS_VALUE_TYPE.md` | **Superseded** — struct is not a primitive; built from `pack` + `identifier` + `scope` (D-03) |
| 4 | `CLASS_WITH_ACT.md` | **Superseded** — class is not a primitive; built from `pack` + `reference` + `scope` (D-03); act decomposes to function tag + `reference` + `scope` |
| 5 | `ACT_AS_FUNCTION.md` | **Superseded** — historical; superseded by `DELEGATE.md` before this synthesis |
| 6 | `FUNCTIONS.md` | **Modified** — first-class function model adopted; `function` + `call` as separate primitives (D-04); explicit closure capture confirmed |
| 7 | `FINAL_BY_DEFAULT.md` | **Not directly applicable** — governs inheritance policy, not primitive-level; orthogonal to primitive set |
| 8 | `NAMESPACES.md` | **Superseded** — namespace is not a primitive; built from `identifier` + `scope` + visibility (D-05) |
| 9 | `DELEGATE.md` | **Superseded** — delegate is not a primitive; built from `reference` + `function` + ownership (D-05); execution is orthogonal to declaration |
| 10 | `COMPOSITION_OVER_INHERITANCE.md` | **Not directly applicable** — design pattern principle, not primitive-level; composition confirmed as primary mechanism |

These documents were synthesised into the complete primitive set
specification at `what/PRIMITIVE_BLOCKS.md` (Plan 03-01), replacing its
earlier DRAFT placeholder. The resulting set was then verified against
every concept research file across all four tiers (~132 files) to
confirm completeness and minimality (Plan 03-02).

The following open items from Phase 2 (SEMANTIC_MODEL.md § Mutation,
"Deferred to Phase 3") were resolved:

- **Interior mutability (Cell/RefCell):** NOT a primitive — a derived
  Standard Library feature built on `reference` + mutation semantics.
- **Mutation in closures:** Closures capture variables as immutable by
  default. Explicit `mut` on the captured binding is required for
  mutable capture.
- **`mut` vs `&mut`:** One keyword (`mut`) serves both binding-level
  and reference-level mutation marking.

### Decision

Adopt the **9-primitive set** as Orthon's Level 1 (Primitive Operations)
in the Semantic Dependency Architecture, specified in
`what/PRIMITIVE_BLOCKS.md`:

**Data Primitives (3):**
1. **`literal`** — Inline value notation from source text.
2. **`identifier`** — Named reference to a value; binding point for ownership.
3. **`pack`/`unpack`** — Symmetric composition/decomposition pair (one primitive, two operations).

**Data Modifier Primitives (6):**
4. **`assignment`** — Bind a value to an identifier; creates or updates a binding.
5. **`function`** — Parameterized computation declaration. Three declaration kinds (`fun`/`proc`/`new`) are tags on this primitive.
6. **`call`** — Invocation of a declared function; triggers evaluation.
7. **`attribute access`** — Access a member of a composite value (`.` syntax).
8. **`scope`** — Lexical boundary for names and lifetimes.
9. **`reference`** — Indirection to a value without ownership transfer; two modes (shared `&T`, exclusive `&mut T`).

This replaces the earlier 11-item hypothesis by:
- Removing `operator definition` (syntactic sugar for `function`)
- Folding `pack`/`unpack` into one symmetric pair
- Decomposing `struct`, `class`, `delegate`, `namespace` as compositions
- Treating `fun`/`proc`/`new` as function tags, not separate primitives

### Consequences

**Positive:**
- The 9-primitive set is verified complete — every concept in the ~132-file
  research catalog decomposes onto these primitives
- The set is minimal — removing any primitive makes at least one known
  feature inexpressible
- Phase 4 (Derived Features) has a settled foundation: every concept must
  decompose onto these 9 primitives, and any concept that cannot
  decomposes signals an incomplete set that must be revisited
- Excluded concepts (`operator definition`, `struct`, `class`, `delegate`,
  `namespace`) have clear decomposition paths documented in
  `what/PRIMITIVE_BLOCKS.md`
- D-10 open items resolved for the primitive set, removing ambiguity for
  Phase 4 planning

**Negative:**
- None identified.

### Compliance

1. Every Phase 4 concept must have a decomposition path onto these 9
   primitives (per PRIM-02 verification).
2. If a Phase 4 concept cannot decompose, the primitive set must be
   revisited before Phase 4 completes.
3. The verification section in `what/PRIMITIVE_BLOCKS.md` (§ 10) serves as
   the compliance record for the initial verification.
4. New concepts added after Phase 4 must pass the same decomposition
   check as part of the Concept Design Review.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Two-primitive Data/Data Modifiers model (from `FOUNDATIONAL_ABSTRACTIONS.md`) | Too coarse for Phase 4 decomposition verification — a `for` loop and a function call would both be "Data Modifiers" (D-02) |
| 11-item hypothesis (original set including `operator definition`) | `operator definition` rejected as syntactic sugar per DESIGN_PRINCIPLES.md § Named Before Symbolic (D-01) |
| Including `struct`/`class` as primitives | Type constructors are compositions of simpler primitives (D-03); would mix abstraction levels |
| Including `delegate` as primitive | Execution policy, not an atomic operation (D-05) |
| Including `namespace` as primitive | Organisational convenience, not irreducible (D-05) |
| Not separating `function` and `call` | Serve different semantic dimensions (declaration vs. invocation) — merging would obscure the eval/lifetime split (D-04) |
| Including interior mutability as primitive | Derived Standard Library feature; not irreducible (D-10) |

### Related Concepts

- `what/PRIMITIVE_BLOCKS.md` — Full specification of the primitive set
- `what/SEMANTIC_MODEL.md` (EDR-013) — Six Semantic Dimensions each primitive serves
- `how/DESIGN_PRINCIPLES.md` — Constitutional design rules governing the set
- `how/architecture/ARCHITECTURE.md` § Semantic Dependency Architecture — Level 1 position
- `what/GLOSSARY.md` — Terminology (Primitive Block, Data Primitive, Data Modifier Primitive)

### Supersedes

*None* — this is a new decision, not a replacement.
