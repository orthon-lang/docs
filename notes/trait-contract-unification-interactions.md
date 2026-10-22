# Interaction Draft — Unified Contract Construct (EDR-098)

> **Status:** Draft, exploratory. Not part of the specification.
> **Purpose:** the per-concept "Interactions" input that
> [`CONCEPT_PIPELINE.md`](../how/CONCEPT_PIPELINE.md) Stage 11 expects, prepared
> early as gate resolution R8 of
> [EDR-098](../how/decision_records/architecture/EDR-098-trait-contract-unification.md)
> (Proposed). It feeds the pair-wise matrix of the Cross-Cutting Review (Phase 6,
> [`what/CROSS_CUTTING.md`](../what/CROSS_CUTTING.md), still a placeholder) and does
> not replace it.
> **Format:** one row per pair of the unified construct with an accepted concept.
> The matrix format is still open ([`interaction-matrix-format.md`](interaction-matrix-format.md),
> open question 2); severity uses the words proposed there: compatible,
> constrained, undefined (incompatible: none found).
> **Date:** 2026-10-22

## How it was made

The rows come from the review decisions 1–24 of the hypothesis
[`TRAIT_CONTRACT_UNIFICATION.md`](../how/concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md)
and from a keyword search of `what/concepts/` for `trait`, `impl ... for`,
`Callable` and `contract`. A row marked **undefined** was not examined in depth
and needs the Phase 6 analysis. Concepts with no occurrence in the search are
listed at the end as not examined, not as proven independent.

## Interactions

| Concept | Interaction | Severity | Constraint or decision | Open item |
|---------|-------------|----------|------------------------|-----------|
| TRAITS (EDR-019) | Merged with CONTRACTS into one concept; coherence, orphan rule, dispatch carried over | constrained | Keyword and conformance syntax superseded in part once the words are decided (EDR-098 Decision item 9) | Words and position (`CONFORMANCE_SYNTAX.md`) |
| CONTRACTS (EDR-056, EDR-097) | Merged; conditions, purity, Liskov inheritance, named result unchanged | constrained | Invariant is checked after creation and after each `proc` exit (decision 18) | The accepted wording "entry and exit of each public method" is amended on acceptance |
| STRUCTURAL_TYPING (EDR-044) | `structural` is a mode of a type contract | constrained | Signatures only, never conditions; functions obey explicitly (decision 21) | Wording clarification of EDR-044 |
| GENERICS | Bounds written with `as`; a generic function may obey a contract (decision 23) | constrained | Parameters linked by name, bounds not stronger than the contract's; a function contract as a bound waits for v0.2 | Deferral D-1 |
| CONSTRAINED_TYPES (EDR-080) | Same desugaring shape (`struct` + `Callable` + conformance); a value constraint stays a different kind | compatible | F2 reuses the pattern | None |
| EQUALITY (EDR-017) | `Eq` is a type-level contract; the Transitivity Invariant needs a multi-instance property | constrained | Stays in prose in v0.1 (decision 17) | Deferral D-3 |
| ERROR_UNION, ERROR_HANDLING | Result of an implementation converts to the contract result; narrower error set to wider (decision 20) | constrained | An error-union result in a contract means any set | Notation if `ERROR_REPRESENTATION_UNIFICATION.md` is accepted |
| SYSTEM_ERROR (EDR-089) | A violated or unevaluable condition is a `PROGRAM_ERROR` (decision 22) | compatible | Termination is defined when checks are enabled | Class and termination fixed by EDR-098; resolution R9 |
| ITERATOR_PROTOCOL | `Iterator<T>` is a type-level contract generic over `T` | compatible | Unchanged apart from the rename | Associated types stay with type-level contracts (hypothesis Open Question 5) |
| ALGEBRAIC_DATA_TYPES | Sealed trait hierarchies and derived conformance for variants | compatible | Unchanged apart from the rename | None found |
| PATTERN_MATCHING_DISPATCH | Complement to single-receiver contract dispatch | compatible | No change | None found |
| DELEGATION, COMMAND_PATTERN_VIA_DELEGATE | A type obeys a contract by composing a contained instance; forwarding is generated | undefined | Forwarded members must keep the member kind (decision 19) | Examples still use an older receiver spelling (todo 2026-10-21) |
| CONCURRENCY_MODEL (EDR-033), COPY_ON_WRITE | `Send` and `Sync` are marker contracts with no members | compatible | Marker contracts are allowed (decision 24) | None found |
| AST_MACROS, DERIVE_SERIALIZATION | `@derive` generates conformance statements | constrained | Generated text follows the chosen words | Rename cascade |
| EXTENSION_FUNCTIONS | Member precedence against contract methods; whether extension functions satisfy a contract is an open question there | undefined | Not decided by EDR-098 | Phase 6 |
| COMPILE_TIME_EXECUTION | Bounds on comptime parameters; conditions proved or evaluated at compile time where possible | undefined | A function contract as a bound waits for v0.2 | Phase 6 |
| REQUIRE_USING_DEPENDENCY_SLOTS (EDR-081) | `require` and `requires` differ by one letter in the same tail position | constrained | Recorded as a syntax remark | Syntax pipeline |
| NAMED_AND_OPTIONAL_PARAMETERS | Parameters are bound by name (decision 2); whether a signature match includes parameter names and defaults is not stated | undefined | Exact match of parameters in v0.1 (decision 20) | Phase 6 |
| ASYNC_AWAIT (EDR-047) | `async` is a modifier on method kinds; whether it takes part in kind matching (decision 19) is not stated | undefined | Kind names are an open todo | Phase 6 |
| GRADUAL_TYPING | Calls type annotations on signatures "contracts"; overlaps with the umbrella term | constrained | Terminology only (R1, proposed glossary entry) | Wording check |
| PROPERTIES | The word *property* collides with a candidate name for multi-instance contracts | compatible | The naming is deferred with D-3 | Deferral D-3 |
| COMPILER_AS_STATIC_ANALYZER | New diagnostics: kind mismatch, error set, structural conditions, stronger bound, violation report | compatible | Diagnostic wording not specified yet | Resolution R15 |
| PERSISTENT_DATA_STRUCTURES, TYPE_INFERENCE, TYPE_LEVEL_COMPUTATION, SMART_CAST, UNION_INTERSECTION_TYPES | Mention traits or contracts in passing; no interaction was worked out | undefined | None | Phase 6 |

## Not examined

Concepts with no occurrence of the search terms, such as COLLECTION_LITERAL_SYNTAX,
INDEXING, RANGE, SLICE, SPAN, UNPACKING, GENERATORS and ITERATION_LOOP. This is not
a proof of independence.

## Next step

Fold the rows into `what/CROSS_CUTTING.md` when Phase 6 starts and decide the
matrix format. Every row marked undefined becomes either a resolved row or an
entry in [`what/CONFLICT_REGISTRY.md`](../what/CONFLICT_REGISTRY.md).
