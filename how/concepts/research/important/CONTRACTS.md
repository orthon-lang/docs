# Design by Contract

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language provide verifiable guarantees about a function's behaviour — both to human readers and to LLMs generating code?

Traditional approaches:

- **Documentation-only** (Javadoc, docstrings) — human-readable but compiler-ignored. Contracts drift from implementation with no mechanism to detect divergence.
- **Defensive assertions** (`assert` in C, Python) — runtime-checked but not part of the type system. No caller-side guarantees.
- **Type-level encoding** (Rust's trait bounds, Haskell's type classes) — powerful but limited. Cannot express relational properties between inputs and outputs.
- **Eiffel-style contracts** — full Design by Contract with `require`, `ensure`, `invariant`. Proved the concept but had limited adoption, partly due to runtime-only enforcement and performance assumptions.

The core problem: **a function signature describes what types flow in and out, but not what relationship they satisfy**. For LLMs generating code, a typed signature gives structure but not intent. A contract gives both — and the compiler can use it for verification, test generation, and error diagnosis.

Three specific gaps motivate contracts as a first-class Orthon feature:

1. **LLM generation accuracy** — An LLM given `fn sqrt(x: Float) -> Float` may produce `sqrt(-1.0)`. A contract `requires x >= 0.0` and `ensures result * result ≈ x` tells the LLM the *intent*, not just the *shape*. This reduces hallucination of nonsensical calls.

2. **Compiler-verified intent** — Contracts checked at compile time (where possible) or runtime detect contract violations at the earliest possible moment, converting silent logic errors into diagnosed failures.

3. **Test synthesis** — Contracts are executable specifications. The compiler or toolchain can generate test cases from contracts, targeting edge cases at contract boundaries.

## Principles

1. **Contracts are part of the function signature** — `requires`, `ensures`, and `invariant` appear in the function declaration alongside parameters and return type. They are not comments or optional metadata.

2. **Framework independence** — Contracts are a language feature, not a library. No annotation framework, decorator, or external DSL is needed.

3. **Static where possible, dynamic where necessary** — Contracts that the compiler can verify statically produce compile-time errors. Contracts that require runtime values degrade to runtime assertions. The programmer should not need to distinguish between the two modes.

4. **No performance penalty in release (when satisfied)** — Contracts are checked during development and testing. In release builds, satisfied contracts can be elided; the programmer opts into runtime checks explicitly.

5. **Contracts compose** — A caller's `ensures` must satisfy the callee's `requires`. Object or module `invariant`s are checked on entry and exit of every public method. Contract inheritance follows Liskov substitution: a subtype's contracts may weaken preconditions and strengthen postconditions (covariant results, contravariant arguments).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Contract Enforcement Policy | Determines when contracts are checked (compile-time, runtime, release-build elision) |
| Contract Inheritance Policy | Governs how subtype contracts relate to supertype contracts (covariant/contravariant rules) |
| Contract Language Policy | Specifies the expression language for contracts (pure expressions only, or any expression) |
| Error Policy | Controls how contract violations are reported and whether recovery is possible |

## Model (What)

### Function Contracts

A function declares its contract as part of its signature:

```orthon
fn sqrt(x: Float) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
```

- **`requires`** — precondition. Satisfied by the caller before every call.
- **`ensures`** — postcondition. Guaranteed by the function after every successful return.
- **`result`** — implicit variable binding the function's return value in the `ensures` clause.
- **`old`** — implicit variable capturing a parameter's value at function entry (useful in `ensures` for mutable data).

### Object/Module Invariants

Types and modules can declare invariants that must hold at every public boundary:

```orthon
class Queue<T>
    invariant size >= 0
    invariant size <= capacity

    fn enqueue(item: T)
        requires size < capacity
        ensures size == old.size + 1
```

### Higher-Order Contracts

Functions accepting functions can constrain their arguments:

```orthon
fn apply_twice(f: (Int) -> Int, x: Int) -> Int
    requires f.requires(x)              # caller ensures x satisfies f's precondition
    ensures result == f(f(x))
    requires f.ensures(f(x)) ≈ ...      # composition constraint
```

*Note: Full higher-order contract support depends on the Type System design and may be deferred.*

### Contract Expressions

Contract expressions are **pure** — they must not produce side effects. The compiler enforces this:

- No mutation of captured variables.
- No I/O operations.
- No non-deterministic functions.
- Contracts may only call other pure functions.

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 0–1 (Data Model / Primitive Operation)

Library test:   Can Design by Contract be implemented as a Standard Library
                function using existing Core constructs?

                Requires syntactic integration into function signatures
                (`requires`, `ensures` keywords in the declaration). A library
                could provide `check_precondition(fn, cond)` as a wrapper, but
                it cannot alter the language's syntax or make contracts a
                first-class part of the type signature visible to the caller.

                → FAIL (cannot be a library).

Pattern test:   Can contracts be expressed as a composition of existing
                Primitive Operations and Data Model constructs?

                A pattern could approximate contracts using wrapper functions
                and runtime assertions, but:
                - No existing primitive provides caller-visible constraints
                  on a function's domain.
                - No existing primitive provides an `ensures` construct that
                  references the return value as `result`.
                - Composition of `function` + `call` + `assert` cannot express
                  compile-time contract checking or inheritance rules.

                → FAIL (cannot be composed from existing primitives).

Primitive test: Is Design by Contract an atomic operation on data that
                cannot be decomposed?

                Contracts introduce a new semantic mechanism: declarative
                constraints on function behaviour that are part of the type
                signature and enforceable by the compiler. This is atomic —
                it cannot be decomposed into simpler operations because
                the constraint semantics themselves are the primitive.

                → PASS — belongs in Primitive Operations (Level 1).

Filter result: Level 1 (Primitive Operation).
```

## Default Strategy

Contracts are checked at compile time when the compiler can prove satisfaction (simple preconditions on literal arguments, type-level constraints). Otherwise, contracts are checked at runtime during debug and test builds. In release builds, contracts are elided unless the programmer explicitly requests `--enable-contracts`.

The default expression language is a pure subset of Orthon itself — no separate assertion DSL.

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Static-only** | All contracts verified at compile time; no runtime checks. Requires expressive type-level contract language. Suitable for safety-critical systems (see `EMBEDDED_STRATEGY.md`). |
| **Runtime-only** | Contracts are unchecked at compile time; always checked at runtime. Simpler compiler, no proof engine. Suitable for rapid prototyping (see `LLM_STRATEGY.md`). |
| **Ghost annotations** | Contracts are written as special comments or annotations that the compiler ignores but tooling (LLM Schema Provider, documentation generator) uses. No enforcement; pure documentation value. Suitable for gradual adoption. |
| **Eiffel-style** | Full runtime contract checking with inheritance, `old`, `result`, and class invariants. Used in Eiffel and Ada 2012. |

## Open Questions

1. Should contracts be part of the type system (i.e., `fn sqrt(x: Float) -> Float | requires x >= 0.0`) or a separate declaration block attached to the function?
2. How do contracts interact with higher-order functions and generic type parameters?
3. Should contract expressions support quantifiers (`forall`, `exists`) over collection types?
4. Can the compiler prove simple arithmetic constraints (e.g., `x >= 0.0`) at compile time, or is that deferred to an SMT solver backend?
5. Are contracts inherited across Implementation Strategies? If `DEFAULT_STRATEGY` checks contracts at runtime but `HIGH_PERFORMANCE_STRATEGY` elides them, does the contract still appear in the schema for LLM tooling?
6. Should contracts affect the calling convention (e.g., `??` operator propagation for contract violations)?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md` — Contract Enforcement Policy, Contract Inheritance Policy
- [x] `how/concepts/research/ERROR_HANDLING.md`
- [x] `how/concepts/research/GRADUAL_TYPING.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] `how/strategies/EMBEDDED_STRATEGY.md`
- [ ] `how/strategies/HIGH_PERFORMANCE_STRATEGY.md`
- [ ] Other: `how/architecture/CORE_INCLUSION_FILTER.md`
