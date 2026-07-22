---
id: SEED-001
status: dormant
planted: 2026-07-22
planted_during: Phase 1.1 — Foundation Completion
trigger_when: "when entering Phase 4 (Derived Features & Decision Pipeline)"
scope: medium
---

# SEED-001: Triage concept research documents into Tier 1/2/3 to sequence Phase 4's Decision Pipeline

## Why This Matters

Phase 4 of `when/ROADMAP.md` runs every concept in `how/concepts/research/` through
a 10-question Decision Pipeline before it can be accepted into `what/concepts/`.
At ~100 research files, reviewing them in arbitrary order risks burning review
effort on sugar/library/tooling concepts before the semantic foundations they
depend on are even settled — an accepted `PATTERN_MATCHING.md` is worthless if
`DATA_MODEL.md` (which it destructures) is still unresolved underneath it.

This triage applies the Pareto principle: ~20% of concept files (Tier 1) define
~80% of the language's semantic identity. Sequencing Tier 1 first, then Tier 2,
then explicitly deferring Tier 3 to v0.2/v0.3, cuts the effective Phase 4 review
surface from ~100 files to ~40 and gives a concrete order of operations instead
of an undifferentiated backlog.

It also flags several Tier 3 candidates that may directly **contradict**
Orthon's stated principles (`PROTOTYPE.md`, `SIGNIFICANT_WHITESPACE.md`,
`DYNAMIC_TYPING.md`, `CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md`) — these
should probably be rejected outright during Phase 4 rather than merely
deferred, and that determination itself is a decision worth an EDR.

## When to Surface

**Trigger:** when entering Phase 4 (Derived Features & Decision Pipeline)

This seed should surface automatically during `/gsd-new-milestone` if milestone
scope touches Phase 4, and should also be manually pulled up via
`/gsd-discuss-phase 4` or `/gsd-plan-phase 4` when that phase starts — its
tier ordering is meant to directly inform the plan's task sequencing.

## Scope Estimate

**Medium** — this isn't new work by itself; it's a sequencing artifact for
Phase 4, which itself is a full phase of the roadmap (concept design review +
EDR authoring for every accepted concept). The triage itself is done; applying
it is the work of Phase 4.

## Breadcrumbs

- `when/ROADMAP.md` §"Phase 4: Derived Features & Decision Pipeline" (line ~219) — the
  10-question Decision Pipeline this triage is meant to sequence
- `how/concepts/research/` — the ~100 files being triaged
- `how/gates/DECISION_VALIDATION.md` — the six validation gates each concept must pass
  regardless of tier
- `what/concepts/` — destination for accepted concepts
- `how/decision_records/INDEX.md` — where the "reject as contradicting principles"
  calls for Tier 3 outliers should be recorded as EDRs

## Notes

Full triage tables (Tier 1 ~18 files, Tier 2 ~23 files, Tier 3 ~45+ files) are
preserved below verbatim since they are the actual working artifact — do not
re-derive this list from scratch when Phase 4 starts; start from here and
verify each file still exists / hasn't since been accepted.

### Tier 1 — Must-Have (semantic bedrock)

| Concept | Why it's essential |
|---------|-------------------|
| FOUNDATIONAL_ABSTRACTIONS.md | The Data + Data Modifiers hypothesis is the central organizing idea of Orthon. If this is wrong, everything built on it is wrong. |
| DATA_MODEL.md | Types, representations (Value, Tuple, Reference, Sequence, Set, Option, Result) — the type system itself. |
| FUNCTIONS.md | Functions are the primary unit of computation. No language without them. |
| OWNERSHIP.md | Who owns data? Linear, shared, borrowed? — affects every memory operation. |
| MUTABILITY.md | Mutation semantics — immutable by default? Affects every assignment. |
| ALLOCATION.md | Memory allocation — stack vs heap vs arena. Affects every value. |
| EQUALITY.md | Structural vs identity comparison — affects every `==`. |
| EXECUTION_PROGRAM.md | The core innovation: decoupling semantics from execution strategy. Defines Orthon's identity vs other langs. |
| ERROR_HANDLING.md | How errors propagate — affects every function call. |
| NULL_SAFETY.md | Fundamental safety guarantee. Without it, every reference is nullable. |
| GENERICS.md | Parametric polymorphism — no standard library without it. |
| PATTERN_MATCHING.md | Destructuring control flow — replaces chains of if/instanceof. |
| VALUE_SEMANTICS.md | Value semantics by default — core principle that affects every type. |
| NAMESPACES.md | Module system — affects every file and every import. |
| VISIBILITY_AND_ENCAPSULATION.md | public/private/etc — affects every API boundary. |
| COMPOSITION_OVER_INHERITANCE.md | Structural composition model — replaces class inheritance. |
| TRAITS.md | Interfaces/typeclasses — without them, no polymorphism. |
| EXPRESSION_ORIENTED_LANGUAGE.md | Expression vs statement — affects every syntactic construct. |
| FINAL_BY_DEFAULT.md | Immutability default — affects every variable declaration. |

**Count: ~18 — the language's skeleton.**

### Tier 2 — Important (usable and expressive)

| Concept | Why it matters |
|---------|---------------|
| CONCURRENCY.md | Parallel execution — most programs need it, but could be defined as a library. |
| ASYNC_AWAIT.md | Async programming — could be syntactic sugar over concurrency primitives. |
| GENERATORS.md | Lazy sequences — elegant but expressible via closures. |
| ITERATION_LOOP.md | Loop construct — could be syntactic sugar over recursion/sequences. |
| OBJECT_INITIALIZATION.md | Constructor patterns — ergonomic but not foundational. |
| UNPACKING.md | Destructuring — ergonomic but could be explicit. |
| PROPERTIES.md | Field access with logic — getter/setter sugar. |
| SMART_CAST.md | Type narrowing — nice but not blocking. |
| EXTENSION_FUNCTIONS.md | Adding methods to existing types — ergonomic. |
| COPY_ON_WRITE.md | Memory optimization — important for correctness but implementation detail. |
| DELEGATION.md | Composition via delegation — useful pattern. |
| STRUCTURAL_TYPING.md | Duck typing — powerful but not essential (nominal typing works). |
| GRADUAL_TYPING.md | Optional static types — important LLM-era adoption strategy, but not core. |
| ENUM_ALTERNATIVES.md | Sum types — could be built from sealed traits + pattern matching. |
| ALGEBRAIC_DATA_TYPES.md | ADTs — natural extension of the data model. |
| CONTEXT_LIMITED_MODULES.md | Capability-based access — important for security but not core semantics. |
| SLOTS.md | Compact field storage — partially subsumed by REPRESENTATION_MODIFIERS.md. |
| DATACLASSES.md | Auto-generating equals/hash/toString — ergonomic but could be a compiler macro. |
| REPRESENTATION_MODIFIERS.md | struct(T)/boxed(T)/shared(T) — powerful but builds on the data model. |
| CONTRACTS.md | Design-by-contract — important for safety-sensitive code. |
| DECLARATION_BY_ASSIGNMENT.md | `let x = ...` syntax — nice but not semantic. |
| SORTING.md | Sort algorithm — standard library, not language. |
| SPAN.md | Memory views — important for systems programming but library-level. |
| NAMED_AND_OPTIONAL_PARAMETERS.md | Function call ergonomics. |

**Count: ~23 — the language's muscles.**

### Tier 3 — Nice-to-Have (sugar, domains, tooling, or deferrable)

| Concept | Why it's Tier 3 |
|---------|-----------------|
| CUSTOM_OPERATORS.md | Cool but dangerous — can wait for v0.2. |
| METAOBJECTS.md | Metaprogramming — compiler macros or library, not core. |
| REFLECTION_ALTERNATIVES.md | Reflection — important but not for v0.1. |
| LITERATE_PROGRAMMING.md | Documentation-oriented programming — niche. |
| LLM_NATIVE_TOOLCHAIN.md | LLM tooling — important for Orthon's identity but separate from language spec. |
| DIALECTS.md | Language subsets — advanced, not for v0.1. |
| HOMOICONICITY.md | Code-as-data — philosophically interesting but not required. |
| TYPED_HOLES.md | `_` as typed placeholder — useful but sugar. |
| PROPERTY_WRAPPERS.md | @property-like — sugar over extension functions. |
| PROTOCOL_EXTENSIONS.md | Default implementations in protocols — nice but not core. |
| DEFAULT_INTERFACE_METHODS.md | Same — can wait. |
| MIXIN.md | If you have traits + delegation, you have mixins. |
| OPEN_CLASSES.md | Modifiable classes — possible anti-pattern in Orthon's philosophy. |
| PARTIAL_CLASSES.md | Split class definitions — C# specific, niche. |
| NESTED_CLASSES.md | Inner classes — expressible via namespaces. |
| SINGLETON_CLASS.md | Singleton pattern — objects work without a dedicated feature. |
| ROOT_OBJECT.md | Universal base class — Java/C# pattern, Orthon may not need it. |
| OBJECTS_AND_SINGLETONS.md | Object literals — expressible via anonymous types. |
| PROTOTYPE.md | Prototype-based OO — **contradicts Orthon's design; candidate for outright rejection.** |
| CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md | Classes as primary — **contradicts Data First principle; candidate for outright rejection.** |
| ACTORS.md | Actor model — advanced concurrency, not for v0.1. |
| SOFTWARE_TRANSACTIONAL_MEMORY.md | STM — research-grade, not for v0.1. |
| TASK_PARALLEL_LIBRARY.md | Parallel patterns — library, not language. |
| DIFFERENTIABLE_PROGRAMMING.md | ML/AI — domain-specific, not for v0.1. |
| EVENTS.md | Event system — library, not language. |
| QUERY_EXPRESSIONS.md | LINQ-like — syntactic sugar, not core. |
| INDEXERS.md | `obj[key]` syntax — sugar over function calls. |
| BUILDER_PATTERN_ELIMINATION.md | Named parameters + defaults make builders obsolete. |
| DISPOSABLE_PATTERN.md | Resource management via contexts/RAII, not a language feature. |
| LAZY_INITIALIZATION.md | `lazy` keyword — sugar over closure + memoization. |
| SAFE_SANDBOX.md | Sandboxed execution — advanced, not for v0.1. |
| SEMANTIC_ANNOTATIONS.md | @pure, @async — interesting but speculative. |
| SIGNIFICANT_WHITESPACE.md | Python-style — **contradicts Orthon's syntax principles (ROADMAP Phase 5: "No significant whitespace"); candidate for outright rejection.** |
| SCRIPT_EXECUTION_MODEL.md | REPL/script mode — tooling question, not language design. |
| INTERACTIVE_DEVELOPMENT.md | LSP, REPL — tooling, not spec. |
| NATIVE_COMPILATION.md | AOT compilation — execution strategy, not semantics. |
| METADATA_ANNOTATIONS.md | @Override, @Deprecated — compiler hints, not core. |
| DYNAMIC_TYPING.md | `dynamic` type — **contradicts Orthon's explicitness principle; candidate for outright rejection.** |
| FUNCTION_OVERLOADING.md | Multiple dispatch — Orthon may not need it. |
| TOP_LEVEL_DECLARATIONS.md | Top-level functions — convenient but trivial. |
| USING_DIRECTIVES.md | Import sugar — trivial. |
| CODE_BLOCK_AS_HOF.md | Last-block-arg syntax — sugar. |
| COLLECTIONS_FUNCTIONAL_API.md | map/filter/reduce — standard library. |
| DYNAMIC_COLLECTIONS.md | Auto-growing arrays — implementation detail. |
| IMPERATIVE_CRUTCH_*.md (10 files) | Anti-pattern research — informs design but not language features themselves. |
| LANGUAGE_LLM_COMPARISON.md | Research reference — not a feature. |
| COMPILE_TIME_CONCURRENCY_SAFETY.md | Advanced static analysis — aspirational. |
| GENERATORS_IDEMPOTENT_GENERATION.md | Deterministic code gen — LLM tooling concern. |
| IMPERATIVE_CRUTCHES_INDEX.md | Index — meta. |
| EXPLICIT_COMPOSITION.md | Composition syntax — could be sugar over traits. |
| DEVELOPER_TOOLING.md | Tooling spec — separate from language spec. |

**Count: ~45+ — the language's accessories.**

### Summary by Volume

| Tier | Count | Approx % | Character |
|------|-------|----------|-----------|
| Tier 1 — Must-have | ~18 | ~20% | Skeleton |
| Tier 2 — Important | ~23 | ~25% | Muscles |
| Tier 3 — Nice-to-have | ~45+ | ~55% | Accessories |

### Recommended Action for Phase 4

Run Tier 1 through the Decision Pipeline first, do a quick pass through Tier 2,
and explicitly defer Tier 3 to a v0.2/v0.3 milestone with a one-paragraph
rationale per concept (except the four flagged Tier 3 files above, which
should be evaluated for outright rejection via EDR rather than deferral).
