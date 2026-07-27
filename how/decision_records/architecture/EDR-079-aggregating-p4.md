# EDR-079: Phase 4 — Derived Features & Decision Pipeline

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Project

**Supersedes:** *None*

---

### Context

Phase 4 processed all outstanding language feature concepts through a
rigorous pipeline: Decision Pipeline → Primitive Decomposition →
Classification → Concept Design Review → Validation Gates → EDR. This
EDR records the global acceptance of all processed concepts and their
interactions.

The Phase 4 scope covered:

- **22 essential-tier concepts** (Wave 1/2) — the semantic bedrock of
  Orthon's v0.1 specification
- **36 important-tier concepts** (Wave 4/5/6) — ergonomics,
  expressiveness, and safety guarantees
- **4 rejected concepts** (Wave 7) — formally evaluated and rejected
  via EDR-075 through EDR-078
- **50 deferrable-tier concepts** — triaged to v0.2/v0.3 or
  post-Freeze, documented with deferral rationale
- **3 policy-level concepts** — routed to `how/strategies/` per D-04
- **2 borderline corrections** — CONTEXT_PARAMETERS (→ SEMANTIC_MODEL)
  and REPRESENTATION_MODIFIERS (→ PRIMITIVE_BLOCKS)

~112 concept research files across all four tiers were processed,
triaged, classified, and either accepted, rejected, or deferred.

---

### Decision

The following concepts are accepted as part of Orthon v0.1:

#### Essential Tier — Language (16 concepts)

| # | Concept | EDR | Key Decision |
|---|---------|-----|--------------|
| 1 | EQUALITY | EDR-017 | Three-operator model: `===` (value), `==` (semantic), `is` (identity) |
| 2 | NULL_SAFETY | EDR-018 | `Option<T>` sum type; no `null` sentinel; `?.`/`??`/`!` operators |
| 3 | TRAITS | EDR-019 | Nominal trait system; static dispatch by default; `dyn` for dynamic |
| 4 | ERROR_HANDLING | EDR-020 | `Result<T,E>` monadic type; `?` propagation; no exceptions |
| 5 | LAZY_SEQUENCE_GENERATORS | EDR-021 | `emit` keyword; lazy by default (D-06); state machine compilation |
| 6 | ITERATOR_PROTOCOL | EDR-022 | `Iterator[T]` trait; `for` loop desugaring; combinators as StdLib |
| 7 | ERROR_UNION | EDR-023 | `!T` tag-only error union; inferred error sets; structural widening |
| 8 | GENERICS | EDR-024 | Trait-bounded parametric polymorphism; monomorphisation; cross-ref comptime |
| 9 | PATTERN_MATCHING | EDR-025 | Exhaustive, expression-oriented; destructuring; guards; or patterns |
| 10 | PATTERN_MATCHING_DISPATCH | EDR-026 | Multimethod dispatch via definition-site `match` declaration |
| 11 | TYPE_INFERENCE | EDR-027 | Local bidirectional inference; explicit annotations at API boundaries |
| 12 | TYPE_LEVEL_NULL_SAFETY | EDR-028 | Flow-sensitive `Option<T>` narrowing after match/if checks |
| 13 | AST_MACROS | EDR-029 | AST macros as comptime functions; `@derive` sugar; hygienic |
| 14 | COMPILER_AS_STATIC_ANALYZER | EDR-030 | Seven verification layers; compiler IS the analyzer |
| 15 | COMPILE_TIME_EXECUTION | EDR-031 | Unified comptime; same semantics earlier phase; Zig-inspired |
| 16 | CONCURRENCY_MODEL | EDR-033 | Delegate-based; `act` modifier; `<-` message operator; data-race freedom |

#### Essential Tier — Standard Library (1 concept)

| # | Concept | EDR | Key Decision |
|---|---------|-----|--------------|
| 17 | COMPOSABLE_COLLECTION_OPS | EDR-032 | map/filter/reduce on `Iterator[T]`; lazy; StdLib classification |

#### Essential Tier — Policy (3 concepts)

| # | Concept | EDR | Routing |
|---|---------|-----|---------|
| 18 | ALLOCATION | EDR-034 | `how/strategies/` — Allocation Policy |
| 19 | REGION_BASED_MEMORY_MANAGEMENT | EDR-035 | `how/strategies/` — Arena sub-policy |
| 20 | EXECUTION_PROGRAM | EDR-036 | `how/strategies/` — Execution Model Policy |

#### Essential Tier — Corrections (2 concepts)

| # | Concept | EDR | Correction Target |
|---|---------|-----|------------------|
| 21 | CONTEXT_PARAMETERS | EDR-037 | SEMANTIC_MODEL (Evaluation/Visibility dimensions) |
| 22 | REPRESENTATION_MODIFIERS | EDR-038 | PRIMITIVE_BLOCKS (pack/reference annotations) |

#### Important Tier — Language (19 concepts)

| # | Concept | EDR | Key Decision |
|---|---------|-----|--------------|
| 23 | ALGEBRAIC_DATA_TYPES | EDR-039 | Combined sum/product; subsumes enum construct |
| 24 | LITERAL_TYPES | EDR-043 | Values as singleton types; `let` preserves, `var` widens |
| 25 | STRUCTURAL_TYPING | EDR-044 | Structural trait satisfaction via `structural` keyword; nominal-by-default |
| 26 | UNION_INTERSECTION_TYPES | EDR-045 | `A | B` structural union; intersection NOT accepted for v0.1 |
| 27 | TYPE_LEVEL_COMPUTATION | EDR-046 | Closed set of 8 non-recursive intrinsics; no user type-level language |
| 28 | ASYNC_AWAIT | EDR-047 | Async as orthogonal modifier; stackless coroutines; colourless model |
| 29 | GENERATORS | EDR-050 | Bidirectional `yield`; generator expressions; `yield from` delegation |
| 30 | EMIT_AS_INTERMEDIATE_RESULT | EDR-052 | Dual purpose of `emit`: lazy sequence + intermediate result; `.final()` |
| 31 | ITERATION_LOOP | EDR-053 | `for`/`while`/`loop`; `for` desugars to Iterator protocol; no C-style |
| 32 | UNPACKING | EDR-055 | Destructuring assignment matching pack/unpack symmetry |
| 33 | CONTRACTS | EDR-056 | `requires`/`ensures`/`invariant`; `result`/`old`; release-build elision |
| 34 | EXTENSION_FUNCTIONS | EDR-058 | Method-call syntax on external types; static dispatch; explicit import |
| 35 | GRADUAL_TYPING | EDR-059 | Optional annotations; boundary checks; critical for LLM adoption |
| 36 | SMART_CAST | EDR-060 | Flow-sensitive narrowing after `is`/`isnt` checks; immutable prereq |
| 37 | SLOTS | EDR-063 | Fixed-field storage default; `dynamic` modifier for opt-out |
| 38 | SPAN | EDR-064 | Lifetime-tracked, bounds-checked non-owning memory view |
| 39 | CONTEXT_LIMITED_MODULES | EDR-072 | Module system with effect isolation; context budget diagnostic |
| 40 | DECLARATION_BY_ASSIGNMENT | EDR-074 | First assignment creates variable; `let` for shadowing; read-before-write error |
| 41 | COMMAND_PATTERN_VIA_DELEGATE | EDR-071 | Existing delegate model subsumes Command pattern (documentation confirmation) |

#### Important Tier — Standard Library (15 concepts)

| # | Concept | EDR | Key Decision |
|---|---------|-----|--------------|
| 42 | CONCURRENCY | EDR-049 | Channels, select, supervision, timers — StdLib on delegate model |
| 43 | PUSH_STREAMS | EDR-051 | Observable-style reactive streams on delegate + channel |
| 44 | OBJECT_INITIALIZATION | EDR-054 | Named params, builders, copy-with-modify — all StdLib/macro |
| 45 | COLLECTION_LITERAL_SYNTAX | EDR-041 | `[1,2,3]` desugars to `List(1,2,3)`; syntax deferred to Phase 5 |
| 46 | DATACLASSES | EDR-042 | `@derive(init, eq, repr, hash)` via existing macro system |
| 47 | DELEGATION | EDR-057 | `@delegate` macro for class delegation; property delegation protocols |
| 48 | NAMED_AND_OPTIONAL_PARAMETERS | EDR-065 | Sugar over function call model; macro-based desugaring |
| 49 | SORTING | EDR-066 | Stable sort default (Timsort); algorithm selection is Policy |
| 50 | DECLARATIVE_MULTI_KEY_SORT | EDR-067 | Key-path sort sugar over `Ord` trait comparisons |
| 51 | IMMUTABLE_DATE_TIME | EDR-068 | Value-semantics date/time types; immutable by construction |
| 52 | PERSISTENT_DATA_STRUCTURES | EDR-069 | `Immutable` trait accepted v0.1; collections deferred to v0.2 |
| 53 | DERIVE_SERIALIZATION | EDR-070 | `@derive(Serialize, Deserialize)` via macros; format-agnostic |
| 54 | DECLARATIVE_CONSTRUCTS | EDR-073 | Declarative patterns with documented desugaring; query exprs deferred |
| 55 | PROPERTIES | EDR-062 | Getter/setter sugar over attribute access; uniform `.name` access |
| 56 | COPY_ON_WRITE | EDR-061 | Invisible optimisation for value semantics; StdLib / Strategy |

#### Important Tier — Correction (1 concept, already counted above)

*CONTEXT_PARAMETERS and REPRESENTATION_MODIFIERS were classified as corrections during the essential tier processing (see EDR-037, EDR-038). No additional corrections arise from the important tier.*

#### Rejected (4 concepts)

| # | Concept | EDR | Rejection Rationale |
|---|---------|-----|-------------------|
| R1 | PROTOTYPE-BASED OBJECT MODEL | EDR-075 | Contradicts nominal trait system |
| R2 | SIGNIFICANT WHITESPACE | EDR-076 | Violates Explicitness and LLM Generability |
| R3 | DYNAMIC TYPING | EDR-077 | Contradicts Declarative-With-Static-Guarantees; Gradual Typing covers use case |
| R4 | CLASS/STRUCT AS PRIMARY COMPOSITION | EDR-078 | Superseded by traits + primitive blocks |

#### Deferred (50 concepts)

See DECISION_PIPELINE.md § Deferrable Tier for the complete list.
Deferral categories: Scope boundary, Dependency gated, Complexity deferred,
Low priority, Implementation detail, LLM toolchain.

---

### Consequences

**Positive:**
- Every concept research file has been processed through triage →
  pipeline → classification → EDR
- Language vs. StdLib boundary is clearly defined (35 Language,
  16 StdLib, 3 Policy, 4 Rejected, 50 Deferred, 2 Corrections)
- Essential and important tiers are fully accepted and registered in
  CORE_CONCEPTS.md
- Rejected concepts have explicit EDRs documenting the reasoning,
  preventing re-litigation
- Corrections to SEMANTIC_MODEL and PRIMITIVE_BLOCKS are documented
  and traceable

**Negative:**
- Concept volume (35 Language + 16 StdLib accepted) creates
  maintenance burden for cross-references across specification
  documents
- 50 deferred concepts remain in research inbox — while properly
  triaged, they represent backlog for v0.2/v0.3 planning
- Some accepted concepts (e.g., PERSISTENT_DATA_STRUCTURES) are
  partially deferred — the `Immutable` trait is accepted but
  collection implementations are deferred, requiring careful
  specification wording

---

### Gate Validation

**ARCHITECTURAL_INTEGRITY_GATE:**
- Does the Phase 4 output cohere as a complete set? **Yes.** Every concept
  in the essential and important tiers has passed the Decision Pipeline,
  been classified via D-03/D-04 criteria, and been accepted via EDR. The
  Language/StdLib/Policy boundary is unambiguous and documented.
- Does every accepted concept have a verifiable trace to its primitive
  decomposition? **Yes.** CORE_CONCEPTS.md includes a Primitive Decomposition
  section for every concept, showing how it maps to the 9-primitive set
  (PRIMITIVE_BLOCKS) or why it cannot be decomposed (new semantics at
  the compiler level).

**LOGICAL_CONSISTENCY_GATE:**
- No contradictions between accepted concepts. **Verified.**
  - CONCURRENCY_MODEL (EDR-033) and ASYNC_AWAIT (EDR-047) are
    complementary — async modifies proc/fun/new; delegates provide
    isolation boundaries.
  - NULL_SAFETY (EDR-018) and TYPE_LEVEL_NULL_SAFETY (EDR-028) form
    a consistent progression (type → flow-sensitive narrowing).
  - PATTERN_MATCHING (EDR-025) and PATTERN_MATCHING_DISPATCH (EDR-026)
    share the same underlying exhaustiveness semantics.
  - GENERICS (EDR-024) and COMPILE_TIME_EXECUTION (EDR-031) converge
    on comptime-as-generics mechanism.
  - ERROR_HANDLING (EDR-020) and ERROR_UNION (EDR-023) coexist with
    complementary roles (payload-bearing vs. tag-only errors), sharing
    the `?` propagation operator.
  - STRUCTURAL_TYPING (EDR-044) respects nominal-by-default priority
    (explicit `impl` overrides structural match).
  - COMMAND_PATTERN_VIA_DELEGATE (EDR-071) confirms no new construct
    needed — existing delegate/function model subsumes the pattern.
  - DECLARATION_BY_ASSIGNMENT (EDR-074) is compatible with TYPE_INFERENCE
    (EDR-027) — type inferred from initializer.

---

### Alternatives Considered

1. **Defer all important-tier concepts to v0.2.** Rejected. Important-tier
   concepts like ASYNC_AWAIT, ALGEBRAIC_DATA_TYPES, and CONTRACTS are
   essential for a complete v0.1 specification. Without them, the
   specification describes a language too minimal to be practically useful.

2. **Process concepts without the Decision Pipeline.** Rejected. The
   10-question pipeline ensures consistent evaluation across all concepts
   and prevents classification drift. Concepts evaluated later under the
   same rubric receive the same quality gate.

3. **Merge all StdLib concepts into a single "StdLib" entry.** Rejected.
   Each StdLib concept has distinct semantics, rationale, and primitive
   decomposition. A single entry would lose traceability.

4. **Elevate POLICY concepts to Language status.** Rejected per D-04.
   Policies are implementation choices, not semantic additions. Routing
   them to `how/strategies/` preserves the separation between language
   semantics and implementation strategy.
