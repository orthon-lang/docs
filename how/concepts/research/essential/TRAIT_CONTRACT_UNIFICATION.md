# Contract Unification (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the 2026-10-18 review of the CONTRACTS concept. Proposes that
> `trait` (type-level contract) and the function contract of EDR-056
> (method-level contract) are one construct applied to different subjects.
> Home: `how/concepts/research/essential/` — semantic hypothesis (essential
> tier). Its syntax questions are registered by pointer in the
> `how/syntax/` decision queue (EDR-087).
>
> **Last updated:** 2026-10-21
>
> **Related:** [`TRAITS.md`](../../../../what/concepts/TRAITS.md) (EDR-019),
> [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) (EDR-056),
> [`../important/CONTRACTS.md`](../important/CONTRACTS.md) (research draft),
> [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md) (EDR-080),
> [`EQUALITY.md`](../../../../what/concepts/EQUALITY.md) (EDR-017),
> [`STRUCTURAL_TYPING.md`](../../../../what/concepts/STRUCTURAL_TYPING.md) (EDR-044),
> [`FUNCTIONS.md`](FUNCTIONS.md), [`ANONYMOUS_FUNCTION.md`](ANONYMOUS_FUNCTION.md),
> [`CLOSURE_CAPTURE.md`](CLOSURE_CAPTURE.md),
> [`../important/TRAIT_BLANKET_IMPLEMENTATION.md`](../important/TRAIT_BLANKET_IMPLEMENTATION.md),
> [`EFFECT_FOOTPRINT.md`](EFFECT_FOOTPRINT.md),
> [`../deferrable/FRAME_CONDITIONS.md`](../deferrable/FRAME_CONDITIONS.md),
> [`../../../DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md) (Orthogonality, Explicitness, Data First),
> [EDR-090](../../../decision_records/architecture/EDR-090-data-oriented-programming.md) (DOP lineage),
> [review note: free function vs type](../../../../.planning/notes/2026-10-18-contracts-free-function-vs-type.md)

---

## Concept Design Review — progress

Review of this hypothesis against [`concept-design-review.md`](../../../concept-design-review.md). Steps 0–5 were prepared on 2026-10-21 by the drafting agent; the author confirms them at Step 7.

| Step | Status | Where |
|------|--------|-------|
| 0 Primitive decomposition and layer classification (pipeline Stages 3–4) | done | [Step 0](#step-0--primitive-decomposition-and-layer-classification) |
| 1 Idea / Problem | done | [Problem](#problem) |
| 2 Alternatives | done, Selection recorded | [Step 2](#step-2--alternatives-comparison-and-selection) |
| 3 Minimal Solution | done | [Step 3](#step-3--minimal-solution) |
| 4 Principle Check | done, no violation found | [Step 4](#step-4--principle-check) |
| 5 Examples | done | [Step 5](#step-5--examples-by-usage-context) |
| 6 Tooling Implications | done | [Step 6](#step-6--tooling-implications) |
| 7 Convergence Check (syntax review, edge cases, sign-off; Open Question 10 closed by decision 18) | in progress: decisions 19–24 recorded (2026-10-22); the author's sign-off is pending | [Step 7](#step-7--convergence-check) |
| 8 EDR (EDR-098) | drafted 2026-10-22, Status Proposed; the author's acceptance is pending | [EDR-098](../../../decision_records/architecture/EDR-098-trait-contract-unification.md) |

## Problem

The specification already treats one idea in two places:

1. **Traits.** [`TRAITS.md`](../../../../what/concepts/TRAITS.md) defines a trait as "a behavioural contract that types can implement": a named set of operations a type promises to provide, with explicit conformance (`impl Trait for Type`), coherence, and static or dynamic dispatch.
2. **Function contracts.** [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) (EDR-056) defines `requires` / `ensures` / `invariant` as clauses written inside a function signature.

The 2026-10-18 review accepted that contracts should become **named objects** that a function refers to (`obeys`), separate from the function body, with reuse across many implementations. Taken alone, that adds a third mechanism with its own conformance check, its own Liskov rules, and its own coherence story, parallel to traits. Three concrete costs follow:

- **Duplicate machinery.** "This element declares that it satisfies a named specification" would exist twice, with the same rules written twice (signature match, weaken-pre / strengthen-post, at most one declaration per pair, orphan rule).
- **Contracts on function arguments have no home.** Higher-order contracts are deferred to v0.2 (EDR-056, open question 1) because nothing in the language can say "this argument is a function that satisfies this specification".
- **Properties of a whole trait have no home.** The Transitivity Invariant in [`EQUALITY.md`](../../../../what/concepts/EQUALITY.md) quantifies over three values of one type. It cannot be a `requires` / `ensures` (one call) or an `invariant` (one state), so it lives in prose.

The project's own discipline ([`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md) Stage 3) also requires a new construct to decompose into existing primitives. A separate contract-object construct currently has no such decomposition.

### Classical problem and scientific grounding (Step 1)

The problem maps to a known programming-language theory problem: **behavioral subtyping and the separation of a specification from its implementations.** A signature fixes shapes but not behaviour, so an implementation can match a signature and still break the assumptions of its callers. The rule that makes substitution safe — an implementation may weaken preconditions and strengthen postconditions — was stated by Liskov and Wing (*A behavioral notion of subtyping*, ACM TOPLAS, 1994). Meyer's Design by Contract (*Applying Design by Contract*, IEEE Computer, 1992) attached such specifications to declarations. The Expression Problem, already registered in [`LANGUAGE_PROBLEMS.md`](../../../../what/LANGUAGE_PROBLEMS.md), is adjacent (traits) but is not the same problem. The registry had no entry for this one; it was added on 2026-10-21. The citations are from general knowledge and are listed under "Not verified".

## Hypothesis

**One construct — provisionally `contract` — names a set of operations with optional conditions. A program element declares conformance with one keyword — provisionally `obeys`. The kind of contract (type-level or method-level) is not a property of the declaration; it is determined by the application site.**

Supporting premise: **a function is syntactic sugar for a unique anonymous type with one callable member.** Then a contract on a function is simply a contract with one callable member applied to that type, and a trait is the multi-member case.

Consequences claimed by the hypothesis (to be tested, not assumed):

1. One conformance mechanism, one set of Liskov and coherence rules, for types and functions.
2. Contracts on function arguments are ordinary generic bounds (`<Sqrt as F>`), so higher-order contracts need no new feature. (A v0.2 consequence: deferred by Review decision 15, not part of v0.1.)
3. The error "wrong kind of contract applied" disappears: the only error is structural mismatch (a required member is missing or has the wrong signature).
4. The separate contract-object construct is a Language Pattern (Level 2) over existing primitives, not a new primitive.

## Model (What)

> Syntax in this section is **provisional**. Spellings (`contract`, `obeys`, parameter and return notation) belong to the syntax pipeline and are coupled to the open entries in [`how/syntax/`](../../../syntax/README.md). Semantics are what this hypothesis proposes.

### One construct, kind decided by application

```orthon
contract Printable                              // several members: type-level
    fun format(self) -> String

contract Sqrt(x: Float) -> root: Float          // short form: one callable member
    requires x >= 0.0
    ensures root >= 0.0
```

| `X obeys C` where X is… | What the compiler checks |
|---|---|
| a type | every member of `C` is provided as an operation of `X` with a matching signature |
| a function (named or lambda) | `C` has exactly one callable member; the function's signature matches it |
| a method | same as a function; the receiver is part of the callable's parameters |

No rule of the form "contracts of kind K apply only to subject S" exists. `Float obeys Sqrt` fails because `Float` has no callable member `call(x: Float) -> Float`, by the same check that rejects a missing method.

### A function is a unique anonymous type

```orthon
fun sqrt_newton(x: Float) -> Float obeys Sqrt
    ...

// desugaring (illustrative)
struct sqrt_newton_T                                  // anonymous; closures hold captured values as fields
impl Callable(Float) -> Float for sqrt_newton_T
    fun call(x: Float) -> Float ...
conformance Sqrt for sqrt_newton_T                    // what `obeys` expands to
let sqrt_newton = sqrt_newton_T{}
```

The callable member has no receiver, as in the constrained-type desugaring (`fun call(v: Int)`): in v0.1 the anonymous type has no observable state of its own. This matches the short form of a contract (Review decision 24). How a receiver is spelled in members of type contracts (`fun size()` or `fun size(self)`) is a separate syntax question.

The mechanism already exists in the specification: [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md) desugars `type Age = Int requires ...` into a `struct` plus `impl Callable(Int) -> Age for Age`, and closures already capture explicitly. The anonymous type is not nameable by the programmer; `obeys` is the only surface that mentions it.

### Inline clauses and the short form

- Clauses written directly in a function signature (`requires` / `ensures`) are an **anonymous contract** satisfied only by that function. They are the common case and stay valid unchanged.
- `contract Name(params) -> result` is sugar for a contract with one callable member `call` (Review decision 24). Parentheses are mandatory. The short form has no receiver, its kind is always `fun`, and it is matched by call signature under any function name. A contract with several members, or with a mutating member, is written as a list of members; there members are matched by name and signature.
- A header with conditions only and no parentheses is an error in v0.1. A contract that attaches conditions to a position, whatever the member name, is deferred to v0.2 with its own spelling; named predicates cover reuse until then.

```orthon
contract Sqrt(x: Float) -> result: Float     // ok: short form
contract Positive -> result: Int             // ERROR in v0.1: no parentheses, conditions only
contract Area(self) -> result: Float         // ERROR: a receiver belongs to a member of a list
```
- A named contract is the reuse form: one specification, many implementations, tests derived once.

### Conformance rules (inherited, not new)

| Rule | Source |
|---|---|
| Signature of the element matches the contract's member | TRAITS (explicit satisfaction) |
| An implementation may weaken `requires` and strengthen `ensures`; stronger `requires` is an error | CONTRACTS principle 5 (Liskov) |
| At most one `obeys C` per (element, C) | TRAITS (coherence) |
| The element or the contract must be local to the declaring module | TRAITS (orphan rule) |
| `obeys A, B` is allowed; conditions of both apply | TRAITS (composition via bounds) |
| Dispatch is static by default; `dyn C` is opt-in and visible | TRAITS (Dispatch Policy) |

### Taxonomy of contract kinds

"Contract" becomes the umbrella term. Kinds differ by subject, content, and when they are checked.

| Kind | Subject | Content | Conformance | Checked |
|---|---|---|---|---|
| Type-level contract (today's trait) | a type | member signatures, associated types, defaults | `obeys` on a type | compile time (shape); member conditions as for functions |
| Method-level contract | a call | `requires` / `ensures` | `obeys` on a function, or inline | statically where provable, otherwise debug |
| Value constraint | a value | predicate over `v` | constrained type (EDR-080) | at construction |
| Invariant of mutable state | a mutable type or module | predicate over `self` | `invariant` in the type | at the boundary of public operations |
| Multi-instance contract (v0.2) | several values or calls | property quantified over inputs | declared in a contract | property-based generation, debug |

Adjacent, not part of this hypothesis: module-level effects and dependencies ([`EFFECT_FOOTPRINT.md`](EFFECT_FOOTPRINT.md), hypothesis; [`../MODULE_HEADER_FILE.md`](../MODULE_HEADER_FILE.md), draft) and mutation scope ([`../deferrable/FRAME_CONDITIONS.md`](../deferrable/FRAME_CONDITIONS.md), draft).

### Multi-instance contracts (provisional name: *law*) — deferred to v0.2

> **Deferred to v0.2 (Review decision 17).** Not part of v0.1; kept here as the analysis for later.

A condition that quantifies over several values or calls: transitivity of `Eq` over three values, the round trip `from_json(to_json(x)) == x` for `Serialize` / `Deserialize`. It is a contract with no subject function and with free inputs:

```orthon
contract Eq
    fun ==(self, other: Self) -> Bool

    law transitive(a: Self, b: Self, c: Self)          // keyword provisional
        requires a == b and b == c
        ensures a == c
```

It is not checked at a call boundary. It is checked by generating inputs (property-based testing) or by debug-mode assertions, as EQUALITY already allows ("the compiler may insert runtime checks in debug mode"). The name `law` is considered too heavy; candidates are listed in Open Question 3.

## Step 5 — Examples by usage context

Syntax is provisional (see the note under Model). The examples follow the categories of the Concept Design Review: minimal, typical, edge cases, incorrect usage; the equivalent canonical forms are shown together at the end.

### Minimal

```orthon
contract Sqrt(x: Float) -> result: Float
    requires x >= 0.0
    ensures result >= 0.0

fun sqrt_newton(x: Float) -> Float obeys Sqrt
    ...

contract Printable
    fun format(self) -> String

User obeys Printable
    fun format(self) -> String
        return "User({self.name})"
```

### Typical

```orthon
// Many implementations of one specification (function level)
fun sqrt_newton(x: Float) -> Float obeys Sqrt
    ...
fun sqrt_table(x: Float) -> Float obeys Sqrt
    ...

// Methods of a contract carry conditions; implementations inherit them
//    (mutating methods are `proc`; relations to the state before the call
//    need `old`, which is deferred: see CONTRACT_OLD_STATE.md)
contract Stack<T>
    proc push(item: T)
        requires self.size < self.capacity
        ensures self.top() == item
    proc pop() -> item: T
        requires not self.is_empty()

// Generic bound on types (unchanged form)
fun print_all<Printable as T>([T] items)

// Inline anonymous contract (EDR-056 with the named result of EDR-097)
fun withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    requires amount <= balance
    ensures result == balance - amount

// Several contracts
fun sqrt_safe(x: Float) -> Float obeys Sqrt, Logged
    ...
```

### Edge cases

```orthon
// A different result name in the implementation is allowed;
// inherited clauses use the name of the declaring signature
contract Shape
    fun area(self) -> result: Float
        ensures result >= 0.0

Circle obeys Shape
    fun area(self) -> a: Float
        return 3.14159 * self.r * self.r

// Error-union result: the name binds the success value; ensures is checked on success only
fun read_config(path: String) -> cfg: !Config
    ensures cfg.version >= 1

// An implementation may add a stronger ensures
fun sqrt_bounded(x: Float) -> root: Float obeys Sqrt
    ensures root <= x + 1.0

// Conformance declared outside the type's own declaration
// (orphan rule: the contract is local to the declaring module)
ForeignType obeys LocalContract
    ...
```

### Incorrect usage

All of these reduce to structural mismatch or to an inherited rule:

```orthon
Float obeys Sqrt                             // ERROR: Float has no callable member call(x: Float) -> Float
fun f(x: Int) -> Int obeys Printable         // ERROR: function has no member format(self)
fun g(x: Float) -> Float obeys Sqrt
    requires x > 10.0                        // ERROR: stronger requires than the contract
User obeys Printable                         // ERROR if format is missing or its signature differs
```

### Deferred to v0.2 (not part of v0.1)

Review decisions 15 and 17.

```orthon
// Dynamic dispatch on a function value
fun apply_dyn(f: dyn Sqrt, x: Float) -> Float
    return f(x)

// Higher-order contract: a bound on a function-valued argument
fun map_roots<Sqrt as F>(f: F, xs: [Float]) -> [Float]

// Lambda that declares conformance (spelling provisional)
let root = fn(x) -> newton(x) obeys Sqrt
```

Multi-instance contracts: see the deferred subsection of Model.

### All canonical forms

These three spell the same thing; the first two are the common case:

```orthon
// 1. Inline clauses: the anonymous contract of this function
fun withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    ensures result == balance - amount

// 2. Named contract (short form) plus conformance
contract Withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    ensures result == balance - amount
fun withdraw(balance: Int, amount: Int) -> Int obeys Withdraw

// 3. Named contract (long form): one callable member
contract Withdraw
    fun call(balance: Int, amount: Int) -> result: Int
        requires amount > 0
        ensures result == balance - amount
fun withdraw(balance: Int, amount: Int) -> Int obeys Withdraw
```

Value constraints (`type Age = Int requires v >= 0 && v <= 150`) stay as they are: their subject is a value and they are checked at construction. They are a different kind, not a member of this unification.

## Business-rule use

A named contract is a **business rule**: named, reusable, and independent of any implementation. This follows [`VISION.md`](../../../../why/VISION.md) ("the language defines contracts; implementations fulfill them") and DOP principle 4 (EDR-090): the specification is a separate named object applied at boundaries.

| Rule property | Provided by the design |
|---|---|
| Has a name | named contract object |
| Separate from implementation | contract object plus `obeys`; the binding is in the signature, so the shape is checked by the compiler |
| One rule, many places | one contract obeyed by several functions; shared named predicates |
| Rule over a value | constrained type (EDR-080) |
| Rule over state | `invariant` |
| Rule over several values | multi-instance contract (deferred to v0.2, Review decision 17) |
| Readable in domain vocabulary | named pure predicates (`is_positive`, `covered_by`) |
| Machine-readable export | Schema Provider; the structured export of the [`SEMANTIC_ANNOTATIONS.md`](../deferrable/SEMANTIC_ANNOTATIONS.md) draft |

```orthon
// module of rules: depends on domain types only, never on implementations
fun is_positive(amount: Money) -> Bool
    return amount > Money(0)

fun covered_by(balance: Money, amount: Money) -> Bool
    return amount <= balance

contract Withdraw(balance: Money, amount: Money) -> new_balance: Money
    requires is_positive(amount)               // "the amount is positive"
    requires covered_by(balance, amount)       // "the balance covers the amount"
    ensures new_balance == balance - amount    // "the balance decreases by the amount"

// module of implementations
fun withdraw(balance: Money, amount: Money) -> Money obeys Withdraw
    return balance - amount

fun withdraw_with_fee(balance: Money, amount: Money) -> Money obeys Withdraw
    return balance - amount - FEE              // violates ensures: a contract-derived test shows it
```

If the business decides to charge a fee, the rule changes in one place, and every implementation that no longer fits is found by checking against it. The rule is not edited inside the code that enforces it.

**Dependency direction.** The module of rules must not import implementation modules; otherwise the rules are tied to code again. This is a project rule (or a lint), not a language rule.

**Honest limits.** "Self-describing" does not mean that non-programmers read the expressions: contract expressions are code. The realistic gain is domain-vocabulary names for rules and predicates, and documentation generated from them.

## Evidence

### In the repository

- [`TRAITS.md`](../../../../what/concepts/TRAITS.md) (line 44) already says: "A trait is a behavioural contract that types can implement."
- [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md) decomposes a new construct into `struct` + `Callable` with "no new primitives" — the same style of decomposition used here.
- [`EQUALITY.md`](../../../../what/concepts/EQUALITY.md) states the Transitivity Invariant without a formal home for it.
- [`FUNCTIONS.md`](FUNCTIONS.md) is a draft, and its open question 4 asks how functions interact with the type system and whether function types are a distinct thing. This hypothesis proposes an answer; it does not conflict with an accepted decision.
- [EDR-090](../../../decision_records/architecture/EDR-090-data-oriented-programming.md) records DOP lineage. Principle 4, "separate data schema from data representation" (schema optional, applied at boundaries), is the same move applied to behaviour: the specification is a named object separate from the implementation.

### In the literature (searched 2026-10-18)

| Source | Finding | Bearing |
|---|---|---|
| Racket contract system ([reference](https://docs.racket-lang.org/reference/contracts.html)) | contracts are values built by combinators, attached at module boundaries via `contract-out` | precedent for contracts as separate named objects applied at boundaries |
| Findler & Felleisen, ICFP 2002 ([tech report](https://www.cs.northwestern.edu/~robby/pubs/papers/ho-contracts-techreport.pdf)); Dimoulas et al. POPL 2011; Lazarek et al. POPL 2020 | blame is assigned between two parties; semantics for dependent higher-order contracts are subtle; blame does not reliably locate bugs | caution for higher-order contracts: do not promise precise diagnostics |
| Clojure spec `fdef` ([docs](https://clojuredocs.org/clojure.spec/fdef)) | function specs live in an external registry bound by name; `instrument` checks only `:args`, `:ret` and `:fn` need the test runner | binding at the declaration (`obeys`) avoids drift and partial enforcement; each clause kind needs a defined check point |
| Dafny ([blog](https://dafny.org/blog/2023/08/14/clear-specification-and-implementation/), [FAQ](https://dafny.org/dafny/HowToFAQ/FAQImportAbstractModule)) | abstract interface with `ensures` and no body; implementations are checked for adherence; per-implementation ghost state is the cost | one specification, many implementations, compiler-checked adherence |
| OCL ([lecture notes](https://st.inf.tu-dresden.de/files/teaching/03-swe-ocl.pdf)) | contracts as a side-effect-free specification language, used for business rules | contracts as readable specifications separate from code |
| SPARK ([user guide](https://docs.adacore.com/spark2014-docs/html/ug/en/source/subprogram_contracts.html)) | `Pre` / `Post` aspects attach to declarations in the package specification | the contract belongs to the declaration, not the body |
| Dafny / Whiley named results; C++ P1323 | the result is named in the signature or in the postcondition | feeds the open result-naming question |

### Not verified (to be checked before an EDR)

- Eiffel's inheritance rule (`require else` / `ensure then`) is cited from secondary sources only.
- That a precondition of a public routine may mention only names visible to callers (Eiffel) is from general knowledge.
- That Rust gives every function item a unique type implementing the `Fn` traits is from general knowledge.
- That the keyword `contract` evokes Solidity (a class-like unit) for LLMs is an assumption; it is a generability risk to test, not a finding.
- Primary papers were not read in full; findings above come from search summaries.
- The classical-problem citations of Step 1 (Liskov and Wing, 1994; Meyer, 1992) and the Rust and Haskell rows of Step 2 are from general knowledge.

## Step 2 — Alternatives (comparison and selection)

Candidates in the canonical set of [`concept-design-review.md`](../../../concept-design-review.md): a language construct, syntactic sugar, a library or composition, do nothing, and at most one wild alternative. The IDs A0–A4 are kept from earlier sections; A5 is the wild alternative.

| ID | Canonical class | Candidate | Notes |
|----|-----------------|-----------|-------|
| A0 | language construct | Separate contract-object construct beside `trait` / `impl`: new `contract` / `obeys` for functions | two conformance mechanisms; the duplication this hypothesis targets |
| A1 | language construct (unified; pattern over conformance) | **This hypothesis:** `contract` replaces `trait`, `obeys` replaces `impl` | one mechanism; largest cascade |
| A2 | language construct (unified; pattern over conformance) | Keep the name `trait` as the single construct; `obeys` is the application keyword for types and functions | same semantics as A1; smaller cascade; `trait Sqrt(x: Float) -> Float` reads oddly for a function specification |
| A3 | library / composition | Pattern only: a function contract is a trait with a `Callable` member using today's `impl` | no new keywords; needs nameable unique function types; verbose per implementation |
| A4 | do nothing | Inline clauses only (EDR-056 as amended by EDR-097) | loses reuse and the specification-first workflow |
| A5 | wild | Contracts bound from outside by name, as a registry (Clojure spec `fdef`, OCL) | action at a distance; drift between the registry and the declaration |

Naming (`contract` versus `trait`, A1 versus A2) is a syntax question; the semantic selection below does not depend on it.

How other languages solve it, with the closest candidate:

| Language | Mechanism | Closest candidate |
|----------|-----------|-------------------|
| Racket | first-class contracts attached at module boundaries | A1 / A5 |
| Clojure | spec `fdef` in an external registry bound by name | A5 |
| Dafny | abstract interface with `ensures`, adherence checked | A1 |
| OCL (UML) | constraints bound to model elements from outside | A5 |
| Eiffel, SPARK | conditions attached to the declaration | A4 |
| Rust | traits for shape, no behavioural contract (general knowledge, not verified) | status quo traits |
| Haskell | typeclasses, laws stated in documentation (general knowledge, not verified) | status quo traits |

Scoring against fixed criteria (+ favourable, 0 neutral, − unfavourable):

| Criterion | A0 | A1 | A2 | A3 | A4 | A5 |
|-----------|----|----|----|----|----|----|
| Orthogonality (one concept, one construct) | − | + | + | + | + | 0 |
| Minimal Core (no new primitive) | − | + | + | + | + | 0 |
| DRY (no duplicate mechanism) | − | + | + | + | + | 0 |
| Explicitness | + | + | + | + | + | − |
| New keywords (net) | − | 0 | 0 | + | + | 0 |
| Reuse of one specification across implementations | + | + | + | 0 | − | + |
| Home for higher-order contracts (v0.2) | − | + | + | 0 | − | 0 |
| LLM Generability (prior for the keywords) | 0 | − | 0 | + | + | 0 |
| Cascade over accepted documents | + | − | 0 | + | + | + |
| Semantic risk | 0 | 0 | 0 | − | + | − |

Notes. A3 scores 0 on reuse because every implementation needs a nameable wrapper type (a Strategy-like object); its semantic risk is the verbosity that makes function-level conformance impractical. The semantic risk of A5 is the drift and partial enforcement seen in Clojure spec, where `instrument` checks only `:args`.

**Selection: A1 or A2 — the unified construct** (the choice between them is a naming decision for the syntax pipeline). Rationale: it is the only candidate that solves the problems of Step 1 — one conformance mechanism, a home for reuse and for the higher-order path — without a new primitive and without a duplicate mechanism. A3 is the fallback if the F2 definition is rejected. A4 is the reject baseline: it leaves the problems open. A0 exits at pipeline Q3. A5 is rejected on explicitness and drift.

The tables are carried verbatim into the "Alternatives Considered" field of the EDR (Step 8).

## Step 3 — Minimal Solution

Selected alternative: the unified construct (A1 or A2; naming open). Minimal semantic addition for v0.1:

1. **Conformance applies to a second kind of subject.** Today a type declares conformance to a trait. After the change a named function declares conformance to a named contract with one callable member, under the same rules (signature match, weaken `requires` and strengthen `ensures`, coherence, the orphan rule).
2. **F2 as the definition.** A function declaration is sugar for an anonymous type with one callable member. This explains function-level conformance and decomposes it (Step 0). It adds no new surface for types.
3. **Nothing else is added.** Trait semantics are unchanged. Inline clauses remain the anonymous contract of the function (EDR-056, EDR-097). The short form `contract Name(params) -> result` is sugar for a contract with one callable member.

Not in the minimal solution, each decided or deferred elsewhere: `old` (EDR-097); contracts through generics, `dyn`, collections of functions and lambdas (v0.2, Review decision 15); multi-instance contracts (v0.2, Review decision 17); imposing a rule from outside (deferred, Review decision 13); the words `contract` and `obeys` (syntax pipeline); the rename cascade (a tracked task after acceptance).

Orthogonality check: this is one addition (conformance for functions), because a named function contract is an inline clause set given a name, and F2 is its definition. If the review finds that naming contracts and unifying with traits are separable, the concept can be split; no such split is needed today.

Merge outcome (Review decision 16): if accepted, TRAITS and CONTRACTS become one concept document.

## Step 0 — Primitive decomposition and layer classification

Pipeline Stages 3 and 4 ([`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md)), completed on 2026-10-21 by the drafting agent; the author confirms them at Step 7.

### Stage 3 — primitive decomposition

Available primitives: `identifier`, `literal`, `assignment`, `function`, `call`, `attribute access`, `scope`, `reference`, `pack`, `unpack`, `operator definition` ([`PRIMITIVE_BLOCKS.md`](../../../../what/PRIMITIVE_BLOCKS.md)).

| Part of the unified construct | Decomposition | Source of the pattern |
|-------------------------------|---------------|-----------------------|
| Contract declaration (members) | `function` signatures + `identifier` + `scope` | TRAITS registry entry (trait declaration) |
| Conformance declaration (`obeys`) | `function` implementations + `scope` | TRAITS registry entry (`impl`) |
| Function-level conformance | the function declaration as `pack` (captured values, possibly none) + `function` (the callable member); conformance as for a type | F2 as a definition; same shape as the constrained-type desugaring (EDR-080) |
| Named result | `identifier` declared in the signature + `scope` (visible only in `ensures`) | EDR-097 |
| Conditions (`requires`, `ensures`) | compiler-recognized signature modifiers plus pure-expression analysis | CONTRACTS registry entry; unchanged by this hypothesis |

Result: **no new primitive.** What stays compiler-level, exactly as today: coherence and the orphan rule, bound resolution and dispatch (TRAITS), contract purity analysis and enforcement (CONTRACTS). The unification moves none of these.

### Stage 4 — layer classification

- The type-level part keeps today's classification of traits (Language, D-03); the unification changes its name and the subjects it applies to, not its level.
- The function-level part is a **Level 2 Language Pattern**: a composition of Level 0–1 constructs with no new semantics. Composition formula:

```text
fun f(params) -> result: T obeys C   =   pack{captures} + function(call: params -> T) + conformance(C)
```

- F2 is used only as the definition behind this formula. It adds no Level 0 or Level 1 construct. Its relation to the structural function type, generics and dispatch is deferred to v0.2 (Review decision 15).

These results agree with the pipeline verdict (Q3: yes, conditionally).

## Decision Pipeline Run

> **Pipeline run (2026-10-19).** Applied to the hypothesis as a whole, with the
> sub-features that can end differently named explicitly. Per
> [`DECISION_PIPELINE.md`](../../../process/DECISION_PIPELINE.md). This is
> provenance for the Concept Design Review, not a binding verdict; the
> per-concept record in [`DECISION_LOG.md`](../../../gates/DECISION_LOG.md) is
> written only when an EDR is filed.

Sub-features:

- **F1** — one conformance construct for types and functions (the unification).
- **F2** — premise: a function is a unique anonymous type with a callable member.
- **F3** — named contract objects (accepted in review as a kind of object).
- **F4** — multi-instance contracts (provisional name *law*).
- **F5** — spellings (`contract`, `obeys`): a syntax question, not a pipeline subject.

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Two mechanisms (traits, function contracts) for one idea — a named specification that an element declares it satisfies — and a third arriving with contract objects. Missing homes: contracts on function arguments (higher-order contracts, deferred in EDR-056) and properties over several values (the Transitivity Invariant in EQUALITY). Reuse of one specification across implementations. |
| Q2 | Language problem or library problem? | **Language.** Conformance checking, coherence and the Liskov rules are compiler services; EDR-019 and EDR-056 already classify both parts as Language. For F4 the declaration is Language and input generation is toolchain or Standard Library. |
| Q3 | Can it be solved with existing primitives? | **Yes, conditionally.** With F2 a function contract is `struct` + `Callable` + trait conformance, the composition already used by constrained types (EDR-080). Without F2: no. **A separate new contract-object construct (alternative A0) exits here:** it decomposes into existing primitives, so it is not a new language feature. |
| Q4 | Does it violate any Design Principle? | No violation found. Orthogonality, Uniformity, Explicitness, Minimal Core (Core Change Criterion, if Q3 holds), Parsimony, DRY, Schema Separate from Representation (DOP) and Named Before Symbolic read in favour. Semantic Purity: one word with one semantic role (the pending wording amendment from "one meaning" to "one semantic role" matters here). Caution — POLA: users of `trait` / `impl` meet a rename, and "a function has a type" is new. Neutral — Data Is Immutable by Default; invariants of mutable state sit outside DOP (Open Question 10, closed by Review decision 18). |
| Q5 | Does it add new semantics (vs sugar)? | **Mixed.** New: F2 (functions have unique nominal types), function-level conformance, F4. Sugar: the short form `contract Name(params) -> r`, inline clauses as an anonymous contract, `obeys` on a function as shorthand for conformance of its anonymous type. |
| Q6 | Can it be expressed through composition? | **Yes for F1, given F2:** `struct` + `Callable` + conformance. F4: partly — a contract with free inputs plus a generator in the toolchain. |
| Q7 | Can it be syntactic sugar over existing primitives? | **Partly.** F3 and the short and inline forms are sugar over F1 + F2. F1 itself is a unification of existing constructs, not sugar. |
| Q8 | Is this an optimisation, not semantics? | No. Eliding checks in release builds is a Contract Enforcement Policy matter and is unchanged. |
| Q9 | Does it affect backward compatibility? | Pre-v1.0: no external users. Consistency with accepted decisions: EDR-019 (keyword and conformance syntax superseded in part), EDR-056 (amended; `old` already deferred), EDR-044, EDR-029, EDR-080; 52 files reference EDR-019 or `TRAITS.md`, 35 `impl ... for` lines in 17 files. Forward: reserves `obeys` and, under A1, `contract`. |
| Q10 | Is it worth adding at all? | **Yes, as a unification** (A1 or A2): removes duplicated machinery and gives higher-order contracts and a home for multi-instance properties without a new primitive. **Conditional on F2.** F4: **DEFER** (v0.2 candidate). A separate construct (A0): not worth adding (Q3). |

**Verdict:** Pipeline **PASS** for F1 and F3 in the unified form, **conditional on F2 as a definition by desugaring only** (narrowed 2026-10-19, Review decision 15); F4 **DEFER**; F5 goes to the syntax pipeline ([`CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md)).

Exit path per alternative:

| Alternative | Result |
|---|---|
| A0 separate contract-object construct | exits at Q3 (decomposes into existing primitives): at most a Language Pattern, not a new construct |
| A1 `contract` replaces `trait`, `obeys` replaces `impl` | passes; F2 needed as a definition only |
| A2 keep `trait`, `obeys` for types and functions | passes; F2 needed as a definition only |
| A3 pattern over trait and `Callable` with today's `impl` | passes; equivalent to A1 without new keywords; F2 needed as a definition only |
| A4 inline clauses only (EDR-056 as accepted) | passes trivially; fails Q1 (leaves the listed problems open) |

**Conditions before the next stage:**

1. **F2 (narrowed 2026-10-19).** F2 is needed only as the definition that explains `obeys` on a function. Its consequences — the relation to the structural function type, generics, `dyn`, lambdas — are deferred to v0.2 (Review decision 15; Open Question 1). It is no longer a blocker. The definition is to be pointed to from the `FUNCTIONS.md` review as the v0.1 answer to its open question 4: each function declaration is sugar for an anonymous type with one callable member; function-type notation stays undecided.
2. **Transparency — gate guard.** None of the four rejected proposals (EDR-075 prototype, EDR-076 significant whitespace, EDR-077 dynamic typing, EDR-078 class or structure as primary composition) is similar to this one. The alternatives rejected inside TRAITS (class inheritance, structural interfaces by default, unrestricted orphans) are not reopened: the hypothesis keeps explicit conformance, coherence and the orphan rule.
3. **Transparency — architecture metric.** More than one substantive semantic change after initial gate approval triggers a mandatory architectural review. The accepted CONTRACTS concept already changed once (the deferral of `old`, 2026-10-19, an editorial amendment); this hypothesis would be a second change for CONTRACTS and the first for TRAITS. Whether the `old` deferral counts is the author's call; the recommended handling is to treat the Concept Design Review of this hypothesis as that architectural review.
4. **Syntax.** F5 follows the semantic decision through the syntax pipeline.
5. **Pipeline Stages 3 and 4.** The primitive decomposition check against `PRIMITIVE_BLOCKS.md` and the layer classification (Level 2 for the function-level part) were completed in Step 0 of the Concept Design Review (2026-10-21).

## Step 4 — Principle Check

Mapped to [`DESIGN_PRINCIPLES.md`](../../../DESIGN_PRINCIPLES.md) on 2026-10-21 by the drafting agent; the author confirms it at Step 7. `DESIGN_PRINCIPLES.md` is closed for modification ([`DECISION_PROCESS.md`](../../../process/DECISION_PROCESS.md)): the check asks whether this concept violates a principle, in which case a Tier 1 EDR would be needed and solo-author acceptance would not be possible.

| Principle | Result | Reading |
|-----------|--------|---------|
| Data First | Pass | Neutral: contracts describe behaviour, not data |
| Data Is Immutable by Default | Pass | Unchanged; invariants of mutable state are kept with a narrower check (Review decision 18) |
| Schema Separate from Representation | Pass | The specification is a separate named object applied at boundaries (EDR-090, principle 4) |
| Intent Over Implementation | Pass | The contract states intent; implementations vary |
| Orthogonality | Pass | One construct applied to different subjects instead of overlapping constructs |
| Semantic Purity | Pass | One word with one semantic role for conformance of types and functions. The pending wording amendment from "one meaning" to "one semantic role" is a separate Tier 1 matter. The `->` overload does not enter v0.1 because lambdas are deferred |
| Simplicity | Pass | Fewer mechanisms; the added concept (F2) is a definition, not a surface |
| Explicitness | Pass | Conformance is declared; the result is named in the signature (EDR-097) |
| Consistency | Pass | Similar concepts (type and function conformance) look similar |
| Uniformity | Pass | Equivalent concepts are written in equivalent ways |
| Explicit Semantics | Pass | No hidden conversion: the anonymous type of a function is not observable in v0.1 |
| Declarative With Static Guarantees | Pass | Shape is checked at compile time; conditions are proved statically where possible (EDR-056) |
| Named Before Symbolic | Pass | A word; no new symbol |
| Minimal Core | Pass | Holds if the Step 0 decomposition holds: no new primitive (Core Change Criterion) |
| Justified Solutions | Pass | The solution matches the stated problem (Step 1) |
| Transparency | Caution | Gate guard: none of the four rejected proposals (EDR-075 to EDR-078) is similar. Architecture metric: this would be a second substantive change to CONTRACTS and the first to TRAITS, which triggers an architectural review; treating this Concept Design Review as that review is the recommended handling (the author's call) |
| Principle of Parsimony | Pass | Fewer entities than A0 |
| Idempotent Generation | Pass (unmeasured) | A named contract is a fixed specification to generate against; no measurement yet (LLM Generability Gate) |
| Representation Symmetry | Not applicable | |
| Deterministic Behavior | Pass | Unchanged from EDR-056: eliding checks in release builds is a Contract Enforcement Policy matter (an existing tension, not introduced here) |
| Stable Mental Model | Pass | One model: conformance to a named specification |
| Semantics Before Optimization | Pass | Semantics do not depend on optimization |
| Explicit Optimization | Not applicable | |
| Correctness Before Performance | Pass | Contracts serve correctness; enforcement cost is a Policy |
| SOLID | Pass | Small contracts per responsibility; rules do not depend on implementations (dependency inversion) |
| DRY | Pass | The main driver: one conformance mechanism |
| POLA | Caution | Users of `trait` / `impl` meet a rename; "a function has a type" stays hidden in v0.1, which limits the surprise |

Result: **no violation of a principle; two cautions** (Transparency, POLA). No Tier 1 EDR is required.

## Step 6 — Tooling Implications

Checked on 2026-10-21 against the questions of the Concept Design Review: does the concept enable a new tool, require tooling to be usable, or constrain the implementation repository? The author confirms the result at Step 7.

| Candidate | Decision | Where | Reason |
|-----------|----------|-------|--------|
| Hover and go to the contract at `obeys`, "find implementations" | note, no new file | [`LANGUAGE_SERVER.md`](../../../tooling/LANGUAGE_SERVER.md) | hover, go-to-definition and references exist; a contract is one more navigation target |
| Contracts in the symbol query (Schema Provider) | extend an existing document | [`ORTHON-DESCRIBE.md`](../../../tooling/ORTHON-DESCRIBE.md) | the tool already needs a queryable IR; a contract is one more entity |
| Contract-derived tests | **new Tooling Requirement** | [`ORTHON-TEST.md`](../../../tooling/ORTHON-TEST.md) (P2, M3) | no such tool exists; real impact on the specification (below) |
| Rule catalog (business-rule documentation) | no new file | `ORTHON-DESCRIBE.md`, an output mode | the same query as the previous row |
| Check that the module of rules imports no implementation | not a Tooling Requirement | candidate fitness function, [`FITNESS_FUNCTIONS.md`](../../../architecture/FITNESS_FUNCTIONS.md) | a project rule, not a language rule |
| Proof backend | note, no new file | [`ORTHON-PROVE.md`](../../../tooling/ORTHON-PROVE.md) | a named contract is a unit proved once; invariants checked after creation and `proc` reduce the obligations |
| Diagnostics (failed predicate, the `old` message, the unnamed-result hint) | not a Tooling Requirement | Static Analyser requirements; EDR-097 | compiler obligations already stated in the specification |
| LLM generability measurement harness | none now | the deferred measurement protocol | a step of the review process, not a product tool |

Result: one new Tooling Requirement and three notes in existing documents.

**Dependencies on the specification found by this step:**

1. **The IR keeps contracts and conformance as named, queryable entities**, so that the implementations of a contract can be enumerated ([`IR.md`](../../../architecture/IR.md)). Contract-derived tests, "find implementations" and the symbol query all need it.
2. **The source of a rule's description** is the predicate name or its comment (Review decision 12). No accepted concept defines doc comments; there are only deferrable drafts ([`LITERATE_PROGRAMMING.md`](../deferrable/LITERATE_PROGRAMMING.md), [`METADATA_ANNOTATIONS.md`](../deferrable/METADATA_ANNOTATIONS.md), [`SEMANTIC_ANNOTATIONS.md`](../deferrable/SEMANTIC_ANNOTATIONS.md)). Whether comments are retained in the AST is open (Open Question 16); a rule catalog works from names alone until it is decided.
3. **Contract-derived tests need values of parameter types to be generated.** Where that capability is defined — a trait, a compiler service, constrained types — is open (Open Question 15). It is recorded as a dependency in `ORTHON-TEST.md`.

## Step 7 — Convergence Check

Prepared 2026-10-22 by the drafting agent; the decisions below are the author's. The sign-off is **not** set here.

| # | Check | Status | Basis |
|---|-------|--------|-------|
| 1 | Alternatives documented | Done | Step 2: six candidates and the Selection row |
| 2 | Syntax reviewed | Done, with remarks | Remarks below; the words are decided by the syntax pipeline |
| 3 | Edge cases probed | Done | Six gaps found and closed by decisions 19–24 |
| 4 | Desugaring checked (Level 2) | Done | Step 0 formula; member kinds fixed by decision 19; the `call` expansion kept as a definition by decision 24 |
| 5 | Author's sign-off | **Pending** | The field is empty; agents do not set it |
| 6 | No ambiguities | Done for v0.1 semantics | Gaps D1–D6 closed; what remains is deferred explicitly (decisions 13, 15, 17) |

### Remarks for the syntax pipeline

1. `require` (dependencies, EDR-081) and `requires` (precondition) differ by one letter in the same tail position of a signature. Different meaning, same position: a risk for LLM generation. Recorded for [`CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md).
2. The short and the long form of a contract are two spellings of one thing (Syntax Principle 1). Accepted as explicit sugar by decision 24.
3. The signature tail already carries `require`, bounds and `->`; `obeys` joins it unless the separate-statement position (candidate P1) is chosen.

### Edge cases and the decision that closes each

| Area | Case | Outcome |
|------|------|---------|
| Mutation | `proc` member in a contract, implementation declared `fun` | Rejected: kinds must match (decision 19) |
| Errors | Contract promises `Float`, implementation returns an error union | Rejected: result must convert to the contract result (decision 20) |
| Errors | Condition violated, or the condition itself fails | `PROGRAM_ERROR`, two codes, different guilty side (decision 22) |
| Composition | `structural` satisfaction for a function | Not allowed; functions obey explicitly (decision 21) |
| Generics | `obeys` on a generic function | Allowed with linked parameters and bounds not stronger (decision 23) |
| Forms | Short or long form, contract without members | Both forms; markers allowed; conditions-only header is an error (decision 24) |
| State | Invariant checking | Decision 18 |
| `old` | `old` in a postcondition | Diagnostic of EDR-097 |

## Policy Footprint

| Policy Type | Role |
|---|---|
| Contract Enforcement Policy | when conditions are checked (compile time, debug, release; boundary against internal) |
| Contract Inheritance Policy | weaken `requires`, strengthen `ensures`, now one rule for types and functions |
| Dispatch Policy | static by default; `dyn C` for function values means dynamic call through the contract |
| Coherence Policy | orphan rule for conformance of functions and types |
| Subtyping Policy | a contract creates no subtype relation; `dyn C` is a handle |
| Representation Policy | how a function's anonymous type and captured values are represented |
| Allocation Policy | closure captures as fields of the anonymous type |

## Impact and cascade (counts as of 2026-10-18)

- 52 files under `what/` and `how/` reference EDR-019 or `TRAITS.md`.
- 41 `trait` declaration lines and 35 `impl ... for` lines (in 17 files) appear in code examples.
- Accepted records affected if A1 is chosen: EDR-019 (keyword and conformance syntax superseded), EDR-056 (amended: contract objects), EDR-044 (the `structural` opt-in is declared on the trait), EDR-029 (`@derive` generates `impl` blocks), EDR-080 (desugaring uses `impl Callable`).
- Syntax decisions coupled to this hypothesis: [`FUNCTION_ARGUMENT_SYNTAX.md`](../../../syntax/FUNCTION_ARGUMENT_SYNTAX.md), [`FUNCTION_RETURN_SYNTAX.md`](../../../syntax/FUNCTION_RETURN_SYNTAX.md), [`FUNCTION_RETURNING_FUNCTION.md`](../../../syntax/FUNCTION_RETURNING_FUNCTION.md), [`BOUNDS_IN_ANGLE_BRACKETS.md`](../../../syntax/BOUNDS_IN_ANGLE_BRACKETS.md), and [`TRAIT_BLANKET_IMPLEMENTATION.md`](../important/TRAIT_BLANKET_IMPLEMENTATION.md).

## Review decisions already taken (2026-10-18; author's decisions, no EDR yet)

1. Contracts become a **named kind of object** (subject to this unification).
2. Parameters are bound **by name**, as in the signature.
3. `self` exists **only for types**; free functions have no `self`.
4. The undefined approximate-equality glyph is removed from the specification; the canonical `sqrt` example uses an explicit tolerance and `ensures result >= 0.0` (quick task 261018-i9n).
5. Contract expressions use the **lambda form `fn(x) -> ...`** where a binder is needed; the colon-binder form is dropped. `fn` reads as "mini-function" ([`GLOSSARY.md`](../../../../what/GLOSSARY.md) Lambda entry).
6. Contracts are allowed on **any** function or method; rules differ by position (names visible to callers in `requires` of public API, Liskov rules on trait methods).
7. Separating the contract from the implementation is consistent with DOP principle 4 (EDR-090).
8. A comparison of free function and type is recorded in the [review note](../../../../.planning/notes/2026-10-18-contracts-free-function-vs-type.md).
9. **The result is named in the signature** (`-> result: Float`) and used as an ordinary name in `ensures` clauses (option B, decided 2026-10-19). The name is visible only in `ensures` clauses: not in `requires`, not in the body. The lambda binder in `ensures` (`ensures fn(r) -> ...`) is dropped, because it would shadow the signature name. Predicate-valued clauses (`ensures is_non_negative`) are not introduced in v0.1: an expression of function type in `ensures` is a compile error with a hint to call it on `result` (`is_non_negative(result)`).
10. **`old` is deferred out of v0.1** (decided 2026-10-19): it adds complexity now and gives little in return. The analysis is kept as a deferred hypothesis in [`../deferrable/CONTRACT_OLD_STATE.md`](../deferrable/CONTRACT_OLD_STATE.md). Postconditions in v0.1 speak about the state after the call and about the result. The accepted [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) still describes `old`; amending it is a separate step.
11. **A violation report names the rule** through the called predicate name: compiler diagnostics print the predicate that failed (for example `requires covered_by(balance, amount)`). No new feature is needed (decided 2026-10-19).
12. **Description and source of a rule come from the predicate name or its comment.** The `as` label is **not** introduced for rules (decided 2026-10-19).
13. **Deferred:** imposing a rule from outside on a foreign function (precedent: Racket `contract-in`) and a rule registry file. A comment may carry a reference to the rule; a separate module may use it to pull the description by reference; a registry file in the project may be added later. The form of the reference is not specified (decided 2026-10-19: mark and defer).
14. **`obeys` is accepted as a candidate to replace `impl`** (decided 2026-10-19). It is a candidate only: the surface form is decided in the syntax pipeline, tracked in [`../../../syntax/CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md), and is blocked on this hypothesis. It is justified only as a replacement for `impl`; adding it next to `impl` would give one concept two spellings (Syntax Principle 1).
15. **Deferred to v0.2** (decided 2026-10-19): using contracts through generics and function values — a function contract as a generic bound, `dyn` on function values, collections of contract-bearing functions, and `obeys` on lambdas. These are the higher-order contracts that EDR-056 already deferred to v0.2. The notation for function types is undecided in any case (`->` is overloaded, see [`FUNCTION_RETURNING_FUNCTION.md`](../../../syntax/FUNCTION_RETURNING_FUNCTION.md)), so this document does not write function types in new examples. The premise "a function is sugar for a unique anonymous type with one callable member" stays **only as a definition**: it explains `obeys` on a function and decomposes it into existing primitives (Pipeline Q3). Its consequences for generics, dispatch, and function-type notation are deferred. **v0.1 scope:** types obey contracts (replacing `trait` / `impl`); functions obey named contracts at their declaration (shape check, contract-derived tests, documentation).
16. **Merge of TRAITS and CONTRACTS** (decided 2026-10-21): if the unification is accepted, the accepted concepts TRAITS (EDR-019) and CONTRACTS (EDR-056, as amended by EDR-097) are merged into one concept document. How the unifying EDR relates to EDR-019 and EDR-056 (supersedes or amends) is stated in the EDR itself (Step 8).
17. **Multi-instance contracts are deferred to v0.2** (decided 2026-10-21). Open Question 4 is closed; the naming questions (Open Questions 3 and 14) are not needed for v0.1.
18. **`invariant` stays in v0.1 with a narrower check** (decided 2026-10-21; closes Open Question 10). An invariant of a type or module is checked after the value is created and after each `proc` exits. It is not checked for `fun` or `new` methods, which never mutate `self` ([`SEMANTIC_MODEL.md`](../../../../what/SEMANTIC_MODEL.md) § Mutation) and so cannot break it. Re-entry: while a `proc` runs, nested calls on the same object do not check the invariant on entry; it is checked when the outer `proc` exits. For an immutable value the rule is a constrained type (EDR-080), checked once at creation. The accepted `CONTRACTS.md` still says that invariants are checked on entry and exit of each public method; it is amended together with the unification (EDR-098).

19. **Kind of a member (Step 7, D1; decided 2026-10-22).** The declaration kind of an implementing method must equal the kind of the contract member it implements. The callable member of the short form is always the reading kind (`fun`); a contract that must describe a mutating operation uses the member-list form with the kind written out. For a named function without captures the kind follows from the definition in decision 15. Effects declared with `@modifies` are not part of conformance in v0.1. The names of the kinds (`new` or `trans`, and `into`) stay open; see the todo `2026-10-21-resolve-the-naming-of-method-declaration-kinds-new-vs-trans`.
20. **Result and parameter types (D2; decided 2026-10-22).** The result of an implementation must convert to the result of the contract member under the existing conversion rules: a narrower error set converts to a wider one, and an implementation that cannot fail is allowed. Parameters must match exactly in v0.1. An error-union result in a contract without an explicit set means any set. Variance of parameters and explicit error sets return with Open Question 1 of `ERROR_UNION.md` and with v0.2; the notation may change if [`ERROR_REPRESENTATION_UNIFICATION.md`](ERROR_REPRESENTATION_UNIFICATION.md) is accepted.
21. **`structural` contracts (D3; decided 2026-10-22; closes Open Question 9).** A `structural` contract applies only to types and carries member signatures only, without conditions. Conditions are applied independently, as a separate contract named with `obeys`. Conformance of a function is always explicit. Consequence: the wording of EDR-044 on structural satisfaction needs a clarification that conditions are never matched structurally.
22. **Violation and failing condition (D4; decided 2026-10-22).** (1) A violated condition is a `PROGRAM_ERROR` with a structural diagnostic: contract, clause, values of the names in it, and the guilty side (`requires`: the caller; `ensures`: the implementation). Termination is defined; there is no recovery in code. (2) A condition that cannot be evaluated (for example an out-of-range index inside `requires`) is a `PROGRAM_ERROR` with a separate code, and the fault lies with the contract author. (3) Conditions are evaluated in the order written and the first violated one is reported. (4) The class and the termination are fixed by the unifying EDR (Step 8). Tests derived from a contract count a violation as a failed case instead of terminating (`ORTHON-TEST.md`).
23. **`obeys` on a generic function (D5; decided 2026-10-22).** Allowed when the type parameters of the contract and of the function are linked by name and the bounds of the function are not stronger than the bounds of the contract. Conformance is checked once on the generic declaration and so holds for every instantiation; tests derived from a contract choose their own instantiations. A non-generic function may obey a contract with concrete type arguments (`Sort<Int>`). Using generic functions as values (generic bounds, `dyn`, collections) stays deferred (decision 15); Open Question 7 narrows to that.
24. **Short and long forms of a contract (D6; decided 2026-10-22).** A contract is written as a list of members (the general form). Members of a type contract are matched by name and signature. A contract with a single callable member also has a short form, `contract Name(params) -> result`, which is sugar: parentheses are mandatory, it has no receiver, its kind is always `fun`, and it is matched by call signature only, under any function name. A contract with no members (a marker such as `Send`) is allowed. A conditions-only header without parentheses (`contract Positive -> result: Int`) is an error in v0.1. A contract that attaches conditions to a position, whatever the member name, is deferred to v0.2 with its own distinct spelling; named predicates (`ensures is_positive(n)`) cover reuse in v0.1. The expansion of the short form into a single `call` member is kept as a definition that stays invisible to users; which parts of it remain in the v0.1 text is decided after the syntax is accepted (author, 2026-10-22).

## Open Questions

1. **Unique type per function — kept as a definition only (Review decision 15).** The premise stays as the definition that explains `obeys` on a function. How it relates to the structural function type (coercion or erasure through a callable bound), generics, and dispatch is **deferred to v0.2** with higher-order contracts; coupled to `FUNCTIONS.md` open question 4 and `ANONYMOUS_FUNCTION.md`. No longer a blocker for v0.1.
2. **Name of the construct.** `contract` (matches TRAITS' own wording; possible Solidity association) or keep `trait` (alternative A2)? A1 and A2 are semantically equivalent; the choice is **deferred until the syntax is accepted** (author, 2026-10-22). Until then `contract` is a provisional spelling and no document is renamed; the rename cascade is sized for A1 and is smaller for A2.
3. **Deferred to v0.2 (Review decision 17). Name of multi-instance contracts.** `law` is judged too heavy. Candidates: `property` (collides with the PROPERTIES concept), `rule` (matches the business-rule use but is generic), `check`, `claim`, `holds`, `axiom` (as heavy as `law`). Undecided.
4. **Closed (Review decision 17): multi-instance contracts are deferred to v0.2.** Needed for `Eq` and `Serialize`, but not blocking.
5. **Conformance keyword.** `obeys` is the candidate to replace `impl` (Review decisions, item 14). Word, position, multiple contracts, lambdas, and blanket conformance are tracked in [`../../../syntax/CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md); associated types remain open here.
6. **`old` (state at entry).** Result naming is settled (Review decisions, item 9) and `old` is deferred (item 10). The analysis and reintroduction triggers live in [`../deferrable/CONTRACT_OLD_STATE.md`](../deferrable/CONTRACT_OLD_STATE.md); nothing about `old` blocks this hypothesis.
7. **Deferred to v0.2 (Review decision 15). Generics and overloads.** A generic function is a family of anonymous types (one per instantiation); each overload has its own type. Consequences for `obeys` on generic functions are unexamined.
8. **Deferred to v0.2 (Review decision 15). `dyn C` on function values.** Cost and meaning of dynamic dispatch through a function contract.
9. **Closed (Review decision 21): `structural` contracts apply only to types and carry signatures only.** Conformance of a function is always explicit; conditions are never matched structurally.
10. **Closed (Review decision 18): `invariant` stays in v0.1 with a narrower check.** DOP treats data as immutable, so state invariants fall outside it; they remain for mutable types, checked after creation and after each `proc`, with the re-entrancy rule of decision 18.
11. **Enforcement by position.** Strictness at module boundaries versus internally, and partial disabling of checks. Deferred to the enforcement discussion.
12. **Deferred to v0.2 (Review decision 15). Lambda conformance.** Spelling of `obeys` on a lambda, and whether a lambda may be accepted into `dyn C` without declaring it.
13. **Deferred: external imposition of rules.** Form of the reference in a comment, how a separate module pulls the description by reference, a rule registry file, and imposing a contract on a foreign function (Racket `contract-in`), which shifts blame because the function itself promises nothing.
14. **Deferred to v0.2 with Open Question 3. Do not name the multi-instance kind "rule".** If every contract is a business rule, the word is too general for one kind among several. Note that `obeys` pairs naturally with *rule* and *law* as objects ("obeys the rule"); this is an input to the naming of the construct (Open Question 2) and of the multi-instance kind (Open Question 3), not a decision.
15. **Generation of values of a type** for contract-derived tests ([`ORTHON-TEST.md`](../../../tooling/ORTHON-TEST.md)): where the capability lives (a trait, a compiler service, constrained types). Not blocking v0.1; a question for the merged concept document.
16. **Retention of doc comments in the AST** for rule descriptions (Review decision 12): open until a doc-comment concept exists. A rule catalog works from predicate names alone in the meantime.

## Next steps (route through the pipeline)

1. **Done:** Decision Pipeline run (2026-10-19); Concept Design Review Steps 0–5 (2026-10-21), see the progress table.
2. **Done:** Step 6, Tooling Implications (2026-10-21): one new Tooling Requirement, `ORTHON-TEST.md`, and notes in `LANGUAGE_SERVER.md`, `ORTHON-DESCRIBE.md` and `ORTHON-PROVE.md`; see Step 6.
3. **Step 7, Convergence Check:** the syntax review and the probing of edge cases are done and the resulting decisions 19–24 are recorded (2026-10-22), see [Step 7](#step-7--convergence-check). **Remaining:** the author's sign-off after reading Steps 0–7. The words (`contract`, `obeys`) and the position candidates stay in [`CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md).
4. **Step 8, EDR-098** (Architecture): **drafted 2026-10-22 as Proposed** ([`EDR-098`](../../../decision_records/architecture/EDR-098-trait-contract-unification.md)); it carries the Step 2 tables verbatim and states its relation to EDR-019 (amended) and EDR-056 (amended). Remaining: the author's acceptance, the seven gates in `DECISION_LOG.md`. **Do not file a contract-object EDR before this hypothesis is decided:** it would be superseded immediately.
5. **Validation:** all seven gates with entries in `DECISION_LOG.md`; a measurement protocol for the LLM Generability Gate.
6. **If accepted:** the merged concept document in `what/concepts/` (8-section `_concept.md` template), registration in `CORE_CONCEPTS.md` and `INDEX.md`, the rename cascade over about 52 files as a tracked task, and the syntax pipeline for the words.

## Human Sign-off

> **Reviewed-by**: mniedre · **Date:** 22.10.2026 · **Verdict:** LOCKED
> Required before any EDR may be filed (Concept Design Review Step 7). Agents and GSD flows must not self-certify this field.

---

### Affected Documents

- [ ] `what/concepts/TRAITS.md` (EDR-019)
- [ ] `what/concepts/CONTRACTS.md` (EDR-056)
- [ ] `what/concepts/CONSTRAINED_TYPES.md` (EDR-080)
- [ ] `what/concepts/EQUALITY.md` (EDR-017)
- [ ] `what/concepts/STRUCTURAL_TYPING.md` (EDR-044)
- [ ] `what/GLOSSARY.md` (Contract umbrella term, Trait, Lambda)
- [x] `what/LANGUAGE_PROBLEMS.md` (entry added 2026-10-21: behavioral subtyping and separation of specification from implementation)
- [ ] the merged concept document replacing `what/concepts/TRAITS.md` and `what/concepts/CONTRACTS.md` (Review decision 16)
- [ ] `how/concepts/research/essential/FUNCTIONS.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/decision_records/INDEX.md`
- [ ] Other: the 52 files referencing EDR-019 / `TRAITS.md`

## See also

- [`TRAITS.md`](../../../../what/concepts/TRAITS.md), [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md), [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md)
- [`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md), [`SYNTAX_PIPELINE.md`](../../../SYNTAX_PIPELINE.md), [`concept-design-review.md`](../../../concept-design-review.md)
