# Library Boundary

> Summary table of all language features classified during Phase 4
> (Derived Features & Decision Pipeline). Each entry is derived from
> the classification field in its corresponding EDR.
>
> **See also:** Each concept EDR for full classification rationale
> and primitive decomposition.
>
> **Status:** Finalized — Phase 4 complete.
> **See also:** [`CORE_CONCEPTS.md`](CORE_CONCEPTS.md),
> [`DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md)

---

## Classification Summary

| Category | Count | Concepts |
|----------|-------|----------|
| **Language** | 35 | EQUALITY, NULL_SAFETY, TRAITS, ERROR_HANDLING, LAZY_SEQUENCE_GENERATORS, ITERATOR_PROTOCOL, ERROR_UNION, GENERICS, PATTERN_MATCHING, PATTERN_MATCHING_DISPATCH, TYPE_INFERENCE, TYPE_LEVEL_NULL_SAFETY, AST_MACROS, COMPILER_AS_STATIC_ANALYZER, COMPILE_TIME_EXECUTION, CONCURRENCY_MODEL, ALGEBRAIC_DATA_TYPES, LITERAL_TYPES, STRUCTURAL_TYPING, UNION_INTERSECTION_TYPES, TYPE_LEVEL_COMPUTATION, ASYNC_AWAIT, GENERATORS, EMIT_AS_INTERMEDIATE_RESULT, ITERATION_LOOP, UNPACKING, CONTRACTS, EXTENSION_FUNCTIONS, GRADUAL_TYPING, SMART_CAST, SLOTS, SPAN, CONTEXT_LIMITED_MODULES, DECLARATION_BY_ASSIGNMENT, COMMAND_PATTERN_VIA_DELEGATE |
| **Standard Library** | 16 | COMPOSABLE_COLLECTION_OPS, CONCURRENCY, PUSH_STREAMS, OBJECT_INITIALIZATION, COLLECTION_LITERAL_SYNTAX, DATACLASSES, DELEGATION, NAMED_AND_OPTIONAL_PARAMETERS, SORTING, DECLARATIVE_MULTI_KEY_SORT, IMMUTABLE_DATE_TIME, PERSISTENT_DATA_STRUCTURES, DERIVE_SERIALIZATION, DECLARATIVE_CONSTRUCTS, PROPERTIES, COPY_ON_WRITE |
| **Policy** | 3 | ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM |
| **Rejected** | 4 | PROTOTYPE, SIGNIFICANT_WHITESPACE, DYNAMIC_TYPING, CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION |
| **Deferred** | 50 | See [`DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md) § Deferrable Tier |
| **Corrections** | 2 | CONTEXT_PARAMETERS → SEMANTIC_MODEL, REPRESENTATION_MODIFIERS → PRIMITIVE_BLOCKS |

## By Tier

### Essential Tier (22 concepts)

Essential-tier concepts are the semantic bedrock of Orthon — must-haves for v0.1. Of the 22 essential-tier research files, the Decision Pipeline classified:

**Language (16):**
1. EQUALITY — three-operator model (value, semantic, identity)
2. NULL_SAFETY — `Option<T>` without null sentinel
3. TRAITS — nominal trait system for polymorphism
4. ERROR_HANDLING — `Result<T, E>` with explicit propagation
5. LAZY_SEQUENCE_GENERATORS — `emit` keyword for lazy production
6. ITERATOR_PROTOCOL — trait-based lazy consumption protocol
7. ERROR_UNION — Zig-style `!T` tag-only error union
8. GENERICS — trait-bounded parametric polymorphism
9. PATTERN_MATCHING — exhaustive expression-oriented matching
10. PATTERN_MATCHING_DISPATCH — multimethod dispatch via definition-site matching
11. TYPE_INFERENCE — local bidirectional inference
12. TYPE_LEVEL_NULL_SAFETY — flow-sensitive `Option<T>` narrowing
13. AST_MACROS — AST macros as comptime functions
14. COMPILER_AS_STATIC_ANALYZER — seven verification layers
15. COMPILE_TIME_EXECUTION — unified comptime model
16. CONCURRENCY_MODEL — delegate-based, message-passing concurrency

**Standard Library (1):**
- COMPOSABLE_COLLECTION_OPS — map/filter/reduce on Iterator

**Policy (3):**
- ALLOCATION — allocation as Implementation Policy
- REGION_BASED_MEMORY_MANAGEMENT — arena allocation sub-policy
- EXECUTION_PROGRAM — decoupling semantics from execution strategy

**Corrections (2):**
- CONTEXT_PARAMETERS → SEMANTIC_MODEL (Evaluation/Visibility dimension)
- REPRESENTATION_MODIFIERS → PRIMITIVE_BLOCKS (annotation on pack/reference)

### Important Tier (36 concepts)

Important-tier concepts add ergonomics, expressiveness, and safety guarantees beyond the essential core. Of the 36 important-tier research files, the Decision Pipeline classified:

**Language (19):**
1. ALGEBRAIC_DATA_TYPES — combined sum/product declaration
2. LITERAL_TYPES — values as singleton types
3. STRUCTURAL_TYPING — structural trait satisfaction (opt-in)
4. UNION_INTERSECTION_TYPES — structural union combinator `A | B`
5. TYPE_LEVEL_COMPUTATION — closed set of 8 compiler intrinsics
6. ASYNC_AWAIT — async as explicit modifier on proc/fun/new
7. GENERATORS — bidirectional yield and generator expressions
8. EMIT_AS_INTERMEDIATE_RESULT — semantic refinement of EDR-021
9. ITERATION_LOOP — for/while/loop constructs
10. UNPACKING — destructuring assignment matching pack/unpack
11. CONTRACTS — requires/ensures/invariant clauses
12. EXTENSION_FUNCTIONS — method-call syntax on external types
13. GRADUAL_TYPING — optional type annotations with selective checking
14. SMART_CAST — flow-sensitive type narrowing after checks
15. SLOTS — fixed-field storage for types
16. SPAN — lifetime-tracked safe memory view
17. CONTEXT_LIMITED_MODULES — module system with effect isolation
18. DECLARATION_BY_ASSIGNMENT — first assignment creates variable
19. COMMAND_PATTERN_VIA_DELEGATE — existing delegate/function coverage

**Standard Library (15):**
1. CONCURRENCY — StdLib utilities on delegate model (channels, select)
2. PUSH_STREAMS — observable-style reactive streams on delegate+channel
3. OBJECT_INITIALIZATION — named params, builders, copy-with-modify
4. COLLECTION_LITERAL_SYNTAX — sugar for collection constructors
5. DATACLASSES — @derive-based data carriers
6. DELEGATION — @delegate macro for composition
7. NAMED_AND_OPTIONAL_PARAMETERS — sugar over function call model
8. SORTING — stable sort algorithm on Ord trait
9. DECLARATIVE_MULTI_KEY_SORT — key-path sort sugar
10. IMMUTABLE_DATE_TIME — value-semantics date/time types
11. PERSISTENT_DATA_STRUCTURES — Immutable trait + deferred collections
12. DERIVE_SERIALIZATION — @derive(Serialize, Deserialize) via macros
13. DECLARATIVE_CONSTRUCTS — declarative patterns with documented desugaring
14. PROPERTIES — getter/setter sugar over attribute access
15. COPY_ON_WRITE — invisible optimisation for value semantics

**Deferred (2):**
- PROTOCOL_EXTENSIONS — dependency gated on trait coherence
- PERSISTENT_DATA_STRUCTURES (collections deferred to v0.2, trait accepted)

*Note: 1 concept (PERSISTENT_DATA_STRUCTURES) is dual-listed — its `Immutable` trait is accepted (StdLib), but collection implementations deferred to v0.2.*

### Rejected Tier (4 concepts)

See EDR-075 through EDR-078 and [`how/concepts/research/reject/README.md`](../how/concepts/research/reject/README.md).

1. PROTOTYPE (EDR-075) — prototype-based object model contradicts Orthon's nominal trait system
2. SIGNIFICANT_WHITESPACE (EDR-076) — violates Explicitness and LLM Generability
3. DYNAMIC_TYPING (EDR-077) — contradicts Declarative-With-Static-Guarantees; Gradual Typing covers the use case
4. CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION (EDR-078) — superseded by traits + primitive blocks

### Deferrable Tier (50 concepts)

See [`DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md) § Deferrable Tier for the complete list with deferral rationales, target versions, and dependencies.

### Corrections (2 concepts)

Two borderline concepts resolved as corrections to existing documents rather than standalone features:

| Concept | Correction Target | EDR | Rationale |
|---------|------------------|-----|-----------|
| CONTEXT_PARAMETERS | SEMANTIC_MODEL (Evaluation/Visibility) | EDR-037 | Cross-cutting concern of context flow; not a standalone feature |
| REPRESENTATION_MODIFIERS | PRIMITIVE_BLOCKS (pack/reference annotations) | EDR-038 | Orthogonal annotations on existing primitives, not new operations |

## Classification Criteria

Per D-03 (from 04-CONTEXT.md):

1. **Semantic uniqueness** (primary) — Does this feature add new semantics not expressible through composition of existing primitives? If yes → Language.
2. **Compiler dependency** (secondary) — Must the compiler understand this feature for correct code generation? If yes → Language.
3. If neither → Standard Library (composable from Language primitives) or External (domain-specific, out of scope for M1).

Per D-04: Policy-level concepts route to `how/strategies/` area.

## Decision Pipeline Reference

Every concept above ran through the 10-question Decision Pipeline defined in [`how/process/DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md). The pipeline log for each concept is recorded in that document's Pipeline Application section.
