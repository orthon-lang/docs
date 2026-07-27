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
