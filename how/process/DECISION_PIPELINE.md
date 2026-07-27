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

---

### Important Tier — Wave 4

#### ASYNC_AWAIT + ASYNC_AS_EXPLICIT_MODIFIER (combined)

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How to perform blocking I/O without blocking the OS thread. Async/await transforms imperative code into state machines, letting the programmer write straight-line code while the compiler manages suspend/resume. |
| Q2 | Is this a language problem or a library problem? | **Language.** Async modifies function/proc/new semantics. The compiler must transform async functions into state machines. The `async` modifier, `await` keyword, `spawn`, and `scope` all require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. State machine transformation (coroutine compilation) is not expressible via the primitive set. `Future` as first-class type requires compiler-level suspension tracking. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (`async`/`await`/`spawn`/`scope` are syntactically visible), Minimal Core (modifier on existing kinds, not a new category), Orthogonality (async composes with `proc`/`fun`/`new`). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Stackless coroutine state machine, `await` suspension points, `Future` as first-class type, `spawn` for parallelism, `scope` for structured concurrency. |
| Q6 | Can it be expressed through composition? | No. Coroutine transformation is a compiler-level operation. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Async defines new execution semantics (suspension, resumption). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language doing I/O. Async/await is the proven model (Rust, JS, Python, C#). |

**Classification per D-03:** Language. Async as orthogonal modifier on `proc`/`fun`/`new`. Compiler-level state machine transformation. Colourless model with Future as first-class value.

**EDR:** [EDR-047](../decision_records/architecture/EDR-047-async-await.md)

---

#### CONCURRENCY

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Concrete concurrent execution patterns building on the delegate model — typed channels, select, supervision, timers, fan-out/fan-in. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Channels wrap delegate mailboxes. Select is a macro/function. Supervision is a delegate that monitors other delegates. All are implementable using existing constructs (EDR-033). |
| Q3 | Can it be solved with existing primitives? | Yes — channels = delegate + mailbox; select = polling multiple channels; supervision = delegate lifecycle management; timers = delegates on a scheduler. |
| Q4 | Does it violate any Design Principle? | No. StdLib classification preserves Minimal Core. All utilities are ordinary Orthon code. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** All utilities are function/method implementations, not new language constructs. |
| Q6 | Can it be expressed through composition? | Yes — of `delegate`, `<-`, `$`, and existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all concurrency utilities are ordinary function calls. |
| Q8 | Is this an optimisation, not semantics? | The utilities themselves are not optimisations. Scheduling (work-stealing, pinned-to-thread) is an optimisations. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Channels and select are essential for coordinating work across delegates. StdLib classification means zero language additions. |

**Classification per D-03:** StdLib. Concrete async/concurrent patterns building on CONCURRENCY_MODEL (EDR-033). Channels, select, supervision, timers — all StdLib.

**EDR:** [EDR-049](../decision_records/architecture/EDR-049-concurrency.md)

---

#### GENERATORS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Producers may need bidirectional communication with consumers (consumer sends values back to producer). Simple lazy sequences need concise inline syntax (generator expressions). |
| Q2 | Is this a language problem or a library problem? | **Language.** `yield` is a bidirectional variant of `emit`, requiring compiler-level state machine modification (consumer send values). Generator expressions require compiler-level desugaring to generator functions. `yield from` requires delegation semantics. |
| Q3 | Can it be solved with existing primitives? | No. Two-way communication via `yield` requires the state machine to accept consumer values — a new semantic dimension beyond EDR-021's one-way `emit`. |
| Q4 | Does it violate any Design Principle? | No. Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). Bidirectional yield preserves all existing invariants from EDR-021. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Bidirectional yield adds consumer-to-producer communication. Generator expressions add concise syntax. `yield from` adds delegation. |
| Q6 | Can it be expressed through composition? | No — bidirectional state machine communication is not expressible via composition of primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Generator expressions are sugar over generator functions. Bidirectional yield requires state machine changes. |
| Q8 | Is this an optimisation, not semantics? | No. Two-way communication is a semantic extension of the generator model. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Bidirectional yield enables interactive coroutine patterns. Generator expressions provide concise inline sequence syntax. Both are proven in Python. |

**Classification per D-03:** Language. Bidirectional yield adds consumer-to-producer communication semantics beyond EDR-021. Generator expressions require compiler desugaring.

**EDR:** [EDR-050](../decision_records/architecture/EDR-050-generators.md)

---

#### PUSH_STREAMS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Push-based reactive streams — the dual of pull-based generators. The producer determines when values arrive; the consumer reacts. Needed for events, I/O, GUI, sensors, message brokers. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Push streams are implementable using the delegate model (EDR-033) and channels (EDR-049). The `Stream<T>` type wraps a delegate + channel; combinators are function compositions. |
| Q3 | Can it be solved with existing primitives? | Yes — stream = delegate + channel; subscription = callback registration; backpressure = channel capacity; combinators = function compositions. |
| Q4 | Does it violate any Design Principle? | No. StdLib classification preserves Minimal Core. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Streams are ordinary Orthon types with method implementations. |
| Q6 | Can it be expressed through composition? | Yes — of delegates, channels, closures, and function calls. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all stream operations are method calls. |
| Q8 | Is this an optimisation, not semantics? | No. Stream semantics (push, subscribe, complete, error) are StdLib patterns, not optimisations. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Push streams address a universal need (event-driven programming). StdLib classification means zero language additions. |

**Classification per D-03:** StdLib. Push-based reactive streams built on delegate model. No new language constructs.

**EDR:** [EDR-051](../decision_records/architecture/EDR-051-push-streams.md)

---

#### EMIT_AS_INTERMEDIATE_RESULT

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | A function should be able to publish intermediate results during long-running computation while maintaining a final return value — without separate stream or callback abstractions. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `emit` keyword already exists (EDR-021). This is a semantic refinement clarifying that `emit` serves both lazy sequence production AND intermediate result publication. Adds `.final()` accessor specification. |
| Q3 | Can it be solved with existing primitives? | Partially — the `emit` mechanism (EDR-021) already supports the pattern technically. The refinement is in specification language and the `.final()` accessor. |
| Q4 | Does it violate any Design Principle? | No. The model is consistent with EDR-021. Adds clarity without changing core semantics. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **Semantic refinement.** No new runtime semantics — the compiler already supports this pattern. The `.final()` accessor is a minor extension. The change is to specification language: functions with `emit` can now document both intermediate and final results. |
| Q6 | Can it be expressed through composition? | Yes — the `emit` + `return` pattern is already supported by EDR-021. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — this is a specification refinement of an existing mechanism. |
| Q8 | Is this an optimisation, not semantics? | No. The refinement clarifies semantics that already exist. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Clarifying the dual use of `emit` (lazy sequence + intermediate result) improves specification clarity and programmer understanding. The `.final()` accessor fills a gap in EDR-021's model. |

**Classification per D-03:** Language (semantic refinement). Builds on LAZY_SEQUENCE_GENERATORS (EDR-021). Clarifies that `emit` serves both lazy sequence production and intermediate-result publication.

**EDR:** [EDR-052](../decision_records/architecture/EDR-052-emit-as-intermediate-result.md)

---

#### ITERATION_LOOP

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | How does a language express repeated execution over a sequence of values? The `for`/`while` loop constructs are fundamental to any language. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `for`, `while`, and `loop` keywords require compiler-level syntax, desugaring, and control-flow (break, continue). `for` desugars to ITERATOR_PROTOCOL (EDR-022). |
| Q3 | Can it be solved with existing primitives? | No — loop constructs require compiler-level syntax and control-flow. `for` desugaring requires compiler recognition of `IntoIterator[T]`. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one iteration construct + `while`), Explicit Semantics (`break`/`continue` are visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Loop control flow (break, continue), `for` desugaring to iterator protocol, `while` condition evaluation, `loop` as infinite loop. |
| Q6 | Can it be expressed through composition? | No — loop desugaring requires compiler support. |
| Q7 | Can it be syntactic sugar over existing primitives? | `for` desugars to `IntoIterator` + `loop` + `match` + `next()`. The desugaring itself requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Loop semantics are fundamental control flow. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. The simplest, safest iteration model. |

**Classification per D-03:** Language. `for`/`while`/`loop` constructs require compiler-level syntax and desugaring. `for` desugars to ITERATOR_PROTOCOL (EDR-022). No C-style `for (;;)`.

**EDR:** [EDR-053](../decision_records/architecture/EDR-053-iteration-loop.md)

---

#### OBJECT_INITIALIZATION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Creating objects with many optional fields without telescoping constructors or boilerplate builder patterns. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Named parameters and default values are already part of Orthon's general function call model. Copy-and-update syntax is sugar over `new` + field assignment. Builder auto-generation is an AST macro (EDR-029). No new language semantics. |
| Q3 | Can it be solved with existing primitives? | Yes — named parameters are already in the function call model. Default values are already in the type declaration model. Copy-and-update desugars to `new` + field assignment. Builder is a macro. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (uses existing mechanisms), Explicit Semantics (named parameters are visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** All patterns are sugar over existing language mechanisms. |
| Q6 | Can it be expressed through composition? | Yes — of existing function call, type declaration, and macro mechanisms. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — copy-and-update desugars to `new` + field assignments. Builder is a macro. |
| Q8 | Is this an optimisation, not semantics? | No — initialization is a StdLib pattern, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Solves a genuine ergonomic problem (telescoping constructors, builder boilerplate). Zero language cost. |

**Classification per D-03:** StdLib. Constructor patterns, builder patterns, copy-and-update syntax — all StdLib / macro features. No new language semantics.

**EDR:** [EDR-054](../decision_records/architecture/EDR-054-object-initialization.md)

---

#### UNPACKING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Concise extraction of values from compound data structures without verbose indexing. Explicit indexing is brittle and non-obvious. |
| Q2 | Is this a language problem or a library problem? | **Language.** Destructuring patterns require compiler-level parsing and desugaring to `pack`/`unpack` primitives. The syntax mirrors PATTERN_MATCHING (EDR-025). |
| Q3 | Can it be solved with existing primitives? | Semantically, yes — `unpack(point)` already exists as a primitive. But destructuring syntax requires compiler-level pattern recognition and code generation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Representation Symmetry (construction and destruction share syntax), Minimal Core (desugars to existing primitives). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **Syntactic sugar.** All destructuring forms desugar to `pack`/`unpack` primitives. No new runtime semantics — only syntactic transformation. |
| Q6 | Can it be expressed through composition? | Yes — `let (x, y) = point` desugars to `let x, y = unpack(point)`. |
| Q7 | Can it be syntactic sugar over existing primitives? | **Yes** — destructuring is syntax over `pack`/`unpack`. |
| Q8 | Is this an optimisation, not semantics? | No. Destructuring is a syntactic convenience, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic data decomposition. Eliminates boilerplate indexing. Symmetric with `pack` construction syntax. |

**Classification per D-03:** Language. Destructuring syntax matching pack/unpack symmetry (PRIMITIVE_BLOCKS). Compiler must resolve destructuring patterns. However, all forms desugar to `pack`/`unpack` primitives — no new runtime semantics.

**EDR:** [EDR-055](../decision_records/architecture/EDR-055-unpacking.md)

---

### Important Tier — Wave 4

The following 8 important-tier concepts were processed through the Decision Pipeline. See individual EDRs for Gate Validation and detailed reasoning.

#### ALGEBRAIC_DATA_TYPES (combined with ENUM_ALTERNATIVES)

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Data that takes one of several known forms needs language-level support. Without ADTs, programmers use runtime casts (unsafe), inheritance (non-exhaustive), or manual tagged unions (error-prone). Enum alternatives (named constants, iota) are subsumed by ADTs. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `type Name = Variant(fields) | Variant(fields)` declaration form, automatic discriminant generation, recursive type checking, and sealed-variant exhaustiveness require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Combined variant+field declaration (sum+product in one form) is not expressible via manual sealed trait + variant type declarations. Automatic discriminant generation is a new compiler operation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one sum-type mechanism replaces two: ADTs + enums), Orthogonality (variants compose freely with traits, generics, pattern matching), Explicitness (type declaration syntax is visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Combined variant+field declaration, automatic discriminant (tag), sealed exhaustiveness, recursive type termination checking. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — ADT declaration could desugar to sealed trait + variant types, but automatic discriminant generation and recursive type checking require compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Sum types are a semantic concept — what values a type can hold. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Builds on TRAITS (EDR-019) and PATTERN_MATCHING (EDR-025). |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language. ADTs are the proven mechanism for type-safe sum types (Rust, Haskell, OCaml, Swift, Kotlin sealed classes). Subsumes enum construct per "One concept — one syntax." |

**Classification per D-03:** Language. ADTs add new semantics (combined sum/product declaration, automatic discriminant, sealed exhaustiveness) beyond TRAITS + PATTERN_MATCHING.

**EDR:** [EDR-039](../decision_records/architecture/EDR-039-algebraic-data-types.md) — combined with ENUM_ALTERNATIVES (no separate EDR-040; ADTs subsume dedicated enums).

---

#### COLLECTION_LITERAL_SYNTAX

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Creating a collection and manually adding each element is verbose and encourages mutation. `[1, 2, 3]` is universally preferred over `list = List(); list.add(1); list.add(2); list.add(3)`. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** A collection literal desugars to a constructor call (e.g., `[1, 2, 3]` → `List(1, 2, 3)`). No new compiler-level semantics — the language already supports `literal`, `pack`, and `call` primitives. |
| Q3 | Can it be solved with existing primitives? | Yes — each element is a `literal` or `identifier`, and the literal as a whole is a `call` to a collection constructor. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (StdLib classification), Intent Over Implementation (declare what, not how). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Collection literals are syntactic sugar for constructor calls. |
| Q6 | Can it be expressed through composition? | Yes — of `literal` elements + `call` to collection constructor. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — `[1, 2, 3]` desugars to `List(1, 2, 3)`, fully expressible via primitive operations. |
| Q8 | Is this an optimisation, not semantics? | The literal itself is syntax. Large-literal desugaring (builder pattern vs. variadic constructor) is an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Syntax deferred to Phase 5. |
| Q10 | Is it worth adding at all? | **Yes.** Universal ergonomic expectation. StdLib classification means zero language additions. Syntax deferred to Phase 5. |

**Classification per D-03:** StdLib. Syntactic sugar for collection constructors. Syntax deferred to Phase 5.

**EDR:** [EDR-041](../decision_records/architecture/EDR-041-collection-literal-syntax.md)

---

#### DATACLASSES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Boilerplate for passive data carriers — manual constructors, accessors, equality, hashing, string representation. `@derive(init, eq, repr, hash)` eliminates this. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** The `@derive` mechanism (EDR-029) already provides compile-time code generation. Dataclasses are a specific application of derives — no new language semantics. The `with` expression is a limited compiler intrinsic. |
| Q3 | Can it be solved with existing primitives? | Yes — `@derive` (EDR-029) applied to a type declaration generates implementations for `init` (constructor), `eq` (structural equality per EDR-017), `repr` (string representation), `hash` (hashing). |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (reuses existing derive mechanism), Explicitness (`@derive` annotations are visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics** for `@derive(init, eq, repr, hash)` — all derives reuse existing mechanisms. The `with` expression adds limited new semantics (compiler-recognized copy-with-modify intrinsic). |
| Q6 | Can it be expressed through composition? | Yes — each derive target maps to a registered macro function. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — `@derive(init)` desugars to a `@macro` invocation per EDR-029. |
| Q8 | Is this an optimisation, not semantics? | No — code generation is semantic (constructor, equality, etc.). But uses existing mechanisms. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most common boilerplate in any codebase. Reuses existing `@derive` mechanism — no new keywords. |

**Classification per D-03:** StdLib. Dataclass pattern via existing `@derive` mechanism (EDR-029). No dedicated keyword.

**EDR:** [EDR-042](../decision_records/architecture/EDR-042-dataclasses.md)

---

#### LITERAL_TYPES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Modelling closed sets of string/number values without ADT declaration ceremony. A literal value `"GET"` is its own type, composing via `|` into `type Method = "GET" | "POST" | "PUT"`. |
| Q2 | Is this a language problem or a library problem? | **Language.** The compiler must track literal types as singleton types in the type system, apply widening rules (immutable preserves, mutable widens), and support narrowing in pattern matching. |
| Q3 | Can it be solved with existing primitives? | No. The concept of a value being its own type is not expressible via the 9-primitive set. Singleton type tracking is a type-system operation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (widening rule is one explicit rule), Minimal Core (one widening rule covers all cases). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Singleton types for literal values, widening rule, narrowing in pattern matching. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No. Type tracking is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for API boundaries and protocol constants (HTTP methods, status codes). Composes with union types (EDR-045) for closed-set modelling. |

**Classification per D-03:** Language. Values as types require compiler-level literal type tracking, widening, and narrowing.

**EDR:** [EDR-043](../decision_records/architecture/EDR-043-literal-types.md)

---

#### STRUCTURAL_TYPING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Polymorphism without explicit declaration ceremony. Marker traits (`Show`, `Default`) require mechanical `impl` blocks even when the method shape is obvious from the type's structure. |
| Q2 | Is this a language problem or a library problem? | **Language.** Structural trait resolution — checking method signatures at compile time — requires compiler support. The `structural` keyword on trait declarations is a new syntactic form with semantic consequences for trait resolution. |
| Q3 | Can it be solved with existing primitives? | No. Structural method signature matching across types requires the type system to compare method shapes without an explicit `impl` declaration — not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (type satisfies trait structurally without explicit `impl`), Explicitness (`structural` keyword makes the mode visible). Nominal-by-default preserves explicitness for semantically meaningful traits. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Structural matching: the compiler checks method signatures for compatibility without an explicit `impl` declaration. Priority rules: explicit `impl` > structural match. Ambiguity detection for conflicting structural matches. |
| Q6 | Can it be expressed through composition? | No. Structural signature matching is a type-system operation. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Trait satisfaction determines what operations are legal on a type — this is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Builds on TRAITS (EDR-019). |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates ceremony for marker traits. Nominal-by-default maintains explicitness for semantically meaningful contracts. |

**Classification per D-03:** Language. Structural trait resolution adds compiler-level semantics (method signature matching, priority rules, conflict resolution) beyond nominal TRAITS (EDR-019).

**EDR:** [EDR-044](../decision_records/architecture/EDR-044-structural-typing.md)

---

#### UNION_INTERSECTION_TYPES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Ad-hoc composition of alternative types at point of use — `String | Int` for an ID that can be either — without ADT declaration ceremony. |
| Q2 | Is this a language problem or a library problem? | **Language.** The `|` combinator introduces a new type former — an anonymous, untagged union of two or more types. Compiler must resolve union type operations (member checking, narrowing, assignment compatibility). |
| Q3 | Can it be solved with existing primitives? | No. The concept of an anonymous union type is not expressible via the 9-primitive set. It requires new type-system operations. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Orthogonality (union types compose freely), Minimal Core (one combinator — `|` — for all union needs). Overlap with ADTs is accepted as different-level solution (ad-hoc vs. declared). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Anonymous union type formation, narrowing via match/is, structural assignment compatibility. No exhaustiveness — unlike ADTs. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No. Type formation is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ad-hoc composition of alternative types. Restricted to named types (no anonymous structural shapes) limits complexity. Intersection types rejected for v0.1. |

**Classification per D-03:** Language. Union types add new type-system combinator (`A | B`) with narrowing semantics. Intersection types NOT accepted for v0.1.

**EDR:** [EDR-045](../decision_records/architecture/EDR-045-union-intersection-types.md)

---

#### TYPE_LEVEL_COMPUTATION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Deriving new types from existing types — creating DTO types that omit sensitive fields, making all fields optional, extracting property names as a type — without manual parallel type declarations. |
| Q2 | Is this a language problem or a library problem? | **Language (restricted intrinsic set).** The compiler must provide built-in type manipulation operations (`KeyOf`, `Pick`, `Omit`, `Partial`, etc.) — these are not implementable as library functions since they operate on type structures, not values. |
| Q3 | Can it be solved with existing primitives? | No. Type-level operations that transform type shapes are not expressible via value-level primitives. |
| Q4 | Does it violate any Design Principle? | No, with the restricted intrinsic set. A full type-level language would violate Minimal Core and LLM Generability. A closed set of 8 documented intrinsics aligns with Explicit Semantics (each has a fixed, documented meaning). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics (intrinsics).** Each intrinsic defines a type-level transformation — new semantics for the type system. However, the set is closed and non-recursive. |
| Q6 | Can it be expressed through composition? | No — type-level transformations are not expressible via value-level primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Type transformation is semantic — it changes what types exist and how they relate. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes, with the restricted intrinsic set.** The closed set covers the essential DTO-shaping use cases. No Turing-complete type-level language — eliminates compiler-hang failure modes. Derive/macro mechanism is the escape hatch for custom operations. |

**Classification per D-03:** Language (restricted closed intrinsic set). Non-recursive compiler intrinsics for type transformation. NO user-extensible type-level programming language.

**EDR:** [EDR-046](../decision_records/architecture/EDR-046-type-level-computation.md)

---

### Important Tier — Wave 4 (Batch 2)

The following 12 important-tier concepts were processed through the Decision Pipeline. See individual EDRs for Gate Validation and detailed reasoning.

#### SLOTS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Fixed-field storage for types — preventing accidental attribute creation, ensuring ABI-stable layout, and eliminating per-instance dictionary overhead. |
| Q2 | Is this a language problem or a library problem? | **Language.** Fixed-field verification requires compiler-level type checking — accessing an undeclared field must be a compile-time error. |
| Q3 | Can it be solved with existing primitives? | No. Compile-time field-set restriction is not expressible via the 9-primitive set — it requires the compiler to know the declared field set. |
| Q4 | Does it violate any Design Principle? | No. Fixed fields as default aligns with Explicitness (type declaration defines exact fields), Safety (typos caught at compile time). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new runtime semantics** — slots are compile-time verification. The `dynamic` modifier is an annotation on existing type declaration syntax. |
| Q6 | Can it be expressed through composition? | Yes — fixed fields = type declaration with compile-time field verification. Dynamic = type declaration without verification. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — slot restriction is a compile-time gate over existing `attribute access` primitive. |
| Q8 | Is this an optimisation, not semantics? | The slot layout is an ABI optimisation. The compile-time field verification is a safety gate, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Fixed fields as default is the simplest, safest model. |

**Classification per D-03:** Language. Fixed-field verification is compiler-level type checking. Dynamic modifier for opt-out expressibility.

**EDR:** [EDR-063](../decision_records/architecture/EDR-063-slots.md)

---

#### SPAN

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Safe, non-owning access to contiguous memory — preventing dangling pointers, bounds violations, and implicit copies. |
| Q2 | Is this a language problem or a library problem? | **Language.** Lifetime tracking, bounds checking, and slice syntax require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Lifetime tracking to prevent dangling — the span must not outlive backing storage — requires compiler-level borrow checking. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Safety (bounds-checked access), Explicitness (slice syntax is visible), Minimal Core (one concept replaces pointer+length, copy-on-slice, unsafe casts). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Lifetime-tracked borrowed view, bounds-checked access guarantee, sub-span slicing. |
| Q6 | Can it be expressed through composition? | No — lifetime tracking through borrow checking is a compiler-level analysis. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Safe memory access is a semantic guarantee (no dangling, no UB). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for safe, efficient memory view operations. |

**Classification per D-03:** Language. Lifetime-tracked, bounds-checked memory view. Compiler-level borrow-checking integration required.

**EDR:** [EDR-064](../decision_records/architecture/EDR-064-span.md)

---

#### NAMED_AND_OPTIONAL_PARAMETERS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Call-site readability for functions with many parameters — boilerplate overload explosion for optional parameters. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Named arguments desugar to positional calls. Default values are ordinary expressions. The macro system (EDR-029) handles the transformation. |
| Q3 | Can it be solved with existing primitives? | Yes — named args = `call` with argument-name matching (desugars to positional `call`). Defaults = `function` parameter + initializer expression. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (no new syntax), Explicitness (defaults are visible in signature). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Sugar over existing function call mechanism. |
| Q6 | Can it be expressed through composition? | Yes — of `call` primitive + `identifier` (argument name) + desugaring. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — named args desugar to positional `call`. Defaults are initializer expressions. |
| Q8 | Is this an optimisation, not semantics? | No — call syntax is syntactic, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for ergonomic APIs with many parameters. |

**Classification per D-03:** StdLib. Sugar over existing function call mechanism. Macro-based desugaring.

**EDR:** [EDR-065](../decision_records/architecture/EDR-065-named-and-optional-parameters.md)

---

#### SORTING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Sorting stability — multi-key sort pipelines require predictable relative ordering of equal elements. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Sort algorithms are method implementations on collection types. Algorithm selection is an Implementation Policy. |
| Q3 | Can it be solved with existing primitives? | Yes — sort = `function` on collection + `call` to `Ord` comparison. Algorithm choice is implementation. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (StdLib is the right home), Deterministic Behavior (stability guaranteed). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Sort is a function composition. Stability guarantee is a specification property, not new semantics. |
| Q6 | Can it be expressed through composition? | Yes — of comparison operations (`==`, `<` per EDR-017) + collection reordering. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — sort methods are function calls. |
| Q8 | Is this an optimisation, not semantics? | Algorithm selection is an optimisation. The sort operation itself is semantics (ordering). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for any practical language — sorting is universal. |

**Classification per D-03:** StdLib. Sort algorithm as method implementation. Algorithm selection is Implementation Policy.

**EDR:** [EDR-066](../decision_records/architecture/EDR-066-sorting.md)

---

#### DECLARATIVE_MULTI_KEY_SORT

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manual comparator chain boilerplate for multi-field sorting with mixed directions. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Declarative multi-key sort is sugar over SORTING + EQUALITY. Tuple-as-key lexicographic comparison uses existing `Ord` trait. |
| Q3 | Can it be solved with existing primitives? | Yes — `sorted(by: [.a, .b])` desugars to lexicographic comparison using `Ord` + tuple comparison. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (specify sort keys, not comparators). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Sugar over existing `Ord` trait comparisons. |
| Q6 | Can it be expressed through composition? | Yes — of `Ord` comparisons on tuples. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — declarative key paths desugar to `Ord` comparator construction. |
| Q8 | Is this an optimisation, not semantics? | No — declarative API is a convenience, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates comparator chain boilerplate for the common case. |

**Classification per D-03:** StdLib. Sugar over SORTING + EQUALITY. No new language semantics.

**EDR:** [EDR-067](../decision_records/architecture/EDR-067-declarative-multi-key-sort.md)

---

#### IMMUTABLE_DATE_TIME

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Mutable date/time bugs — thread-unsafe formatters, surprising mutation semantics, inconsistent API design. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Date/time types are library types. Immutability is enforced at the type level (no setter methods). |
| Q3 | Can it be solved with existing primitives? | Yes — each date/time type = `pack` (fields) + `function` (arithmetic, formatting). Immutability = no mutating methods. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Data-First model (dates as immutable values), Safety (thread-safe by construction). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Date/time arithmetic is function composition. Parsing returns `Result<T>` (EDR-020). |
| Q6 | Can it be expressed through composition? | Yes — of existing primitives (`pack`, `function`, `call`, `===` for comparison). |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all date/time methods are function calls. |
| Q8 | Is this an optimisation, not semantics? | No — temporal arithmetic is semantic. Calendar system choice is implementation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Universal need. Immutable-by-default eliminates the most common date/time bugs. |

**Classification per D-03:** StdLib. Value-semantics date/time types. Immutable by construction.

**EDR:** [EDR-068](../decision_records/architecture/EDR-068-immutable-date-time.md)

---

#### PERSISTENT_DATA_STRUCTURES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Hash-key safe, concurrent-safe, fully immutable collections with structural sharing across versions. |
| Q2 | Is this a language problem or a library problem? | **StdLib (deferred to v0.2).** `Immutable` marker trait for compiler optimisation hooks. Collection types are library implementations. |
| Q3 | Can it be solved with existing primitives? | Partially — `Immutable` marker trait is a compile-time guarantee. Structural sharing algorithms are library implementations. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (StdLib classification), Safety (immutability guarantee). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **Interface contract only in v0.1.** The `Immutable` trait is a guarantee. Full implementations deferred to v0.2. |
| Q6 | Can it be expressed through composition? | Yes — `Immutable` trait = trait with no methods. Persistent collections = function implementations with structural sharing. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all collection operations are method calls. |
| Q8 | Is this an optimisation, not semantics? | Structural sharing is an implementation technique for value semantics. |
| Q9 | Does it affect backward compatibility? | N/A — v0.1 uses Tuple + CoW. `Immutable` trait is forward-compatible. |
| Q10 | Is it worth adding at all? | **Yes, but deferred.** v0.1 uses Tuple + CoW. `Immutable` trait accepted now as forward contract. |

**Classification per D-03:** StdLib (deferred to v0.2). `Immutable` marker trait + persistent collection types. v0.1 uses Tuple + CoW.

**EDR:** [EDR-069](../decision_records/architecture/EDR-069-persistent-data-structures.md)

---

#### DERIVE_SERIALIZATION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Manual recursive serialization code — error-prone, type-unsafe, maintenance burden. |
| Q2 | Is this a language problem or a library problem? | **StdLib / Macro.** `@derive(Serialize, Deserialize)` uses the existing macro system (EDR-029). Format-agnostic traits. |
| Q3 | Can it be solved with existing primitives? | Yes — `Serialize`/`Deserialize` traits + `@derive` macro generation. Generated code uses `pack`/`unpack` + `call`. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (macro layer, not language core), Explicitness (`@derive` annotations are visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Serialization is trait implementation generation. Deserialization returns `Result<T>` (existing mechanism). |
| Q6 | Can it be expressed through composition? | Yes — of existing primitives + `@derive` macro (EDR-029). |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — `@derive(Serialize)` desugars to macro-generated `impl` blocks. |
| Q8 | Is this an optimisation, not semantics? | No — serialization format is semantic (what bytes represent a value). |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates the most common boilerplate + bug surface in any practical codebase. |

**Classification per D-03:** StdLib / Macro. `@derive(Serialize, Deserialize)` via existing macro system (EDR-029). Format-agnostic.

**EDR:** [EDR-070](../decision_records/architecture/EDR-070-derive-serialization.md)

---

#### COMMAND_PATTERN_VIA_DELEGATE

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | GoF Command pattern requires class-per-command in languages without first-class functions — class explosion and indirection. |
| Q2 | Is this a language problem or a library problem? | **Not a new feature.** Orthon's existing delegate model (EDR-033, EDR-057) and first-class functions already subsume all Command pattern use cases. |
| Q3 | Can it be solved with existing primitives? | Yes — `() -> void` = Command, `() -> V` = Callable, `Event -> void` = ActionListener. All expressed via existing `function` + `call` + `scope` (closure capture). |
| Q4 | Does it violate any Design Principle? | No. Not adding a new concept — the existing delegate model already covers it. |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** This is a documentation-only decision — the pattern disappears into existing delegates. |
| Q6 | Can it be expressed through composition? | Yes — of existing `delegate`, `function`, `scope` concepts. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all Command patterns already expressed via existing primitives. |
| Q8 | Is this an optimisation, not semantics? | N/A — no new feature. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Not as a new feature.** The existing delegate model already covers it. Cross-reference PATTERN_MATCHING_DISPATCH (EDR-026). |

**Classification per D-03:** Existing concept. Not a new feature — delegate model subsumes Command pattern. Documentation confirmation of existing coverage.

**EDR:** [EDR-071](../decision_records/architecture/EDR-071-command-pattern-via-delegate.md)

---

#### CONTEXT_LIMITED_MODULES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | LLM attention window limit — understanding a module requires loading its entire transitive closure. Bounded context for human readers. |
| Q2 | Is this a language problem or a library problem? | **Language.** Module system requires compiler support for scoping, visibility, dependency declaration, and effect isolation. |
| Q3 | Can it be solved with existing primitives? | No. Module scoping, visibility control, and effect verification require compiler-level semantics not expressible via the 9-primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (declared dependencies and effects), Minimal Core (one module system). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Module-level scoping rules, effect isolation at module boundary, explicit dependency declaration, context budget diagnostic. |
| Q6 | Can it be expressed through composition? | No — module system is a fundamental language organisation construct. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q6. |
| Q8 | Is this an optimisation, not semantics? | No. Module organisation determines what code is visible and what effects are permitted — semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for LLM-native design — bounded context window. Also valuable for human reasoning. |

**Classification per D-03:** Language. Module system with explicit public API, declared dependencies, and effect isolation. Compiler-level capability checks.

**EDR:** [EDR-072](../decision_records/architecture/EDR-072-context-limited-modules.md)

---

#### DECLARATIVE_CONSTRUCTS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Declarative constructs reduce LLM generation error rates — specifying intent (what) is easier than implementation steps (how). |
| Q2 | Is this a language problem or a library problem? | **StdLib.** All declarative constructs are method implementations on collection/resource types. Five synthesis-friendliness criteria govern inclusion. |
| Q3 | Can it be solved with existing primitives? | Yes — each declarative construct has a documented desugaring to imperative primitives (collection ops = Iterator protocol, resource mgmt = scope, sorting = Ord trait, serialization = derive macro). |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (what over how), Minimal Core (StdLib classification). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** All constructs have defined desugaring paths. No declarative construct adds expressive power beyond primitive composition. |
| Q6 | Can it be expressed through composition? | Yes — of existing primitives. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — all declarative constructs are sugar over imperative primitives with documented desugaring. |
| Q8 | Is this an optimisation, not semantics? | No — declarative constructs express programmer intent. Implementation efficiency (loop fusion) is optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Declarative constructs improve LLM generation accuracy and human readability. Query expressions deferred to v0.2. |

**Classification per D-03:** StdLib. Declarative constructs for common transformations. All have documented desugaring. Query expressions deferred.

**EDR:** [EDR-073](../decision_records/architecture/EDR-073-declarative-constructs.md)

---

#### DECLARATION_BY_ASSIGNMENT

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Variable declaration ceremony — explicit `let`/`var`/`Type name` before first use adds noise. But no accidental creation via typos. |
| Q2 | Is this a language problem or a library problem? | **Language.** Definite assignment analysis, read-before-write detection, and `let` for shadowing all require compiler support. |
| Q3 | Can it be solved with existing primitives? | No. Definite assignment analysis — verifying variable is assigned on all paths before read — is a compiler-level flow analysis. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicitness (`let` for shadowing is visible, `mut` for mutation is visible), Concision (first assignment creates). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Definite assignment analysis (read-before-write is compile-time error), `let` shadowing semantics, no implicit globals rule. |
| Q6 | Can it be expressed through composition? | No — flow analysis across code paths is compiler-level, not expressible via primitive composition. |
| Q7 | Can it be syntactic sugar over existing primitives? | Partial — `let` for shadowing is syntactic, but definite assignment analysis requires compiler support. |
| Q8 | Is this an optimisation, not semantics? | No. Variable existence and initialization tracking is semantic — determines what operations are legal. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Concrete syntax deferred to Phase 5. |
| Q10 | Is it worth adding at all? | **Yes.** Concise (first assignment creates) + safe (read-before-write error). Deferred to Phase 5 for concrete syntax finalization. |

**Classification per D-03:** Language. Definite assignment analysis, read-before-write detection, explicit `let` for shadowing. Borderline with Phase 5 Syntax.

**EDR:** [EDR-074](../decision_records/architecture/EDR-074-declaration-by-assignment.md)

---

### Important Tier — Wave 4 (continued)

The following 7 important-tier concepts complete the Wave 4 pipeline. See individual EDRs for Gate Validation and detailed reasoning.

#### CONTRACTS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Verifiable guarantees about function behaviour — typed signatures give structure but not intent. Contracts give both, reducing LLM hallucination of nonsensical calls. |
| Q2 | Is this a language problem or a library problem? | **Language.** `requires`, `ensures`, and `invariant` keywords require syntactic integration into function signatures and compiler-level enforcement. |
| Q3 | Can it be solved with existing primitives? | No. Contract semantics (caller-visible constraints on function domain, `result` postcondition variable) are not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees, Explicitness (contracts are visible in signatures), Minimal Core (one mechanism replaces documentation assertions + runtime checks). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Pre/postcondition constraints, `result`/`old` implicit variables, invariant bounds, contract purity enforcement, Liskov inheritance rules. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No. Contract verification is a semantic guarantee. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Essential for LLM code generation accuracy and compiler-verified intent. Proven model from Eiffel and Ada 2012. |

**Classification per D-03:** Language. Contract keywords require syntactic integration into function signatures and compiler-level enforcement. Not expressible via composition of existing primitives.

**EDR:** [EDR-056](../decision_records/architecture/EDR-056-contracts.md)

---

#### DELEGATION

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Boilerplate forwarding when a type reuses behaviour from another type without inheritance (class delegation). Getters/setters with reusable behaviours (property delegation). |
| Q2 | Is this a language problem or a library problem? | **StdLib.** The `@delegate` macro desugars to explicit `impl` blocks using the existing macro system (EDR-029). No new compiler semantics. |
| Q3 | Can it be solved with existing primitives? | Yes — delegation is expressible via composition of existing primitives (trait implementation + manual method forwarding). The macro automates the pattern. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Composition Over Inheritance, Minimal Core (reuses macro system), Explicitness (`@delegate` annotation is visible). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** `@delegate` macro desugars to explicit `impl` blocks. Property delegation uses StdLib protocols. |
| Q6 | Can it be expressed through composition? | Yes — of trait `impl` blocks + method forwarding. |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — `@delegate` is sugar over macro-generated `impl` blocks. |
| Q8 | Is this an optimisation, not semantics? | No. Delegation is a code organisation pattern, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates boilerplate forwarding. StdLib classification means zero language additions. |

**Classification per D-03:** StdLib. Delegation is expressible via composition of existing primitives. Macro system automates the pattern.

**EDR:** [EDR-057](../decision_records/architecture/EDR-057-delegation.md)

---

#### EXTENSION_FUNCTIONS

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Adding behaviour to existing types without modifying source, extending via inheritance, or wrapping in an adapter. Utility function syntax (`sort(list)`) is less natural than method syntax (`list.sort()`). |
| Q2 | Is this a language problem or a library problem? | **Language.** Extension functions require the compiler to recognize receiver-call syntax on types from other modules and resolve dispatch based on static type. |
| Q3 | Can it be solved with existing primitives? | No. Receiver-call syntax on external types requires name resolution rules and static dispatch — not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Explicit Semantics (extension import required, member precedence documented), Intent Over Implementation (natural call syntax). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Extension function resolution — the compiler must find the correct definition based on receiver type, import scope, and precedence rules. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — requires compiler-level name resolution and dispatch. |
| Q8 | Is this an optimisation, not semantics? | No. Extension function dispatch determines which function runs — this is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Proven in Kotlin, C#, Swift — natural extension mechanism without inheritance or wrapper boilerplate. |

**Classification per D-03:** Language. Extension functions require compiler-level name resolution and static dispatch. Not expressible via primitive composition.

**EDR:** [EDR-058](../decision_records/architecture/EDR-058-extension-functions.md)

---

#### GRADUAL_TYPING

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | The same programmer needs dynamic speed while prototyping and static safety while shipping. The language should not force a permanent choice between the two modes. |
| Q2 | Is this a language problem or a library problem? | **Language.** Selective type checking, boundary checks at typed/untyped interfaces, and global consistency passes require compiler-level infrastructure. |
| Q3 | Can it be solved with existing primitives? | No. The concept of a type checker that can be selectively enabled/disabled at module boundaries is not expressible via primitive composition. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (one concept — optional annotations — covers the spectrum), Intent Over Implementation (start dynamic, add types as code matures). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Boundary checking, type inference on unannotated code, global consistency pass as optional lint. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No. Type checking determines program correctness — this is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Critical for LLM adoption — LLM-generated code starts with minimal annotations; compiler catches structural errors. Proven in TypeScript ecosystem. |

**Classification per D-03:** Language. Optional type annotations require compiler-level checking that can be selectively enabled/disabled. Boundary checks, inference, and consistency passes are compiler services.

**EDR:** [EDR-059](../decision_records/architecture/EDR-059-gradual-typing.md)

---

#### SMART_CAST

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | After checking that a value is a specific type, the programmer should not need to cast it again. The compiler should track type-narrowing information through control flow. |
| Q2 | Is this a language problem or a library problem? | **Language.** Flow-sensitive type analysis — tracking type information across control flow edges — is a compiler-level operation. |
| Q3 | Can it be solved with existing primitives? | No. Flow-sensitive type tracking across control flow edges is not expressible via the primitive set. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Declarative With Static Guarantees (narrowing is compiler-enforced), Explicitness (narrowing follows visible type checks). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **New semantics.** Flow-sensitive type narrowing — tracking per-variable type information across control flow edges. |
| Q6 | Can it be expressed through composition? | No — see Q3. |
| Q7 | Can it be syntactic sugar over existing primitives? | No — see Q3. |
| Q8 | Is this an optimisation, not semantics? | No. Type narrowing determines what operations are legal on a value — this is semantic. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. Builds on NULL_SAFETY (EDR-018) + PATTERN_MATCHING (EDR-025). |
| Q10 | Is it worth adding at all? | **Yes.** Eliminates redundant casts after type checks. Partially subsumed by PATTERN_MATCHING, but handles non-pattern scenarios. Proven in Kotlin and TypeScript. |

**Classification per D-03:** Language. Flow-sensitive type narrowing requires the compiler to track type information across control flow edges. Partially subsumed by PATTERN_MATCHING.

**EDR:** [EDR-060](../decision_records/architecture/EDR-060-smart-cast.md)

---

#### COPY_ON_WRITE

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Memory safety without a borrow checker learning curve. Value semantics by default with efficient implementation. |
| Q2 | Is this a language problem or a library problem? | **StdLib / Implementation Strategy.** CoW is an optimisation technique for value-semantics collections. The language spec defines value semantics; CoW is how the runtime achieves it efficiently. |
| Q3 | Can it be solved with existing primitives? | Yes — CoW is an implementation choice for existing value-semantics primitives. The programmer writes value-semantics code; CoW is invisible. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Intent Over Implementation (invisible optimisation), Minimal Core (CoW is an implementation detail). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** CoW is an optimisation technique for existing value semantics. The language surface does not expose CoW. |
| Q6 | Can it be expressed through composition? | Yes — of existing primitives (assignment, mutation, reference counting). |
| Q7 | Can it be syntactic sugar over existing primitives? | Yes — the programmer writes value-semantics code; CoW is an invisible optimisation. |
| Q8 | Is this an optimisation, not semantics? | **Yes.** CoW is purely an implementation optimisation — it changes performance, not observable behaviour. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** CoW provides the performance of sharing with the safety of copying. Avoids borrow checker complexity for common patterns. |

**Classification per D-03:** StdLib / Implementation Strategy. CoW is an implementation technique for value-semantics collections. Cross-reference VALUE_SEMANTICS (SEMANTIC_MODEL.md).

**EDR:** [EDR-061](../decision_records/architecture/EDR-061-copy-on-write.md)

---

#### PROPERTIES

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Exposing data without coupling consumers to internal representation. Stored fields and computed values should use the same call syntax. |
| Q2 | Is this a language problem or a library problem? | **StdLib.** Properties desugar to field access + function calls. The concept of "a named value with getter/setter" is expressible via existing primitives. |
| Q3 | Can it be solved with existing primitives? | Yes — attribute access + function calls already provide the primitive operations. Properties add syntactic sugar. |
| Q4 | Does it violate any Design Principle? | No. Aligns with Minimal Core (no new semantics), Uniform Access (stored and computed use same syntax). |
| Q5 | Does it add new semantics (vs. syntactic sugar)? | **No new semantics.** Implicit getter generation is syntactic sugar over attribute access + function. |
| Q6 | Can it be expressed through composition? | Yes — of attribute access + function calls. |
| Q7 | Can it be syntactic sugar over existing primitives? | **Yes** — properties are sugar over field access + getter/setter function calls. |
| Q8 | Is this an optimisation, not semantics? | No. Properties are a code organisation pattern, not an optimisation. |
| Q9 | Does it affect backward compatibility? | N/A — pre-v1.0. |
| Q10 | Is it worth adding at all? | **Yes.** Proven in C#, Kotlin, Swift — eliminates refactoring friction between stored and computed values. |

**Classification per D-03:** StdLib. Properties desugar to field access + function calls. No compiler-level semantics beyond what attribute access already provides.

**EDR:** [EDR-062](../decision_records/architecture/EDR-062-properties.md)

---

### Deferrable Tier — Deferred to v0.2/v0.3

The following 50 concepts reside in `how/concepts/research/deferrable/` and have been
deferred to v0.2 or v0.3 via the Phase 4 Decision Pipeline triage. Four additional
concepts (PROTOTYPE, SIGNIFICANT_WHITESPACE, DYNAMIC_TYPING,
CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION) were moved from deferrable/ to reject/ —
see EDR-075 through EDR-078 and [`how/concepts/research/reject/README.md`](../concepts/research/reject/README.md).

**Deferral rationale categories:**

| Category | Meaning |
|----------|---------|
| **Scope boundary** | Explicitly out of scope for v0.1 specification |
| **Dependency gated** | Depends on another concept deferred or not yet settled |
| **Complexity deferred** | Design is understood but too complex for v0.1 |
| **Low priority** | Useful but not critical for initial language adoption |
| **Implementation detail** | Better addressed in the implementation phase (Milestone 10) |
| **LLM toolchain** | Belongs to LLM toolchain workstream (post-Freeze) |

| Concept | Deferral Rationale | Target | Dependency |
|---------|-------------------|--------|------------|
| ACTORS.md | **Scope boundary** — the actor model (lightweight processes with mailbox-based communication) is an advanced concurrency concern. Orthon's delegate-based concurrency model (EDR-033) and `async`/`await` (EDR-047) provide the v0.1 concurrency surface. Actors belong in the StdLib or a v0.3 concurrency workstream. | v0.3 | CONCURRENCY_MODEL, DELEGATE |
| ACT_AS_ACTIVE_OBJECT.md | **Superseded** — active objects with mailbox (the `act` modifier) are superseded by Orthon's DELEGATE model (EDR-033) which provides a unified mechanism for all callable entities including concurrent ones. | v0.3 | DELEGATE |
| ASYNC_LAMBDA.md | **Low priority** — async lambda syntax is syntactic sugar over named async functions. Orthon's `async` modifier on `fun`/`proc`/`new` (EDR-047) already covers the semantic model; a concise lambda form can be added when lambda syntax is finalised in Phase 5. | v0.3 | ASYNC_AWAIT, SYNTAX |
| BUILDER_PATTERN_ELIMINATION.md | **Dependency gated** — the builder pattern is rendered unnecessary by Orthon's named-parameter-with-defaults model. However, named parameters are themselves deferred to StdLib status. The builder elimination follows naturally once named parameters are specified. | v0.3 | NAMED_AND_OPTIONAL_PARAMETERS |
| CODE_BLOCK_AS_HOF.md | **Low priority** — syntactic sugar for passing a block of code as a higher-order function argument (Ruby-style `do...end` blocks, Kotlin trailing lambda). Orthon's `function` + `call` primitives already cover function passing; a dedicated block syntax is ergonomic sugar for Phase 5. | v0.3 | SYNTAX |
| COLLECTIONS_FUNCTIONAL_API.md | **Low priority** — a rich functional API for collections (partition, groupBy, chunked, windowed, etc.) is StdLib work, not language specification. Orthon's COMPOSABLE_COLLECTION_OPS (EDR-032) provides the `map`/`filter`/`reduce` foundation; the extended API is implementation work for Milestone 10. | v0.3 | COMPOSABLE_COLLECTION_OPS |
| COMPILE_TIME_CONCURRENCY_SAFETY.md | **Complexity deferred** — compile-time verification of concurrency safety (race condition detection, deadlock prevention) is an advanced compiler analysis topic. Orthon's delegate-based concurrency model (EDR-033) with ownership-based safety already provides strong guarantees; extending to full compile-time concurrency checking is a v0.3+ research area. | v0.3 | CONCURRENCY_MODEL, OWNERSHIP |
| COMPILE_TIME_METAPROGRAMMING.md | **Dependency gated** — advanced compile-time metaprogramming (beyond `@macro` AST macros and comptime execution) depends on the stabilisation of Orthon's comptime model (EDR-031) and macro system (EDR-029). Post-v0.1 metaprogramming expansion is natural once the foundation is proven. | v0.3 | COMPILE_TIME_EXECUTION, AST_MACROS |
| CUSTOM_OPERATORS.md | **Complexity deferred** — custom operators introduce precedence and associativity decisions that complicate the parser and conflict with Orthon's Semantic Purity principle (one symbol, one meaning). Designing a safe operator-overloading mechanism that does not degrade readability is a hard problem requiring community experience before committing. | v0.3 | SYNTAX, PARSER |
| DEFAULT_INTERFACE_METHODS.md | **Dependency gated** — default interface methods are a subset of protocol extensions. Deferred alongside PROTOCOL_EXTENSIONS for the same reasons. Orthon's trait default implementations already cover the primary use case. | v0.3 | TRAITS |
| DEVELOPER_TOOLING.md | **LLM toolchain** — developer tooling (language server, formatter, linter, debugger) is an implementation concern for Milestone 10, not part of the language specification. The specification defines what is correct; tooling implements those definitions. | Post-Freeze | Full specification |
| DIALECTS.md | **Complexity deferred** — language subsets/dialects (e.g., Safe Orthon, Embedded Orthon) require a mature understanding of the full language before subsetting decisions can be principled. Premature subsetting would create arbitrary boundaries. | v0.3 | Full specification |
| DIFFERENTIABLE_PROGRAMMING.md | **Scope boundary** — differentiable programming (automatic differentiation for ML/AI workloads) is a domain-specific language concern. It belongs in an Orthon-for-ML workstream, not the core language specification. | v0.3 | — |
| DISPOSABLE_PATTERN.md | **Low priority** — resource management (dispose/close pattern) is expressible via Orthon's context parameters (EDR-037) and RAII-style lifetime management. A dedicated `using`/`disposable` keyword is syntactic sugar over these existing mechanisms. | v0.3 | CONTEXT_PARAMETERS |
| DYNAMIC_COLLECTIONS.md | **Low priority** — dynamic collections (growable arrays, hash maps, sets) are StdLib types, not language constructs. Orthon's data model provides the semantic foundation; concrete collection types are implementation work for Milestone 10. | v0.3 | DATA_MODEL |
| EVENTS.md | **Low priority** — event systems (publish/subscribe, event emitters) are a library/StdLib pattern expressible via Orthon's delegate mechanism and StdLib collections. A dedicated `event` keyword adds language complexity that is unnecessary for v0.1. | v0.3 | DELEGATE, TRAITS |
| EXPLICIT_COMPOSITION.md | **Low priority** — mechanisms for explicit composition (delegation declarations, composition clauses) overlap with Orthon's DELEGATION (EDR-057) and TRAITS (EDR-019). Research preserved for reference but the concept is covered by existing accepted mechanisms. | v0.3 | DELEGATION, TRAITS |
| FUNCTION_OVERLOADING.md | **Complexity deferred** — function overloading (multiple functions with the same name but different parameter types) interacts with Orthon's type inference (EDR-027), generics (EDR-024), and pattern matching dispatch (EDR-026). Designing a coherent overload resolution rule set that does not conflict with these mechanisms is a significant effort best deferred until they are stable. | v0.3 | TYPE_INFERENCE, GENERICS, PATTERN_MATCHING_DISPATCH |
| HOMOICONICITY.md | **Low priority** — code-as-data (homoiconicity, where the language's primary representation is its own AST) is philosophically interesting and enables advanced metaprogramming, but Orthon already provides `@macro` for structured code generation. Full homoiconicity adds parser/runtime complexity without proportional benefit for v0.1. | v0.3 | AST_MACROS |
| IDEMPOTENT_GENERATION.md | **LLM toolchain** — the property that LLM-generated Orthon code produces the same AST when re-generated (idempotent generation) is an LLM toolchain concern, not a language specification requirement. The spec defines the language; the toolchain ensures generation stability. | Post-Freeze | SYNTAX, LLM_NATIVE_TOOLCHAIN |
| INDEXERS.md | **Low priority** — indexer syntax (`obj[key]` → value) is syntactic sugar over function calls (`obj.get(key)`). Orthon's Semantic Purity reserves `[]` for indexing collections; custom indexers would require expanding the `[]` meaning or adding a new syntactic form. Deferrable. | v0.3 | PROPERTIES, SYNTAX |
| INTERACTIVE_DEVELOPMENT.md | **Scope boundary** — interactive development (REPL, notebook-style execution) is an implementation feature for Milestone 10. The language specification defines what correct programs look like; the runtime implements interactive execution. | v0.3 | EXECUTION_PROGRAM |
| LAZY_INITIALIZATION.md | **Low priority** — a `lazy` keyword for deferred initialisation is syntactic sugar over a closure + memoisation pattern. Orthon's eager-by-default model (with explicit lazy sequences via generators) covers the semantic need; a field-level `lazy` keyword is a convenience for v0.3. | v0.3 | GENERATORS |
| LITERATE_PROGRAMMING.md | **Low priority** — documentation-oriented programming (literate programming with embedded code blocks) is a niche workflow. While it aligns with Orthon's documentation goals, it does not belong in the v0.1 specification. Belongs to IDE/toolchain workstream. | v0.3 | — |
| LLM_NATIVE_TOOLCHAIN.md | **LLM toolchain** — LLM-native tooling (structured generation contracts, AST-aware code generation, confidence scoring) is a separate workstream that runs in parallel to the specification. The v0.1 spec defines the language: the LLM toolchain implements the generation layer. | Post-Freeze | SYNTAX, SEMANTIC_MODEL |
| METADATA_ANNOTATIONS.md | **Low priority** — metadata annotations (custom `@annotations` beyond compiler-recognised ones) are expressible via Orthon's `@` syntax for compiler-recognised metadata (EDR-029). User-defined annotations are a natural extension post-v0.1 when usage patterns emerge. | v0.3 | AST_MACROS |
| METAOBJECTS.md | **Scope boundary** — metaprogramming beyond `@macro` (AST macros) is a deep topic (method interception, `__getattr__`-style hooks) that belongs in a post-v0.1 metaprogramming workstream. The `@macro` mechanism covers the v0.1 metaprogramming surface. | v0.3 | AST_MACROS |
| MIXIN.md | **Dependency gated** — mixins are traits with concrete behaviour rather than a separate construct. The MIXIN research document itself concludes that mixins-as-traits-with-default-impls is the correct model. A dedicated `mixin` keyword or separate construct adds no semantic value and would conflict with Minimal Core. Deferred pending trait-coherence resolution. | v0.3 | TRAITS, DELEGATION |
| NATIVE_COMPILATION.md | **Implementation detail** — native code compilation is an implementation strategy concern, not a language specification issue. The language defines semantics independent of compilation target (see EXECUTION_PROGRAM, EDR-036). Whether the reference implementation compiles to native code is a Milestone 10 decision. | v0.3 | EXECUTION_PROGRAM |
| NESTED_CLASSES.md | **Low priority** — nested classes are expressible via namespacing within a module. Adding dedicated nested-class semantics (inner class access to outer instance, etc.) is a detail that can be deferred until practical usage demands it. | v0.3 | NAMESPACES |
| OBJECTS_AND_SINGLETONS.md | **Low priority** — object literals (anonymous objects with inline method definitions) are expressible via anonymous types implementing a trait. A dedicated syntax for single-use objects can be deferred until the trait system stabilises and usage patterns emerge. | v0.3 | TRAITS |
| OBSERVER.md | **Low priority** — the observer pattern (event listeners, callbacks) is a library pattern expressible via Orthon's delegate mechanism and StdLib collections. A dedicated `observer` keyword or construct is unnecessary — the pattern is already composable from existing primitives. | v0.3 | DELEGATE |
| OPEN_CLASSES.md | **Rejected in spirit** — open classes (modifying existing types from outside their definition) directly contradict Explicitness and violate module boundaries. While not formally rejected via EDR, this concept is unlikely to be accepted. Deferred as a low-priority research topic with no concrete proposal. | v0.3 | — |
| PARTIAL_CLASSES.md | **Low priority** — partial classes (C#-style split class definitions across files) are a code organisation pattern specific to code-generation workflows. Orthon's module system and `@derive` mechanism cover the same use cases without requiring partial-class language support. | v0.3 | MODULES |
| PROPERTY_WRAPPERS.md | **Low priority** — property wrappers (Swift `@propertyWrapper`, Kotlin delegated properties) are syntactic sugar over getter/setter patterns. Orthon's PROPERTIES (EDR-062) already covers the basic pattern; wrappers add ergonomic sugar that can be deferred. | v0.3 | PROPERTIES |
| PROTOCOL_EXTENSIONS.md | **Dependency gated** — protocol extensions (adding default implementations to protocols/traits) are expressible via Orthon's trait system (EDR-019) with default method implementations. The question of whether extensions can be added separately from the original trait definition (Swift-style) depends on coherence rule decisions not yet finalized. | v0.3 | TRAITS |
| QUERY_EXPRESSIONS.md | **Low priority** — LINQ-style query expressions are syntactic sugar over collection combinator chains (`.map()`, `.filter()`, etc.). Orthon's composable collection operations (EDR-032) provide the semantic foundation; query syntax can be added in a sugar workstream post-v0.1. | v0.3 | COMPOSABLE_COLLECTION_OPS |
| REFLECTION_ALTERNATIVES.md | **Scope boundary** — runtime reflection (type introspection, dynamic member access) is explicitly out of scope for the v0.1 specification. Orthon's compile-time execution model (EDR-031) provides comptime reflection; runtime reflection is a separate feature requiring runtime type information (RTTI) infrastructure. | v0.3 | COMPILE_TIME_EXECUTION |
| ROOT_OBJECT.md | **Rejected in spirit** — a universal base class (Java `Object`, C# `object`) contradicts Orthon's trait-based model where behaviour is provided via trait satisfaction, not inheritance from a root type. Research preserved for reference but unlikely to be accepted. | v0.3 | TRAITS |
| SAFE_SANDBOX.md | **Scope boundary** — sandboxed execution (running untrusted code in a restricted environment) is a security concern for the implementation (Milestone 10), not the language specification. The specification defines what programs mean; the runtime implements sandboxing. | v0.3 | EXECUTION_PROGRAM |
| SCRIPT_EXECUTION_MODEL.md | **Scope boundary** — a scripting-mode execution model (interpreted, no explicit compilation step, dynamic dispatch) is a separate execution strategy that Orthon's Execution Program model (EDR-036) already supports as a policy choice. The v0.1 specification defines the default semantics; a scripting strategy is an implementation concern. | v0.3 | EXECUTION_PROGRAM |
| SEMANTIC_ANNOTATIONS.md | **Low priority** — semantic annotations (`@pure`, `@async`, `@deterministic`) are interesting for compiler optimisation hints but are speculative without implementation experience. They may be introduced as StdLib decorations post-v0.1 rather than language-level annotations. | v0.3 | — |
| SINGLETON_CLASS.md | **Low priority** — the singleton pattern is expressible via Orthon's existing mechanisms (a module-level `let` binding of an object). A dedicated `singleton` keyword or construct would add language complexity for a pattern that is already simple to express. | v0.3 | — |
| SINGLETON_PATTERN_ANALYSIS.md | **Low priority** — analysis document for singleton pattern trade-offs. Informational only — no language feature proposed. Singleton is already expressible as a module-level `let` binding. | v0.3 | — |
| SOFTWARE_TRANSACTIONAL_MEMORY.md | **Complexity deferred** — STM (software transactional memory) is an advanced concurrency control mechanism with ongoing research on performance and semantics. It is far beyond the v0.1 concurrency surface and may never be adopted if Orthon's delegate+async model proves sufficient. | v0.3 | CONCURRENCY_MODEL |
| STRATEGY_PATTERN_ELIMINATION.md | **Low priority** — strategy pattern elimination (making strategies a language-level concept rather than a library pattern). Orthon's DELEGATE mechanism (EDR-033) and higher-order functions already cover the strategy pattern idiomatically. A dedicated strategy construct would add no semantic value. | v0.3 | DELEGATE, FUNCTIONS |
| TASK_PARALLEL_LIBRARY.md | **Implementation detail** — parallel execution patterns (parallel-for, data parallelism) are StdLib concerns built on Orthon's delegate-based concurrency primitives (EDR-033). The specification defines the concurrency model; parallel patterns are implementation work for Milestone 10. | v0.3 | CONCURRENCY_MODEL, DELEGATE |
| TOP_LEVEL_DECLARATIONS.md | **Low priority** — top-level declarations (functions, types, and values at module scope without a wrapping class) are already the default in Orthon's module model. This research document explores edge cases (ordering, visibility, shadowing) that are syntactic details for Phase 5. | v0.3 | NAMESPACES, SYNTAX |
| TYPED_HOLES.md | **Low priority** — typed holes (`_` as a typed placeholder that triggers a compile error with the expected type) are useful during development but are an IDE/compiler diagnostic feature, not a core language semantic. They add no semantic value to the specification. | v0.3 | TYPE_INFERENCE |
| USING_DIRECTIVES.md | **Low priority** — `using`/`import` directives (bringing names from other modules into scope) are a syntactic concern for Phase 5. Orthon's module/namespace model is accepted; the specific import syntax is design work for the Syntax phase. | v0.3 | NAMESPACES, SYNTAX |

**Notes:**
- Four concepts originally in `deferrable/` have been formally rejected and moved to `how/concepts/research/reject/`. See EDR-075 (PROTOTYPE), EDR-076 (SIGNIFICANT_WHITESPACE), EDR-077 (DYNAMIC_TYPING), EDR-078 (CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION).
- `ACT_AS_ACTIVE_OBJECT.md` is listed as superseded — its semantic content is subsumed by DELEGATE (EDR-033).

**Total deferrable concepts documented: 50** (across 54 original deferrable files, minus 4 moved to reject/).
