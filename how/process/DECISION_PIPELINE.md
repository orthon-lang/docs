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
