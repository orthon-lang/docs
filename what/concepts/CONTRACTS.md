# Contracts

## Issue (Why)

How does a language provide verifiable guarantees about a function's behaviour — both to human readers and to LLMs generating code? A function signature describes what types flow in and out, but not what relationship they satisfy. For LLMs generating code, a typed signature gives structure but not intent. A contract gives both — and the compiler can use it for verification, test generation, and error diagnosis.

Three specific gaps motivate contracts as a first-class Orthon feature:

1. **LLM generation accuracy** — An LLM given `fn sqrt(x: Float) -> Float` may produce `sqrt(-1.0)`. A contract `requires x >= 0.0` together with `ensures result >= 0.0` and `ensures abs(result * result - x) <= 0.000001 * x` tells the LLM the *intent*, not just the *shape*. The contracted signature names the result (`-> result: Float`), so the clauses can refer to it.
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
| Contract Enforcement Policy | Determines when contracts are checked (compile-time, runtime, release-build elision); values `All`, `None`, `Preconditions`, `Boundary` (see Default Strategy) |
| Contract Inheritance Policy | Governs how subtype contracts relate to supertype contracts |
| Contract Language Policy | Specifies the expression language for contracts (pure expressions only) |
| Error Policy | Controls how contract violations are reported and whether recovery is possible |

## Model (What)

### Function Contracts

A function declares its contract as part of its signature:

```orthon
fun sqrt(x: Float) -> result: Float
    requires x >= 0.0
    ensures result >= 0.0
    ensures abs(result * result - x) <= 0.000001 * x
```

- **`requires`** — precondition. Satisfied by the caller before every call.
- **`ensures`** — postcondition. Guaranteed by the function after every successful exit.
- **Named result** — the return value is named in the signature (`-> result: Float`). The name is chosen by the function's author; `result` is a convention, not a reserved word. It is visible only in `ensures` clauses ([EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md)).
- **`old`** — **deferred out of v0.1** (author's decision, 2026-10-19). It would capture state at function entry for use in `ensures`, and matters only for state a callable mutates. Postconditions in v0.1 speak about the state after the call and about the result. Writing `old` in a postcondition is a compile error with a dedicated message stating that `old` is deferred out of v0.1; the message offers two workarounds: an absolute condition, or a `new` method that returns the changed value ([EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md)). Analysis and reintroduction triggers: [`CONTRACT_OLD_STATE.md`](../../how/concepts/research/deferrable/CONTRACT_OLD_STATE.md).

The `ensures result >= 0.0` clause pins the sign of the result: the relational clause alone would also admit the negated root `-sqrt(x)`.

A function may declare several preconditions; the caller must satisfy every `requires` before the call:

```orthon
fun withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    requires amount <= balance
    ensures result == balance - amount
    return balance - amount
```

Postconditions are **relational** — they tie the output to the inputs, which a typed signature alone cannot express. A function that returned `balance + amount` would be just as well-typed as one returning `balance - amount`; the `ensures` clause is what states which relation is intended. Postconditions may also bound the output without stating its exact value:

```orthon
fun clamp(x: Int, lo: Int, hi: Int) -> result: Int
    requires lo <= hi
    ensures result >= lo
    ensures result <= hi
```

Contract expressions are pure and may call other pure functions, so shared domain predicates can be reused across signatures (see [Contract Expressions](#contract-expressions)).

#### Where Contracts Are Evaluated (Lexical vs. Execution Position)

Syntactic position is not execution position. Contract clauses appear between the header and the body only because that is where a signature lives — visible to the caller and to the compiler — but they are not statements that run in the sequence of the body. Execution is hoisted to the function boundary: `requires` is evaluated on entry, before the body runs, and `ensures` on every normal exit, after the body has already produced its value.

This is why the named result is a **ghost binding**, not a variable of the body's scope. The mistakes the compiler rejects:

```orthon
fun half(x: Int) -> Int
    ensures result == x / 2          // ERROR: the result is not named; write `-> result: Int`
fun f(result: Int) -> result: Int    // ERROR: duplicate declaration of `result`
fun g(x: Int) -> r: Int
    ensures is_positive              // ERROR: function value; write is_positive(r)
```

The name declared in the signature is bound to the returned value when the postcondition is evaluated — and because that moment comes *after* the value exists, the reference is valid even though the clause is written above the body ([EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md)). The rules:

- **Scope.** The name exists only inside `ensures` expressions: it is not visible in `requires`, in `invariant` or in the body, and it cannot be assigned.
- **Duplicates.** A name equal to a parameter name is a same-scope duplicate-declaration error (the rule is stated in [`NAME_RESOLUTION.md`](../../how/architecture/NAME_RESOLUTION.md)). A declaration in the body with the same name as the named result is a **warning**, not an error: the two live in different scopes, but a reader can mistake one for the other.
- **Unnamed result.** An `ensures` that mentions the result of a function with an unnamed result is a compile error with a hint to name it in the signature. A function with no return value, or whose conditions do not mention the result, needs no name.
- **Successful exit.** A successful exit is a normal return with a non-error value. For a function whose declared result is an error union, it is the success case, and the name binds the success value. Termination through a system error is not an exit, and `ensures` is not evaluated.
- **Function values.** An `ensures` expression that is a function value is a compile error with a hint to call it on the result (for example `is_non_negative(result)`).

The compiler instruments the boundary, not the body: the callee's source is untouched, and checks are placed around it. Desugaring (illustrative — the exact rewrite depends on the final return/body syntax):

```text
fn withdraw(balance: Int, amount: Int) -> result: Int
    # ENTRY — requires, evaluated before the body
    assert amount > 0
    assert amount <= balance

    # BODY — as written; each normal exit is rewritten:
    result = <returned value>           # the name declared in the signature is bound to the returned value (declaration by assignment, EDR-074)
    # EXIT — ensures, evaluated after the body produced a value
    assert result == balance - amount
    return result
```

Because the check is attached to the boundary rather than written as a trailing statement, `ensures` covers every normal exit — branches, early returns, and tail expressions — and applies only to successful exits, matching the definition above ("after every successful exit"). Where the compiler can prove a contract statically, no runtime check is emitted. The remaining checks behave as assertions in debug and test builds and are elided in release builds unless `--enforce-contracts` is passed.

#### Where Contracts Apply

*Tier 4 clarification, made together with [EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md); it adds no keyword.*

```orthon
trait Shape
    fun area(self) -> a: Float
        ensures a >= 0.0

impl Shape for Square
    fun area(self) -> result: Float          // a different result name than the trait
        ensures result == self.side * self.side    // stronger than the inherited `a >= 0.0`
        return self.side * self.side         // a method: `self` exists

fun square(x: Int) -> result: Int            // a free function: no `self`
    ensures result >= 0
    return x * x
```

- Contracts may be declared on any function or method, except that generators and async functions carry only `requires` in v0.1 (see [Generators and Async Functions](#generators-and-async-functions)). `self` exists only in methods; a free function has none.
- Contracts of trait methods are inherited by their implementations (Principle 5, Liskov substitution): an implementation may weaken `requires` and strengthen `ensures`; a stronger `requires` is an error.
- Result names need not match between a trait method and its implementation. An inherited clause uses the name of the declaring signature, so the inherited `ensures a >= 0.0` above still reads `a`.

#### Generators and Async Functions

> **Decision (Tier 4, inline; author, 2026-10-23).** In v0.1 a generator function and an async function may carry only `requires`. `ensures` and `invariant` are forbidden, with a hint. The reasons: the survival criterion (the language can live without them) and the fact that their meaning is not yet defined: what "exit" means for a lazy stream and for a function that suspends. They are staged for v0.2 and v0.3.

The two kinds of function:

- A **generator function** returns a lazy `Iterator<T>` and produces values with `emit`; its body runs when the iterator is consumed.
- An **async function** is `async fun`, `async proc` or `async new`; it returns a `Future<T>` and may suspend at `await`.

Rules for v0.1:

1. **`requires` is checked at the call**, before the iterator or the future is created, in the caller's context. The caller is at fault. It is not checked at the first `next()` and not at the start of the body after a suspension.
2. **`ensures` is an error** on a generator function and on an async function. Message: `ensures` is not available on generator and async functions in v0.1. Hint: state the condition on the consumer side, or wait for v0.2.
3. **`invariant` is not combined with suspension.** A type that declares an `invariant` cannot declare an `async proc` or a generator `proc` in v0.1 (error with a hint: the invariant cannot be checked across a suspension, see v0.3). `async fun` and `async new` stay allowed, with `requires` only.
4. **A `gen(...)` expression has no signature**, so it has no contract.
5. **The named result** of an async function names the value of type `T`, not the `Future<T>`. It is not used in v0.1, because there is no `ensures`.

```orthon
fun evens(limit: Int) -> Iterator<Int>
    requires limit >= 0                  // ok: checked at the call, before the iterator is created
    for x in 1..limit
        if x % 2 == 0
            emit x

fun positives(limit: Int) -> result: Iterator<Int>
    ensures is_sorted(result)            // ERROR in v0.1: ensures on a generator; state it on the consumer side

async fun fetch(url: Url) -> String
    requires url.is_valid()              // ok: checked at the call, before the future is created
    return await http.get(url).body

async fun load(url: Url) -> body: String
    ensures body.length() > 0            // ERROR in v0.1: ensures on an async function; planned for v0.2

type Account(balance: Int)
    invariant self.balance >= 0
    async proc deposit(amount: Int)      // ERROR in v0.1: a type with an invariant cannot have an async proc
        await ledger.record(amount)
        self.balance += amount

let squares = gen(x * x for x in 1..10) // no signature, no contract
```

Stages:

| Version | Added |
|---------|-------|
| v0.1 | `requires` at the call |
| v0.2 | `ensures` on the successful completion of `async fun` and `async new` (the named result binds the `T`); conditions on the elements of a generator, as a separate form and not through the result name |
| v0.3 | `invariant` across suspension, together with the task and `exclusive` rules of [`ASYNC_AWAIT.md`](ASYNC_AWAIT.md) |

### Type Invariants

> **Decision (Tier 4, inline; author, 2026-10-23).** Invariants of modules are removed from v0.1. A module has no state in the accepted documents, so a module invariant had no subject. Only types declare invariants. Module invariants may return with the definition of a module boundary and its rules (planned for v0.3, together with the `Boundary` value of the Contract Enforcement Policy).

A type can declare invariants that must hold at every public boundary:

```orthon
type Queue<T>
    invariant self.size >= 0
    invariant self.size <= self.capacity

    proc enqueue(item: T)
        requires self.size < self.capacity
        ensures self.back() == item
```

Invariants attach to the type, not to individual free functions: a free function states its guarantees with `requires`/`ensures`, while an `invariant` declares what must hold across every public operation of the type. Invariants are checked on entry and exit of each public method, so methods do not restate them: `enqueue` above does not repeat `self.size >= 0` and `self.size <= self.capacity`: they already hold when the method runs and must hold again when it returns.

### Contract Expressions

Contract expressions are **pure** — they must not produce side effects. The compiler enforces this:
- No mutation of captured variables.
- No I/O operations.
- No non-deterministic functions.
- Contracts may only call other pure functions.
- Violation reports name the failed predicate: diagnostics print the called predicate with its arguments, for example `requires covered_by(balance, amount)`.

#### What a Condition May Call

> **Decision (Tier 4, inline; author, 2026-10-23).** Purity is checked against a **closed list of effectful operations**, not by an effect model. Named conditions stay. The effect model replaces the list in v0.2, once [`EFFECT_FOOTPRINT.md`](../../how/concepts/research/essential/EFFECT_FOOTPRINT.md) is accepted.

Why purity is a rule and not a convention: the Contract Enforcement Policy may elide a check (value `None`), so a condition with a side effect would change the behaviour of the program between builds. The meaning of a contract must not depend on the policy.

A condition may contain:

1. Operators, literals, and reads of parameters, fields and the named result (in `ensures`).
2. Calls to functions of the standard library that are not on the list below.
3. Calls to functions and methods whose body is known (a concrete declaration) and **passes the check**.
4. Calls to `fun` members of the contract that carries the condition (the contract being declared).

The **check**: a function passes if it is of the non-mutating kind (`fun`, not `proc`), mutates no captured or global state, and, transitively through the functions it calls, calls nothing on the list and no function that fails the check.

The **closed list of effectful operations (v0.1):** input and output (console, files, network), clocks and timers, random numbers, foreign function calls (FFI), reading or writing global mutable state, and any `proc`. The list is part of the standard library boundary ([`LIBRARY_BOUNDARY.md`](../LIBRARY_BOUNDARY.md)) and grows by the usual process. The check is conservative: it may reject a function that is in fact pure but that it cannot prove pure.

**Members of a contract.** If any condition of a contract `C` calls a `fun` member `m` of `C`, then every implementation of `m` must pass the check. The check is made where the implementation is declared; a failing implementation is a compile error naming the contract and the condition that needs it.

**Through a generic bound or `dyn`.** The body is not known, so a condition may call only the members of the contract that carries the condition. Calling a member of another contract through a bound is a compile error with a hint (call a function with a known body, or use a member of this contract). This is a limitation of v0.1; it is lifted by the effect model.

```orthon
fun is_positive(n: Int) -> Bool            // known body, passes the check
    return n > 0

fun log_and_check(n: Int) -> Bool          // calls print: on the list
    print("checking")
    return n > 0

fun withdraw(balance: Int, amount: Int) -> result: Int
    requires is_positive(amount)           // ok: a named condition with a known body
    requires log_and_check(amount)         // ERROR: the condition calls print (input and output)
    ensures abs(result) >= 0               // ok: a library function that is not on the list

contract Stack<T>
    fun is_empty(self) -> Bool
    proc push(item: T)
        ensures not self.is_empty()        // ok: a member of this contract; every implementation of is_empty must pass the check

fun show<Printable as T>(item: T) -> result: String
    ensures result == item.format()        // ERROR: format is a member of another contract, called through a bound
```

**Contracts as business rules.** Named pure predicates carry the rules of a domain and read like the rules themselves:

```orthon
fun is_positive(n: Int) -> Bool
    return n > 0

fun covered_by(balance: Int, amount: Int) -> Bool
    return amount <= balance

fun withdraw(balance: Int, amount: Int) -> result: Int
    requires is_positive(amount)
    requires covered_by(balance, amount)
    ensures result == balance - amount
    return balance - amount
```

Named pure predicates are reused across clauses and signatures. A module of rules should not import implementation modules; that is a project rule or a lint, not a language rule.

## Default Strategy

Contracts are checked at compile time where the compiler can prove satisfaction statically. Remaining contracts degrade to runtime assertions in debug builds. Release builds elide all contracts unless `--enforce-contracts` is passed. Higher-order contract support is deferred to v0.2.

### Contract Enforcement Policy

> **Decision (Tier 4, inline; author, 2026-10-23).** The granularity of enforcement is staged: v0.1 uses one value for the whole build (variant a), v0.2 adds a choice by kind of condition (variant b), v0.3 adds a choice by position at a module boundary (variant c). The values of the Policy are fixed now, so that a reserved name cannot be reused for something else.

| Value | What is checked at run time | Version | Status |
|-------|-----------------------------|---------|--------|
| `All` | every clause that is not proved at compile time: `requires`, `ensures`, `invariant` | v0.1 | Active. The default of a debug build; selected for a release build by `--enforce-contracts` |
| `None` | nothing; a violated contract is not detected | v0.1 | Active. The default of a release build |
| `Preconditions` | `requires` only; `ensures` and `invariant` are elided. A `requires` is the caller's obligation and is the cheaper check | v0.2 | Reserved. Granularity: by kind of condition |
| `Boundary` | the clauses of the public functions of a module (the module boundary); calls inside the module are not checked | v0.3 | Reserved. Granularity: by position; it needs the definition of a module boundary and its rules |

Rules:

1. **v0.1 granularity is one value per build.** There is no per-module, per-function or per-clause selection in v0.1.
2. **The meaning of a contract does not depend on the value.** A contract is the same specification under every value; the value changes only what is detected. Under `None` a violated contract is an undetected program error.
3. **A reserved value cannot be selected in v0.1.** Selecting `Preconditions` or `Boundary` is a build error with a hint that names the version in which the value becomes available.
4. **The choice is made by the build configuration, not in the text of the program.** The policy is not written in the source of a contract.

Examples (the build configuration is outside the program text; spellings of flags other than `--enforce-contracts` are not fixed):

```text
debug build                       -> All
release build                     -> None
release build --enforce-contracts -> All
release build, policy Boundary    -> ERROR in v0.1: the value is reserved for v0.3
```

## Alternative Strategies

| Strategy | Description |
|---|---|
| Runtime-only (Eiffel) | All contracts checked at runtime. No static verification. Simpler compiler, but errors are deferred. |
| Static-only (Dafny) | Contracts must be fully verified at compile time. Strongest guarantees, but limits what can be expressed. |
| Documentation-only | Contracts are comments (Javadoc, docstrings). No verification. Simplest but provides no guarantees. |
| Type-level encoding | Contracts encoded in the type system via phantom types and type states. Powerful but limited to type-level properties. |

## Open Questions

1. Higher-order contracts (contracts on function arguments): *deferred to v0.2.* See [`TRAIT_CONTRACT_UNIFICATION.md`](../../how/concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md).
2. *Deferred with `old` (2026-10-19):* how does `old` work with mutable state? See [`CONTRACT_OLD_STATE.md`](../../how/concepts/research/deferrable/CONTRACT_OLD_STATE.md).
3. Should contracts be inheritable across trait implementations?

   > **Decision:** Yes. Trait-method contracts are inherited; an implementation may weaken `requires` and strengthen `ensures` (see [Where Contracts Apply](#where-contracts-apply)).

4. Performance model for runtime contract checking in hot paths. *Partly addressed (2026-10-23):* `Preconditions` (v0.2) and `Boundary` (v0.3) reduce the cost of enforcement; the cost model itself is still open.
5. *Deferred:* external imposition of rules on foreign functions, and a rule registry file. A comment may carry a reference to a rule; the form is unspecified.
6. How finely may enforcement be controlled?

   > **Decision (Tier 4, inline; author, 2026-10-23):** Staged. v0.1: one value per build (`All` or `None`). v0.2: by kind of condition (`Preconditions`). v0.3: by position at a module boundary (`Boundary`), when the module boundary and its rules are defined. The values are fixed now; see [Contract Enforcement Policy](#contract-enforcement-policy).

7. What may a condition call (what counts as pure)?

   > **Decision (Tier 4, inline; author, 2026-10-23):** Variant B. Named conditions stay; purity is checked against a closed list of effectful operations; members of the contract are allowed if their implementations pass the check; through a generic bound or `dyn` only members of the same contract; the effect model replaces the list in v0.2. See [What a Condition May Call](#what-a-condition-may-call).

8. What do contracts mean for generators and async functions?

   > **Decision (Tier 4, inline; author, 2026-10-23):** Variant b. Only `requires` in v0.1, checked at the call; `ensures` and `invariant` are forbidden with a hint; `ensures` for async completion and generator elements in v0.2; invariants across suspension in v0.3. See [Generators and Async Functions](#generators-and-async-functions).

## Decision History

- **EDR-056:** Contracts accepted as Language feature — compiler-enforced pre/postconditions.
- **Classification per D-03:** Language. Contract keywords (`requires`, `ensures`, `invariant`) require syntactic integration into function signatures and compiler-level enforcement. Not expressible via composition of primitives.
- **Cross-reference:** TRAITS (EDR-019) — contracts compose with trait dispatch; trait methods can declare contracts that implementations must satisfy.
- **2026-10-19 — `old` deferred out of v0.1.** Author's decision: `old` adds complexity now and gives little in return. The deferral was applied inline at the author's instruction and is recorded by [EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md). The `Queue` example now uses `proc` (mutating methods are `proc` per SEMANTIC_MODEL § Mutation) and an absolute postcondition. Analysis kept as a deferred hypothesis: [`CONTRACT_OLD_STATE.md`](../../how/concepts/research/deferrable/CONTRACT_OLD_STATE.md).
- **2026-10-21 — EDR-097 (amends EDR-056).** The result is named in the signature (`-> result: Float`) and visible only in `ensures`; `old` is deferred out of v0.1 with a dedicated diagnostic. Signed off by the author. See [EDR-097](../../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md).
- **Under review (hypothesis, not accepted):** named contract objects, a conformance keyword and multi-instance contracts are examined in [`TRAIT_CONTRACT_UNIFICATION.md`](../../how/concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md).
- **2026-10-23 — Contract Enforcement Policy values fixed (Tier 4, inline).** Author's decision: granularity of enforcement is staged (v0.1 one value per build, v0.2 by kind of condition, v0.3 by position at a module boundary); the values `All`, `None`, `Preconditions`, `Boundary` are fixed now, the last two reserved. No change to the behaviour of v0.1. Recorded in the policy catalogue ([`IMPLEMENTATION_POLICIES.md`](../../how/IMPLEMENTATION_POLICIES.md)).
- **2026-10-23 — Module invariants removed from v0.1 (Tier 4, inline).** Author's decision: only types declare invariants; the section "Object/Module Invariants" became "Type Invariants" and its example was rewritten from `class` to `type` with `self.` (Orthon has no classes). The rule for when an invariant is checked is not changed by this entry; it remains under review in [`TRAIT_CONTRACT_UNIFICATION.md`](../../how/concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md) (Review decision 18) and [EDR-098](../../how/decision_records/architecture/EDR-098-trait-contract-unification.md).
- **2026-10-23 — Purity of condition expressions defined (Tier 4, inline).** Author's decision (variant B): named conditions stay; purity is checked against a closed list of effectful operations, transitively; members of the contract being declared are allowed if every implementation passes the check; through a generic bound or `dyn` only members of the same contract; the effect model replaces the list in v0.2. The accepted bullets "no I/O, no non-deterministic functions, only pure functions" now have a mechanism.
- **2026-10-23 — Contracts for generators and async functions (Tier 4, inline).** Author's decision (variant b): in v0.1 only `requires`, checked at the call before the iterator or future is created; `ensures` and `invariant` forbidden with a hint. Stages: v0.2 `ensures` on async completion and conditions on generator elements; v0.3 invariants across suspension. The first bullet of "Where Contracts Apply" now names the exception.
- **Cross-reference:** Data-Oriented Programming, [EDR-090](../../how/decision_records/architecture/EDR-090-data-oriented-programming.md), principle 4 (schema separate from representation, see [`DESIGN_PRINCIPLES.md`](../../how/DESIGN_PRINCIPLES.md#schema-separate-from-representation)): a contract is a specification kept as a separate object and applied at function boundaries.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [x] `how/IMPLEMENTATION_POLICIES.md` (Contract Enforcement Policy added 2026-10-23)
- [ ] `how/process/DECISION_PIPELINE.md`
