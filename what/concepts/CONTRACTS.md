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

A function may declare several preconditions; the caller must satisfy every `requires` before the call:

```orthon
fn withdraw(balance: Int, amount: Int) -> Int
    requires amount > 0
    requires amount <= balance
    ensures result == balance - amount
    return balance - amount
```

Postconditions are **relational** — they tie the output to the inputs, which a typed signature alone cannot express. A function that returned `balance + amount` would be just as well-typed as one returning `balance - amount`; the `ensures` clause is what states which relation is intended. Postconditions may also bound the output without naming it:

```orthon
fn clamp(x: Int, lo: Int, hi: Int) -> Int
    requires lo <= hi
    ensures result >= lo
    ensures result <= hi
```

Contract expressions are pure and may call other pure functions, so shared domain predicates can be reused across signatures (see [Contract Expressions](#contract-expressions)).

#### Where Contracts Are Evaluated (Lexical vs. Execution Position)

Syntactic position is not execution position. Contract clauses appear between the header and the body only because that is where a signature lives — visible to the caller and to the compiler — but they are not statements that run in the sequence of the body. Execution is hoisted to the function boundary: `requires` is evaluated on entry, before the body runs, and `ensures` on every normal exit, after the body has already produced its value.

This is why `result` is a **ghost binding**, not a variable of the body's scope. It exists only inside `ensures` expressions: it cannot be assigned, it does not leak into ordinary code, and it is unavailable in `requires` and `invariant` clauses. The compiler binds `result` to the value the function is about to return at the moment the postcondition is evaluated — and because that moment comes *after* the value exists, the reference is valid even though the clause is written above the body.

The compiler instruments the boundary, not the body: the callee's source is untouched, and checks are placed around it. Desugaring (illustrative — the exact rewrite depends on the final return/body syntax):

```text
fn withdraw(balance: Int, amount: Int) -> Int
    # ENTRY — requires, evaluated before the body
    assert amount > 0
    assert amount <= balance
    old := snapshot(balance)            # only if `old` is used

    # BODY — as written; each normal exit is rewritten:
    result := <returned value>
    # EXIT — ensures, evaluated after the body produced a value
    assert result == balance - amount
    return result
```

Because the check is attached to the boundary rather than written as a trailing statement, `ensures` covers every normal exit — branches, early returns, and tail expressions — and applies only to successful exits, matching the definition above ("after every successful return"). Where the compiler can prove a contract statically, no runtime check is emitted. The remaining checks behave as assertions in debug and test builds and are elided in release builds unless `--enforce-contracts` is passed.

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

Invariants attach to the aggregate — a type or a module — not to individual free functions: a free function states its guarantees with `requires`/`ensures`, while an `invariant` declares what must hold across every public operation of the aggregate. Invariants are checked on entry and exit of each public method, so methods do not restate them: `enqueue` above does not repeat `size >= 0` and `size <= capacity` — they already hold when the method runs and must hold again when it returns.

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
