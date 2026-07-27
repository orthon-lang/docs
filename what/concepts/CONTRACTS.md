# Contracts

## Issue (Why)

How does a language provide verifiable guarantees about a function's behaviour — both to human readers and to LLMs generating code? A function signature describes what types flow in and out, but not what relationship they satisfy. For LLMs generating code, a typed signature gives structure but not intent. A contract gives both — and the compiler can use it for verification, test generation, and error diagnosis.

Three specific gaps motivate contracts as a first-class Orthon feature:

1. **LLM generation accuracy** — An LLM given `fn sqrt(x: Float) -> Float` may produce `sqrt(-1.0)`. A contract `requires x >= 0.0` and `ensures result * result ≈ x` tells the LLM the *intent*, not just the *shape*.
2. **Compiler-verified intent** — Contracts checked at compile time (where possible) or runtime detect contract violations at the earliest possible moment.
3. **Test synthesis** — Contracts are executable specifications. The compiler or toolchain can generate test cases from contracts.

## Principles

1. **Contracts are part of the function signature** — `requires`, `ensures`, and `invariant` appear in the function declaration alongside parameters and return type.
2. **Framework independence** — Contracts are a language feature, not a library.
3. **Static where possible, dynamic where necessary** — Contracts that the compiler can verify statically produce compile-time errors. Contracts that require runtime values degrade to runtime assertions.
4. **No performance penalty in release (when satisfied)** — Contracts are checked during development and testing. In release builds, satisfied contracts can be elided.
5. **Contracts compose** — A caller's `ensures` must satisfy the callee's `requires`. Contract inheritance follows Liskov substitution.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Contract Enforcement Policy | Determines when contracts are checked (compile-time, runtime, release-build elision) |
| Contract Inheritance Policy | Governs how subtype contracts relate to supertype contracts |
| Contract Language Policy | Specifies the expression language for contracts (pure expressions only) |
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

### Contract Expressions

Contract expressions are **pure** — they must not produce side effects. The compiler enforces this:
- No mutation of captured variables.
- No I/O operations.
- No non-deterministic functions.
- Contracts may only call other pure functions.

## Default Strategy

Contracts are checked at compile time where the compiler can prove satisfaction statically. Remaining contracts degrade to runtime assertions in debug builds. Release builds elide all contracts unless `--enforce-contracts` is passed. Higher-order contract support is deferred to v0.2.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Runtime-only (Eiffel) | All contracts checked at runtime. No static verification. Simpler compiler, but errors are deferred. |
| Static-only (Dafny) | Contracts must be fully verified at compile time. Strongest guarantees, but limits what can be expressed. |
| Documentation-only | Contracts are comments (Javadoc, docstrings). No verification. Simplest but provides no guarantees. |
| Type-level encoding | Contracts encoded in the type system via phantom types and type states. Powerful but limited to type-level properties. |

## Open Questions

1. Should higher-order contracts (contracts on function arguments) be part of v0.1 or v0.2?
2. How does `old` work with mutable reference parameters?
3. Should contracts be inheritable across trait implementations?
4. Performance model for runtime contract checking in hot paths.

## Decision History

- **EDR-056:** Contracts accepted as Language feature — compiler-enforced pre/postconditions.
- **Classification per D-03:** Language. Contract keywords (`requires`, `ensures`, `invariant`) require syntactic integration into function signatures and compiler-level enforcement. Not expressible via composition of primitives.
- **Cross-reference:** TRAITS (EDR-019) — contracts compose with trait dispatch; trait methods can declare contracts that implementations must satisfy.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
