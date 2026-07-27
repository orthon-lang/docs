# Decision Pipeline

> This document defines the 10-question pipeline that every proposed
> language feature must pass before detailed design begins.
>
> **Status:** Finalized during Phase 1.1 (Foundation Completion). The
> Pipeline Application log below stays empty until Phase 4 populates
> it per-feature — this is expected, not a gap.
> **See also:** [`ROADMAP.md`](../../when/ROADMAP.md) § Phase 4,
> [`DECISION_PROCESS.md`](DECISION_PROCESS.md),
> [`Concept Design Review`](../../how/concept-design-review.md)

---

## The Pipeline

For each proposed feature, run through these 10 questions in order:

```
 1. What problem are we solving?
 2. Is this a language problem or a library problem?
 3. Can it be solved with existing primitives?
 4. Does it violate any Design Principle?
 5. Does it add new semantics (vs. syntactic sugar)?
 6. Can it be expressed through composition?
 7. Can it be syntactic sugar over existing primitives?
 8. Is this an optimisation, not semantics?
 9. Does it affect backward compatibility?
10. Is it worth adding at all?
```

## Decision Flow

```
Proposal
    │
    ▼
Q1–Q3 ──► If "library problem" or "exists in primitives" → REJECT as language feature
    │
    ▼
Q4     ──► If violates principle → REJECT (or escalate to EDR for principle change)
    │
    ▼
Q5–Q7  ──► If expressible as sugar or composition → mark as syntactic sugar
    │
    ▼
Q8     ──► If optimisation, not semantics → move to OPTIMIZATION_MODEL
    │
    ▼
Q9     ──► Assess compatibility impact
    │
    ▼
Q10    ──► Final value judgement → ACCEPT / DEFER / REJECT
```

## Pipeline Application

<!-- Populated during Phase 4 — Essential Core (Wave 1) -->

### Essential Core — Wave 1

#### EQUALITY

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Programmers need predictable, unambiguous equality semantics. Reference vs. value equality is the #1 source of bugs in Java/Python/JS. |
| Q2 | Is this a language problem or a library problem? | **Language.** The compiler must know equality semantics for trait constraint checking and code generation (structural comparison). Three distinct operators (`===`, `==`, `is`) require parser and type-system support. |
| Q3 | Can it be solved with existing primitives? | No. `===` (structural value equality) requires compiler-generated field-by-field comparison — not expressible via composition of existing 9 primitives. `is` (identity) requires a runtime concept of reference identity not present in primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (different operators for different semantics), Consistency (same operator = same semantics for all types), Data First (structural by default). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Three distinct semantic operations: structural comparison, user-defined comparison, identity check. |
| Q6 | Can it be expressed through composition? | No. Structural equality requires compiler support to recurse into fields. Identity comparison requires a runtime concept of object identity. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Equality is a semantic operation, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Every programming task involves comparison. |

**Classification per D-03:** Language. Semantic uniqueness of `===` structural comparison not expressible via composition. Compiler must know equality for trait constraint checking.

**Primitive decomposition path:** `===` → compiler-generated field-by-field comparison of `pack`/`unpack` structure; `==` → `function` + trait dispatch (user-defined); `is` → `reference` identity check. None of these decompose to a single primitive without new semantics.

---

#### NULL_SAFETY

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Null pointer errors — the "billion-dollar mistake." Every reference of type `T` can silently be `null`, turning every dereference into a potential crash. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `Option<T>` type requires compiler-enforced exhaustive matching, nullable syntax (`?.`, `??`, `!`), and type-system integration (a `None` value cannot be assigned to a non-optional `T`). |
| Q3 | Can it be solved with existing primitives? | No. The `?` semantics (optional chaining short-circuit, forced unwrap with panic) require compiler support. `Option` as a sum type requires pattern matching exhaustiveness checking. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (absence is statically tracked), Explicitness (forced unwrap `!` is visible), Minimal Core (one concept — `Option<T>` — replaces an entire class of bugs). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Option<T>` introduces sum-type absence tracking. `?.` introduces short-circuit evaluation. `!` introduces a compile-time-checkable unwrap with panic contract. |
| Q6 | Can it be expressed through composition? | No. Pattern matching exhaustiveness requires compiler checking. Optional chaining short-circuit behaviour is not expressible via composition of existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Null safety is a semantic guarantee. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential. Eliminates the most common class of runtime crashes. Orthon cannot ship without it. |

**Classification per D-03:** Language. Option type adds `?` semantics not decomposable to primitives. Compiler must track nullable state.

**Primitive decomposition path:** `Option<T>` decomposes to `literal` (None/Some variants) + `pack`/`unpack` + pattern matching; `?.` adds compiler-enforced short-circuit semantics; `??` adds default-value desugaring. The exhaustiveness check adds compiler semantics beyond primitive composition.

---

#### TRAITS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Polymorphism without the fragility of class inheritance. How do different types express shared behaviour (interfaces/contracts)? |
| Q2 | Is this a language problem or a library problem? | **Language.** Trait bounds on generics, static vs. dynamic dispatch selection, and the orphan rule require compiler support. The `impl Trait for Type` syntax, `where` clauses, and `dyn Trait` dispatch are parser/type-system features. |
| Q3 | Can it be solved with existing primitives? | No. Trait dispatch (vtable for `dyn`, monomorphisation for static) requires compiler code generation. Trait bound resolution and coherence checking are type-system operations not expressible via primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (behaviour separate from data), Composition Over Inheritance (traits compose via `where T: A + B`), Minimal Core (traits replace multiple constructs: interfaces, abstract classes, mixins). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Nominal trait system: explicit `impl`, coherence (orphan rule), static dispatch via monomorphisation, dynamic dispatch via vtable, trait bounds on generics, associated types. |
| Q6 | Can it be expressed through composition? | No. Trait bounds and dispatch semantics require type-system support. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Polymorphism is a semantic concept. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Required for standard library abstractions (Iterator, Collection, Ord, Eq, etc.). |

**Classification per D-03:** Language. Nominal trait system adds interface semantics not decomposable to primitives. Compiler must resolve trait bounds.

**Primitive decomposition path:** Trait declaration → `function` signatures + `identifier` + `scope` (trait block); `impl` block → `function` implementations + `scope`; `dyn Trait` → `reference` + vtable dispatch; static dispatch → monomorphisation of generics (`function` + type parameters). The coherence rule, bound resolution, and dispatch selection add compiler-level semantics beyond primitive composition.

---

#### ERROR_HANDLING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How does a program react to failure without crashing or producing silent corruption? Errors must be visible in the function contract and handling must be composable. |
| Q2 | Is this a language problem or a library problem? | **Language.** `Result<T,E>` is a type with compiler-level propagation mechanism (`?` operator). The compiler must enforce exhaustive handling — unhandled `Result` values are a compile-time error. |
| Q3 | Can it be solved with existing primitives? | No. Short-circuit propagation (`?`) requires compiler support for early return. Exhaustiveness checking on `Result` matches requires pattern-matching completeness checking. The `Result` type itself requires sum type support (`pack`/`unpack` + variants), but the propagation semantics go beyond primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (errors declared in signatures, `?` is visible), Declarative With Static Guarantees (compiler enforces handling), Minimal Core (one concept — `Result<T,E>` — replaces exceptions, error codes, and checked exceptions). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Result<T,E>` is a monadic type with `Ok`/`Error` variants. `?` introduces short-circuit propagation. Combinators (`map`, `and_then`, `or_else`) define error transformation semantics. No exceptions — all fallibility is declared. |
| Q6 | Can it be expressed through composition? | No. `?` operator for automatic propagation is not expressible via composition of existing primitives. Exhaustiveness checking requires compiler support. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `?` could desugar to a `match` + early return, but the pattern-matching exhaustiveness check and type-level `Result` constraint require compiler semantics. |
| Q8 | Is this an optimisation, not semantics? | No. Error handling is a semantic operation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Errors are inevitable; the language must provide a principled mechanism. Result model is the proven modern approach (Rust, Swift, OCaml). |

**Classification per D-03:** Language. `Result<T,E>` is a type with compiler-level propagation mechanism (`?` operator). Not expressible via composition of primitives. Compiler must enforce handling.

**Primitive decomposition path:** `Result<T,E>` → sum type via `pack`/`unpack` + `literal` (Ok/Error variants) + pattern matching; `?` → compiler-enforced short-circuit propagation (match + early return) beyond primitive composition; combinators (`map`, `and_then`, `or_else`) → `function` + pattern matching. Exhaustiveness checking adds compiler semantics beyond primitive composition.

---

#### LAZY_SEQUENCE_GENERATORS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manually implementing an iterator requires writing a stateful class or object with explicit iteration methods — too much boilerplate. The language should eliminate manual iterator-implementation boilerplate by providing generators and lazy sequences as a core feature. |
| Q2 | Is this a language problem or a library problem? | **Language.** Lazy sequence semantics (`emit`) are a compiler-recognized pattern with special evaluation guarantees (lazy-by-default, per Phase 3 D-06). Not expressible via primitives alone. |
| Q3 | Can it be solved with existing primitives? | No. Lazy evaluation of a generator body — pausing and resuming execution — requires a coroutine or continuation mechanism not present in primitives. The `emit` keyword is a new syntactic form with special semantics. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (programmer describes what to produce, compiler handles state machine), Minimal Core (generators replace manual iterator classes), Explicitness (`emit` makes lazy production syntactically visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Generator functions have lazy evaluation semantics — the body does not execute eagerly but returns an iterator that produces values on demand. The `emit` keyword is a yield-like operation with resumable semantics. Infinite sequences are valid. |
| Q6 | Can it be expressed through composition? | No. Resumable function execution (coroutine/semi-coroutine semantics) is not expressible via composition of existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `emit` could desugar to iterator protocol calls, but the state-machine transformation of the generator body requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Lazy production is a semantic guarantee, not an optimization. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for lazy sequence production. Eliminates manual iterator implementation boilerplate. Enables infinite sequences, composition without intermediate allocation, and declarative stream operations. |

**Classification per D-03:** Language. Lazy sequence semantics (`emit`) are a compiler-recognized pattern with special evaluation guarantees (lazy-by-default, per Phase 3 D-06). Not expressible via primitives alone.

**Primitive decomposition path:** Generator function → `function` + state-machine transformation (compiler-generated); `emit value` → iterator protocol `next()` call + suspension/resumption; `return sequence(value)` → iterator completion + value emission; `return value ->` → equivalent desugaring. The state-machine transformation and lazy evaluation semantics add compiler-level semantics beyond primitive composition.

---

#### ITERATOR_PROTOCOL

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How does a language provide lazy, composable, memory-efficient iteration over sequences without forcing the programmer to manage iterator state manually? |
| Q2 | Is this a language problem or a library problem? | **Language.** The `Iterator` trait is a protocol definition with special `for` loop desugaring. The compiler must recognize `Iterator[T]` to desugar `for` loops, enforce type constraints, and enable optimisations. Combinators (map, filter, etc.) are StdLib. |
| Q3 | Can it be solved with existing primitives? | No. `for` loop desugaring to `next()` calls requires compiler recognition of the `Iterator` trait. The `IntoIterator` trait for collection-to-iterator conversion requires type-system support. Range expressions (e.g., `0..10`) producing iterators require syntax-level support. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one protocol covers all iteration: generators, collections, ranges, I/O streams), Orthogonality (Iterator is the consumption side, generators are the production side), Composition (combinators chain without intermediate allocation). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** `Iterator[T]` trait defines a consumption protocol. `for` loop desugaring is compiler-level. Combinators return lazy iterators (one-to-one mapping from source protocol). Single-pass semantics — iterator consumed on traversal. |
| Q6 | Can it be expressed through composition? | No. `for` loop desugaring requires the compiler to recognize `Iterator[T]` and generate the loop-expansion code. Range iterator generation requires syntax-level translation. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partially. `for` desugars to a loop calling `next()`, but the compiler must know which traits to look for. Range syntax `0..10` desugars to a range-iterator constructor, requiring syntax support. |
| Q8 | Is this an optimisation, not semantics? | No. Iteration semantics — lazy, single-pass, composable — are semantic properties, not optimisations. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. The iterator protocol is the foundation for all sequence consumption: collections, generators, ranges, I/O streams, and combinator chains. |

**Classification per D-03:** Language. Protocol definition (`next() -> Option[T]`) is a compiler-level concept (trait with special `for` loop desugaring). Combinators should be StdLib.

**Primitive decomposition path:** `Iterator[T]` trait → trait declaration (`trait` + `function` + `identifier`) per TRAITS model; `for item in iter` → loop + `call` to `next()` + pattern match on `Option`; range `0..10` → syntax desugaring to `RangeIterator` constructor + `literal`; combinators (map, filter, etc.) → `function` implementations on `Iterator[T]` (StdLib, not core). The `for` loop desugaring and range-syntax translation add compiler-level semantics beyond primitive composition.

### Essential Core — Wave 2

#### ERROR_UNION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Error type declaration and conversion boilerplate for tag-only errors. Most errors are simple identifiers (FileNotFound, Timeout) without payload data, but `Result<T, E>` requires explicit enum declaration and `From` implementations for every error type. |
| Q2 | Is this a language problem or a library problem? | **Language.** Inferred error sets, structural widening from subset to superset, and the `anyerror` escape hatch require compiler support. The `!T` type former is a new kind of type, not expressible via composition of existing constructs. |
| Q3 | Can it be solved with existing primitives? | No. Inferred error set semantics — computing the union of error tags from every fallible call in the function body — require compiler-level call-graph analysis not present in the 9-primitive set. Structural widening (subset → superset coercion) is a type-system operation beyond primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one new type former replaces error-enum declaration boilerplate), Explicitness (`!T` makes fallibility visible), Intent Over Implementation (programmer writes `!T`, compiler infers the set). The implicit widening relaxation of Explicitness is weighed against the ergonomic gain — explicit conversion would defeat the purpose. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** The `!T` type former is a distinct kind of type — not sugar for `Result<T, E>`. Inferred error sets are a new semantic operation: the compiler discovers, unions, and tracks error tags across the call graph. Structural widening is a new coercion rule. |
| Q6 | Can it be expressed through composition? | No. Error set inference is inherently compiler-level — the set is derived from the call graph, not composed from primitive constructs. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Error handling semantics (which errors can occur, how they propagate) are semantic, not optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Coexists with `Result<T, E>` from EDR-020. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most common source of error-handling boilerplate. Zig's Error Union model has proven production stability. Complements `Result<T, E>` for payload-bearing errors. |

**Classification per D-03:** Language. `!T` type former adds inferred error set semantics not decomposable to primitives. Compiler must infer and track error sets.

**Primitive decomposition path:** `!T` → not decomposable — the type former itself is new syntax; error tag literal → `literal` (unit-like tag); error set inference → compiler-level call-graph analysis beyond primitive composition; structural widening → type-system coercion rule; `?` propagation → `match` + early return (shared with EDR-020's `?` operator). The inference and widening semantics add compiler-level behaviour beyond primitive composition.

---

#### GENERICS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How to write code that works with multiple types without sacrificing type safety, performance, or readability. Generic parameters must be constrained by behavioural contracts (traits). |
| Q2 | Is this a language problem or a library problem? | **Language.** Trait bounds on generic parameters, monomorphisation, variance rules, associated type resolution — all require compiler support. The `[T: Trait]` and `where T: TraitA + TraitB` syntax are parser/type-system features. |
| Q3 | Can it be solved with existing primitives? | No. Type parameterization — the ability to abstract over types themselves — is not expressible via the 9 primitive operations. Monomorphisation (code generation per concrete type) is a compiler-level transformation. Trait bound resolution and variance checking are type-system operations beyond primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (generics are orthogonal to specific types), Minimal Core (one mechanism — trait-bounded generics — replaces manual type-specific implementations), Explicitness (trait bounds declare constraints visibly in the signature). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Parametric polymorphism adds type-level abstraction — a function parameterised over `T` has different semantics than one specialised to a concrete type. Monomorphisation preserves those semantics for each instantiation. Trait bound resolution, associated type substitution, and variance rules are new type-system operations. |
| Q6 | Can it be expressed through composition? | No. Type-level abstraction is not expressible via the 9 value-level primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Generics are a semantic concept — abstraction over types. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. Enables type-safe collections, algorithms, and abstractions. Required for standard library design. |

**Classification per D-03:** Language. Parametric polymorphism adds new semantics (type parameterization, trait bounds, monomorphisation) not expressible via composition. Cross-reference COMPILE_TIME_EXECUTION (Plan 04-03).

**Primitive decomposition path:** Generic function → `function` + type parameters (new abstraction); monomorphised instantiation → `function` + concrete type substitution (compiler transformation); trait bound → `identifier` (trait name) + type-system constraint; associated type resolution → type substitution during monomorphisation. The type-level abstraction, bound resolution, and monomorphisation add compiler-level semantics beyond primitive composition.

---

#### PATTERN_MATCHING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Cascading `if-else if-else` chains produce poor code — they mix data structure inspection with control flow, the compiler cannot verify exhaustiveness, and missed branches cause bugs. |
| Q2 | Is this a language problem or a library problem? | **Language.** Exhaustiveness checking, destructuring semantics, match ergonomics, and guard evaluation require compiler support. The `match` keyword is a new syntactic form. |
| Q3 | Can it be solved with existing primitives? | No. Exhaustiveness checking — verifying that all variants of a sum type are covered — is a compiler-level analysis not present in the 9-primitive set. Destructuring (decomposing a value by its structure) requires the compiler to know the type's structural representation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (exhaustiveness), Explicitness (`match` makes branching visible), Minimal Core (one construct replaces if-else chains, manual destructuring, and runtime type checking). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Exhaustiveness checking — compiler verification that all cases are covered. Destructuring — compiler-level decomposition of compound types. Guards — conditional predicates evaluated after structural matching. Or patterns — combined arms for multiple patterns. |
| Q6 | Can it be expressed through composition? | No. Exhaustiveness checking requires the compiler to enumerate type variants — not expressible via composition of primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `match` desugars to a decision tree of `if`/`else` and equality checks, but the exhaustiveness verification and destructuring semantics require compiler support beyond simple desugaring. |
| Q8 | Is this an optimisation, not semantics? | No. Pattern matching is a semantic operation — declarative structure description. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any modern language. Replaces entire categories of bugs (unhandled cases) with compiler-enforced correctness. Required for ergonomic use of sum types (Option, Result, Error Union). |

**Classification per D-03:** Language. Exhaustiveness checking, destructuring semantics, match ergonomics. Compiler must verify exhaustiveness. Patterns match against trait-implementing types.

**Primitive decomposition path:** `match` expression → `function` (match arms as closures) + `call` (pattern evaluation) + `scope` (arm bodies); destructuring → `pack`/`unpack` (value composition/decomposition); guard → `function` (predicate) + `call` (predicate evaluation); wildcard `_` → `identifier` (ignored binding). The exhaustiveness verification, decision tree compilation, and type-variant enumeration add compiler-level semantics beyond primitive composition.

---

#### PATTERN_MATCHING_DISPATCH

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | N-way dispatch on multiple arguments requires exponential nested checks. When a function's behaviour depends on the types of multiple arguments simultaneously, single-receiver trait dispatch is insufficient. |
| Q2 | Is this a language problem or a library problem? | **Language.** Definition-site dispatch declaration, specificity resolution, and exhaustiveness across multiple argument patterns require compiler support. The `match` parameter form is a new syntactic declaration pattern. |
| Q3 | Can it be solved with existing primitives? | No. Dispatch on argument types at the function definition site — generating a dispatch tree from declared argument patterns — is not expressible via the 9-primitive set. Specificity resolution (comparing pattern specificity across multiple arguments) is a compiler-level operation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (dispatch is orthogonal to specific type combinations), Explicitness (dispatch variants are visible in the declaration), Minimal Core (one construct replaces nested if-else type checking). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Multimethod dispatch — pattern matching on function arguments at declaration site, resolved at call site. Specificity resolution — deterministic selection of the most specific matching arm. Exhaustiveness across argument type combinations. |
| Q6 | Can it be expressed through composition? | No. Dispatch on multiple argument types simultaneously is not expressible via single-receiver trait dispatch. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — pattern matching dispatch desugars to nested pattern matching on each argument, but the exhaustiveness checking across argument combinations and specificity resolution require compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Dispatch semantics — which implementation runs for a given set of argument types — is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most egregious nested type-check boilerplate — N-way dispatch. Complements trait dispatch for multimethod scenarios. Cross-reference: COMMAND_PATTERN_VIA_DELEGATE (Plan 04-07). |

**Classification per D-03:** Language. Multimethod dispatch — pattern matching applied to function arguments at definition site, resolved at call site.

**Primitive decomposition path:** `match` declaration form → `function` (dispatch function) + `match` (per EDR-025) + `scope` (arm bodies); argument pattern → `identifier` (type name) + `pack`/`unpack` (destructuring); specificity resolution → compiler-level pattern comparison. The dispatch tree generation, specificity analysis, and cross-argument exhaustiveness add compiler-level semantics beyond primitive composition.

---

#### TYPE_INFERENCE

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Type annotations inside function bodies add noise without providing documentation value. The programmer must spell out types that are obvious from context, and type changes cascade through annotations. |
| Q2 | Is this a language problem or a library problem? | **Language.** Type inference is a compiler-level type-system service — it determines types from expression context. The bidirectional inference algorithm (top-down + bottom-up), generic type argument inference, and the annotation boundary rule all require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Type inference is the type system determining types from usage — this is a meta-level operation, not expressible via value-level primitives. The inference algorithm (unification, constraint solving) is a compiler service. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (annotations at API boundaries), Simplicity (inference inside functions reduces noise), Intent Over Implementation (programmer describes what, compiler resolves types). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Type inference is a compiler service — the compiler determines types that are not explicitly written. Type unification (comparison of type structures) depends on EQUALITY (EDR-017) semantics. Bidirectional inference flows are a semantic specification of how inference proceeds. |
| Q6 | Can it be expressed through composition? | No. Type inference is inherently compiler-level — the type system determines types from context. This is not expressible via composition of value-level primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Type inference determines the semantic type of an expression — it is a semantic operation, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic use of generics, lambda expressions, and local variable declarations. Without inference, Orthon would require type annotations on every expression — unacceptable for LLM readability and programmer productivity. |

**Classification per D-03:** Language. Compiler-level semantic service (local bidirectional inference). Type annotations required at public API boundaries. Depends on EQUALITY (EDR-017) for type unification.

**Primitive decomposition path:** Inferred type → compiler-determined, not primitive-expressible; type annotation at API boundary → `identifier` (type name) + `scope` (binding); type unification → `===` (structural equality per EDR-017) applied to type structures; generic argument inference → compiler constraint solving. The inference algorithm, constraint solving, and type unification are compiler-level services beyond primitive composition.

---

#### TYPE_LEVEL_NULL_SAFETY

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | After a pattern match on `Option<T>` that establishes the value is `Some(T)`, the programmer should not need to manually unbox. Without narrowing, every pattern match requires explicit `!` unwrap, defeating the ergonomic benefit of pattern matching. |
| Q2 | Is this a language problem or a library problem? | **Language.** Flow-sensitive type narrowing — tracking type information across control flow edges — requires compiler support. The narrowing rules (after match, after explicit check, per-variable, reset on reassignment) are compiler-level semantics. |
| Q3 | Can it be solved with existing primitives? | No. Flow-sensitive type analysis — tracking that a variable of type `Option<T>` is known to be `T` in a specific code path — is not expressible via the 9-primitive set. It requires the compiler to track type information across control flow edges. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (narrowing is compiler-enforced safety), Explicitness (narrowing follows visible checks), Minimal Core (narrowing eliminates manual unwrap calls without adding new syntax). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Flow-sensitive type narrowing — the compiler tracks per-variable type information across control flow edges. After `if value != None`, the compiler knows `value` is `T` in the true branch. This is a new type-system operation not present in existing constructs. |
| Q6 | Can it be expressed through composition? | No. Flow-sensitive type analysis is inherently compiler-level — the type system must track state across control flow. Not expressible via composition of value-level primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Type narrowing determines what operations are legal on a value — this is a semantic guarantee, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Builds on NULL_SAFETY (EDR-018). |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic null safety. Without narrowing, every `Some` match would require an explicit `!` — eliminating the ergonomic benefit of pattern matching. Makes safe code as concise as unsafe code. |

**Classification per D-03:** Language. Null safety tracked at type level (`Option<T>` vs `T`). Compiler tracks when a value is definitely non-null after a check. Depends on NULL_SAFETY (EDR-018).

**Primitive decomposition path:** Narrowed type → compiler-determined, not primitive-expressible; `match` narrowing → pattern matching (EDR-025) + compiler type tracking; `if` check narrowing → `function` (condition) + compiler type tracking across control flow edges; `!` escape hatch → `function` (unwrapping with panic contract). The flow-sensitive type tracking across control flow edges is a compiler-level analysis beyond primitive composition.

---

### Essential Core — Wave 2

#### AST_MACROS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Metaprogramming — code that writes code — without introducing multiple special-purpose sublanguages (macro\_rules!, proc macros, annotation processors). |
| Q2 | Is this a language problem or a library problem? | **Language.** AST macros operate on compiler-level AST type nodes. The `@macro` annotation, `@derive` sugar, hygienic scoping, and single-pass expansion require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. AST node manipulation (typed AST types, AST construction) is not expressible via primitive composition. The comptime execution engine (EDR-031) provides the runtime; the macro mechanism provides the structured AST-layer interface. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one macro mechanism replaces multiple sublanguages), Explicitness (`@macro` is syntactically visible), Orthogonality (macros compose freely with other constructs). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Typed AST node manipulation at compile time, hygienic scoping, single-pass expansion, `@derive` resolution. |
| Q6 | Can it be expressed through composition? | No. Compiler-level AST types and macro expansion ordering are not expressible via primitive composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `@derive` could desugar to `@macro` invocations, but the macro mechanism itself requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Macro expansion is a semantic operation (code generation), not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for metaprogramming. Eliminates manual code duplication for trait implementations. `@derive` alone justifies the mechanism. |

**Classification per D-03:** Language. Operate on parse tree at compile time, requiring compiler-level understanding. Builds on COMPILE\_TIME\_EXECUTION.

**Primitive decomposition path:** `@macro` function → `function` + comptime annotation (compiler-recognized); `@derive(Trait)` → compiler-resolved macro registry lookup; AST types → compiler-internal type system (not user-visible beyond macro API); hygienic scoping → compiler-enforced scope isolation. The macro registry, AST type contracts, and expansion ordering add compiler-level semantics beyond primitive composition.

---

#### COMPILER_AS_STATIC_ANALYZER

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | The line between compiler errors, warnings, and linter concerns must be explicitly defined. LLM-native design requires a single feedback channel for all static checks. |
| Q2 | Is this a language problem or a library problem? | **Language.** The compiler IS the analyzer — verification layers are part of the compiler pipeline, not an external tool. The existence and ordering of verification layers is a language specification concern. |
| Q3 | Can it be solved with existing primitives? | No. Verification layers (ownership, effects, exhaustiveness) require compiler-level semantic analysis not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (effects tracking requires declared effect boundaries), Declarative With Static Guarantees (compiler enforces correctness before runtime), LLM Readiness (single diagnostic channel). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Each verification layer adds compiler-enforced semantic guarantees: ownership, effect tracking, exhaustiveness, pattern-match completeness. |
| Q6 | Can it be expressed through composition? | No. Ownership analysis, effect tracking, and exhaustiveness checking are compiler-level analyses, not compositions of primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Verification establishes semantic correctness. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for a safe, LLM-native language. The compiler as single verification channel provides the fastest feedback loop. |

**Classification per D-03:** Language. Compiler provides static analysis API — the compiler IS the analyzer. Meta-concept — not a feature programmers invoke, but the architecture of how the compiler verifies correctness.

**Primitive decomposition path:** Not directly applicable — the static analyzer is the compiler itself. Verification layers are meta-operations on the compiler pipeline, not decomposable to user-visible primitives.

---

#### COMPILE_TIME_EXECUTION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Generics, reflection, and metaprogramming should not each require their own sublanguage. A single compile-time execution mechanism replaces four separate mechanisms. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `comptime` keyword, comptime parameter semantics, and comptime evaluation model require compiler support. The comptime interpreter is part of the compiler. |
| Q3 | Can it be solved with existing primitives? | No. Compile-time evaluation of the same language semantics is a new execution phase, not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one mechanism replaces four), Same Semantics Earlier Phase (no separate sublanguage), Explicit Semantics (`comptime` keyword makes phase visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Comptime evaluation phase, comptime parameters, type as first-class comptime value, deterministic sandboxed execution, `@typeInfo`/`@field`/`@hasDecl` reflection operations. |
| Q6 | Can it be expressed through composition? | No. A second execution phase (compile time vs. runtime) is a fundamental semantic addition, not a composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Comptime defines a new execution phase with semantic consequences (code runs at compile time vs. runtime produces different observable behaviour). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Unified comptime is the elegant answer to generics + reflection + metaprogramming. Cross-ref with GENERICS (Plan 04-02) — comptime IS the generic mechanism. LLM generability restrictions documented in concept doc. |

**Classification per D-03:** Language. Unified comptime model (Zig-inspired). Same semantics, earlier phase. Compiler-level execution mode. Cross-ref with GENERICS (Plan 04-02).

**Primitive decomposition path:** Comptime parameter → `function` parameter + comptime annotation; comptime block → `scope` + comptime annotation; `@typeInfo` → comptime-evaluated `call` to compiler intrinsic; monomorphisation → compiler specialization of `function` + types. The comptime execution phase and evaluation engine add compiler-level semantics beyond primitive composition.

---

#### COMPOSABLE_COLLECTION_OPS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manual index-based loops, empty accumulator lists, and explicit search flags force the programmer to describe *how* instead of *what*. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** `.map()`, `.filter()`, `.reduce()` are compositions of `Iterator[T].next()` calls. No new language semantics required — the Iterator Protocol (EDR-022) provides everything. |
| Q3 | Can it be solved with existing primitives? | Yes. Each combinator is implementable as a method on `Iterator[T]` using existing `function`, `call`, `scope`, and `pack`/`unpack` primitives. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (StdLib is the right home), Intent Over Implementation (declarative combinators over imperative loops). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Each combinator is a function composition. Loop fusion is an optimisation, not semantics. |
| Q6 | Can it be expressed through composition? | Yes — of `Iterator[T].next()` calls. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — combinators are function calls, fully expressible via primitive operations. |
| Q8 | Is this an optimisation, not semantics? | The operations themselves are semantic (map, filter, reduce). Loop fusion (combining multiple passes) is a pure optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Declarative collection operations are essential for any practical language. StdLib classification means zero language additions. |

**Classification per D-03:** StdLib. Combinators are compositions of ITERATOR\_PROTOCOL operations. Note compiler-level optimization (loop fusion) is an Implementation Strategy concern.

**Primitive decomposition path:** Each combinator (`map`, `filter`, `fold`, etc.) → `function` implementation on `Iterator[T]` trait + `call` to `next()` + `scope` + `pack`/`unpack` for result construction + `function` (closure parameter). Fully expressible via primitive composition — no new compiler semantics.

---

#### CONCURRENCY_MODEL

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Concurrent execution without shared mutable state, data races, or deadlocks. How does Orthon extend its "no shared mutable state" safety guarantee to parallel execution? |
| Q2 | Is this a language problem or a library problem? | **Language.** The `act` modifier, `delegate` keyword, `<-` message operator, and ownership transfer rules require compiler support. StdLib concurrency utilities (channels, timers) are a separate concern (Plan 04-06). |
| Q3 | Can it be solved with existing primitives? | No. The `act` modifier changes type semantics (isolated state, message-passing interface). The `<-` operator introduces message-queue semantics. Ownership transfer across isolation boundaries requires compiler-enforced rules. |
| Q4 | Does it violate any Design Principle? | No. Aligns with "no shared mutable state" (core principle), Explicit Semantics (`act`, `delegate`, `<-` are syntactically visible), Orthogonality (concurrency model composes with traits, error handling, ownership). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Delegate isolation, message-passing execution, single-threaded per-delegate processing, automatic parallelism from independence, ownership transfer across boundaries, error propagation across delegates. |
| Q6 | Can it be expressed through composition? | No. Delegate isolation and message-passing semantics require compiler-level support — the compiler must enforce that no two delegates share mutable memory. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Concurrency semantics define how programs execute in parallel. Scheduling (work-stealing vs. pinned-to-thread) is an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language targeting multi-core processors. The delegate model provides data-race freedom by construction. Cross-ref with ERROR_HANDLING (EDR-020) and TRAITS (EDR-019). Cross-ref with CONCURRENCY (Plan 04-06) for downstream StdLib utilities. |

**Classification per D-03:** Language. Core semantic dimension (how concurrent execution is defined). Compiler-level guarantees (data-race freedom, isolation). Delegate-based model, `act` modifier, no shared-state threads.

**Primitive decomposition path:** `act` modifier → type declaration modifier (compiler-enforced isolation semantics); `delegate` → `reference` + isolated `scope` + message queue; `<-` operator → compiler-recognized message-send syntax; ownership transfer (`$`) → existing `reference` + ownership semantics across boundaries. The isolation guarantee, message ordering, and single-threaded processing per delegate add compiler-level semantics beyond primitive composition.

---

### Essential — Policy Level

The following concepts run through the Decision Pipeline but are classified as **Policy** (not Language) per D-04. They do not add new semantics expressible via primitives — they are implementation choices about HOW primitives are realised. They are routed to `how/strategies/` area, NOT `what/concepts/`.

#### ALLOCATION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How and when is memory allocated for program data? Allocation affects performance predictability, memory safety, and overall program design. |
| Q2 | Is this a language problem or a library problem? | **Policy.** Allocation is how language semantics are realised in memory — an implementation choice, not a language feature. |
| Q3 | Can it be solved with existing primitives? | Yes — allocation is the materialisation of existing primitives (`literal`, `pack`, `identifier`, `reference`). The programmer declares data structures; allocation is implicit. |
| Q4 | Does it violate any Design Principle? | No. Allocation as Policy aligns with Minimal Core (allocation mechanism is an implementation detail), Intent Over Implementation (programmer declares data; compiler decides allocation), Implementation Independence (allocation strategy is interchangeable). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Allocation is the realisation of existing primitives — it does not add new semantic operations. |
| Q6 | Can it be expressed through composition? | N/A — Policy is about how primitives are realised, not composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — allocation is purely an implementation/optimisation concern. Language semantics are allocation-agnostic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical implementation. Strategy system must define allocation choices. |

**Classification per D-04:** Policy. Allocation is an Implementation Policy — how data is materialised in memory, not what data means.

**EDR:** [EDR-034](../decision_records/architecture/EDR-034-allocation.md)

#### REGION_BASED_MEMORY_MANAGEMENT

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Predictable, efficient memory deallocation without garbage collection. How to manage memory in bulk by scoped regions (arenas). |
| Q2 | Is this a language problem or a library problem? | **Policy.** Region-based allocation is a sub-policy within the Allocation Policy — a specific implementation choice for how Arena allocation works. |
| Q3 | Can it be solved with existing primitives? | Yes — region allocation is a materialisation strategy for the `pack` and `reference` primitives. The programmer never writes arena management code. |
| Q4 | Does it violate any Design Principle? | No. Region allocation as Policy aligns with Intent Over Implementation (arenas are an invisible optimisation), Minimal Core (arena mechanism is implementation detail). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Region allocation refines how Arena policy materialises memory — it does not add new language semantics. |
| Q6 | Can it be expressed through composition? | N/A — Policy classification. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — region allocation is an implementation optimisation (bulk deallocation, bump allocation). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Provides the default allocation strategy (Arena) with three scoping modes: ScopeRegion, ExplicitRegion, NoRegion. |

**Classification per D-04:** Policy. Sub-policy of Allocation Policy (EDR-034). Refines Arena allocation with lifetime-scoping strategies.

**EDR:** [EDR-035](../decision_records/architecture/EDR-035-region-based-memory-management.md)

#### EXECUTION_PROGRAM

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | The fragmented toolchain problem — compilation, packaging, and deployment are separate stages with incompatible formats. The same information is described multiple times (Cargo.toml, Dockerfile, Kubernetes manifests, CI config). |
| Q2 | Is this a language problem or a library problem? | **Policy.** Execution model is an implementation choice — how a program is run, not what the program means. Introduces a new Policy type (Execution Model Policy). |
| Q3 | Can it be solved with existing primitives? | Yes — execution model does not add new language semantics. Core concepts (equality, ownership, pattern matching) are unchanged regardless of execution strategy. |
| Q4 | Does it violate any Design Principle? | No. Execution Program aligns with Intent Over Implementation (programmer declares what; infrastructure decides how), SOLID (single responsibility, dependency inversion), Minimal Core (execution is infrastructure, not language). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Execution model is about packaging and running — it does not change what programs mean. |
| Q6 | Can it be expressed through composition? | N/A — Policy classification. |
| Q7 | Can it be syntactic sugar over existing primitives? | N/A — Policy classification. |
| Q8 | Is this an optimisation, not semantics? | Yes — execution strategy is a build-time/deployment concern, not language semantics. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** This is Orthon's core innovation — decoupling semantics from execution strategy. The Execution Program model eliminates toolchain fragmentation. |

**Classification per D-04:** Policy. New Execution Model Policy type in the Strategy system. Execution is infrastructure, not language.

**EDR:** [EDR-036](../decision_records/architecture/EDR-036-execution-program.md)

---

### Essential — Derived Features (Wave 3)

The following borderline concepts were evaluated per D-03 classification rules. Both resolved as corrections to existing documents rather than standalone Language features.

#### CONTEXT_PARAMETERS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Plumbing execution environment objects (logger, database connection, configuration) through call chains without explicit threading at every intermediate call site. |
| Q2 | Is this a language problem or a library problem? | **Language.** Context resolution requires compiler support for type-directed `given` resolution and lexical scoping rules — not implementable as a library. However, the mechanism is a cross-cutting concern of the Evaluation and Visibility dimensions, not a standalone feature. |
| Q3 | Can it be solved with existing primitives? | No — implicit parameter threading is not expressible via the 9-primitive set. It requires compiler-level parameter resolution. |
| Q4 | Does it violate any Design Principle? | No. Context parameters align with Explicitness (`using` block is visible in signature), Intent Over Implementation (programmer declares context need; compiler resolves it). However, implicit resolution risks violating Transparency (where did this `given` come from?). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | Yes — context parameters add implicit-passing semantics with static resolution. However, these are a correction to the Evaluation dimension (context supply timing) and Visibility dimension (`given` scope resolution), not a standalone feature. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No — parameter passing is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes, but deferred.** Context parameters solve a real ergonomic problem. However, for v0.1, a SEMANTIC_MODEL correction acknowledging context flow as a cross-cutting concern is sufficient. Full specification deferred beyond v0.1. |

**Classification per D-03:** SEMANTIC_MODEL correction. Cross-cutting concern of Evaluation and Visibility dimensions. Not a standalone Language feature. See [EDR-037](../decision_records/architecture/EDR-037-context-parameters.md).

#### REPRESENTATION_MODIFIERS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How to control how values of a type are stored in memory (inline, boxed, packed, FFI-compatible) without changing the type's semantic identity. |
| Q2 | Is this a language problem or a library problem? | **Primitive-level.** Representation modifiers are annotations on existing primitives (`pack` for inline/struct, `reference` for indirection/boxed) — they are not new operations requiring separate language status. |
| Q3 | Can it be solved with existing primitives? | Yes — representation modifiers are orthogonal annotations on the `pack` primitive (struct, packed) and the `reference` primitive (boxed, shared, atomic). They add constraints to how primitives are materialised, not new operations. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (modifier syntax is visible), Orthogonality (modifiers compose with any type), Minimal Core (modifiers are annotations, not new primitives). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | The modifiers add annotation semantics (storage strategy selection), but these are constraints on how existing primitives are realised — not new primitive operations. |
| Q6 | Can it be expressed through composition? | Yes — `struct(T)` = `pack` without runtime metadata; `boxed(T)` = `reference` to heap allocation. The modifiers are composition annotations. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — each modifier desugars to a constraint on how `pack` or `reference` is realised. |
| Q8 | Is this an optimisation, not semantics? | Partially — storage selection is an implementation concern (how to lay out data). The modifier syntax itself is language-level, but the semantics (what operations are legal on the value) are unchanged. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes, but deferred.** Representation modifiers solve a genuine problem. For v0.1, a PRIMITIVE_BLOCKS correction noting that representation annotations are orthogonal modifiers on primitives is sufficient. Full specification deferred beyond v0.1. |

**Classification per D-03:** PRIMITIVE_BLOCKS correction. Representation modifiers are orthogonal annotations on existing primitives (`pack` and `reference`), not new primitives or a standalone Language feature. See [EDR-038](../decision_records/architecture/EDR-038-representation-modifiers.md).
