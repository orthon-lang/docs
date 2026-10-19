# Contract Unification (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the 2026-10-18 review of the CONTRACTS concept. Proposes that
> `trait` (type-level contract) and the function contract of EDR-056
> (method-level contract) are one construct applied to different subjects.
> Home: `how/concepts/research/essential/` — semantic hypothesis (essential
> tier). Its syntax questions are registered by pointer in the
> `how/syntax/` decision queue (EDR-087).
>
> **Last updated:** 2026-10-18
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

## Problem

The specification already treats one idea in two places:

1. **Traits.** [`TRAITS.md`](../../../../what/concepts/TRAITS.md) defines a trait as "a behavioural contract that types can implement": a named set of operations a type promises to provide, with explicit conformance (`impl Trait for Type`), coherence, and static or dynamic dispatch.
2. **Function contracts.** [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) (EDR-056) defines `requires` / `ensures` / `invariant` as clauses written inside a function signature.

The 2026-10-18 review accepted that contracts should become **named objects** that a function refers to (`obeys`), separate from the function body, with reuse across many implementations. Taken alone, that adds a third mechanism with its own conformance check, its own Liskov rules, and its own coherence story, parallel to traits. Three concrete costs follow:

- **Duplicate machinery.** "This element declares that it satisfies a named specification" would exist twice, with the same rules written twice (signature match, weaken-pre / strengthen-post, at most one declaration per pair, orphan rule).
- **Contracts on function arguments have no home.** Higher-order contracts are deferred to v0.2 (EDR-056, open question 1) because nothing in the language can say "this argument is a function that satisfies this specification".
- **Properties of a whole trait have no home.** The Transitivity Invariant in [`EQUALITY.md`](../../../../what/concepts/EQUALITY.md) quantifies over three values of one type. It cannot be a `requires` / `ensures` (one call) or an `invariant` (one state), so it lives in prose.

The project's own discipline ([`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md) Stage 3) also requires a new construct to decompose into existing primitives. A separate contract-object construct currently has no such decomposition.

## Hypothesis

**One construct — provisionally `contract` — names a set of operations with optional conditions. A program element declares conformance with one keyword — provisionally `obeys`. The kind of contract (type-level or method-level) is not a property of the declaration; it is determined by the application site.**

Supporting premise: **a function is syntactic sugar for a unique anonymous type with one callable member.** Then a contract on a function is simply a contract with one callable member applied to that type, and a trait is the multi-member case.

Consequences claimed by the hypothesis (to be tested, not assumed):

1. One conformance mechanism, one set of Liskov and coherence rules, for types and functions.
2. Contracts on function arguments are ordinary generic bounds (`<Sqrt as F>`), so higher-order contracts need no new feature.
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
    fun call(self, x: Float) -> Float ...
conformance Sqrt for sqrt_newton_T                    // what `obeys` expands to
let sqrt_newton = sqrt_newton_T{}
```

The mechanism already exists in the specification: [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md) desugars `type Age = Int requires ...` into a `struct` plus `impl Callable(Int) -> Age for Age`, and closures already capture explicitly. The anonymous type is not nameable by the programmer; `obeys` is the only surface that mentions it.

### Inline clauses and the short form

- Clauses written directly in a function signature (`requires` / `ensures`) are an **anonymous contract** satisfied only by that function. They are the common case and stay valid unchanged.
- `contract Name(params) -> result` is sugar for a contract with one callable member `call`.
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
| Multi-instance contract | several values or calls | property quantified over inputs | declared in a contract | property-based generation, debug |

Adjacent, not part of this hypothesis: module-level effects and dependencies ([`EFFECT_FOOTPRINT.md`](EFFECT_FOOTPRINT.md), hypothesis; [`../MODULE_HEADER_FILE.md`](../MODULE_HEADER_FILE.md), draft) and mutation scope ([`../deferrable/FRAME_CONDITIONS.md`](../deferrable/FRAME_CONDITIONS.md), draft).

### Multi-instance contracts (provisional name: *law*)

A condition that quantifies over several values or calls: transitivity of `Eq` over three values, the round trip `from_json(to_json(x)) == x` for `Serialize` / `Deserialize`. It is a contract with no subject function and with free inputs:

```orthon
contract Eq
    fun ==(self, other: Self) -> Bool

    law transitive(a: Self, b: Self, c: Self)          // keyword provisional
        requires a == b and b == c
        ensures a == c
```

It is not checked at a call boundary. It is checked by generating inputs (property-based testing) or by debug-mode assertions, as EQUALITY already allows ("the compiler may insert runtime checks in debug mode"). The name `law` is considered too heavy; candidates are listed in Open Question 3.

## Examples by usage context

```orthon
// 1. Type-level conformance (was: impl Printable for User)
User obeys Printable
    fun format(self) -> String
        return "User({self.name})"

// 2. Function-level conformance, many implementations of one specification
fun sqrt_newton(x: Float) -> Float obeys Sqrt
    ...
fun sqrt_table(x: Float) -> Float obeys Sqrt
    ...

// 3. Methods of a contract carry conditions; implementations inherit them
//    (mutating methods are `proc`; relations to the state before the call
//    need `old`, which is deferred: see CONTRACT_OLD_STATE.md)
contract Stack<T>
    proc push(item: T)
        requires self.size < self.capacity
        ensures self.top() == item
    proc pop() -> item: T
        requires not self.is_empty()

// 4. Generic bound (unchanged form)
fun print_all<Printable as T>([T] items)

// 5. Dynamic dispatch on a function value
fun apply_dyn(f: dyn Sqrt, x: Float) -> Float
    return f(x)

// 6. Higher-order contract: a bound on a function-valued argument
fun map_roots<Sqrt as F>(f: F, xs: [Float]) -> [Float]

// 7. Lambda that declares conformance (spelling provisional)
let root = fn(x) -> newton(x) obeys Sqrt

// 8. Inline anonymous contract (unchanged from EDR-056)
fun withdraw(balance: Int, amount: Int) -> result: Int
    requires amount > 0
    requires amount <= balance
    ensures result == balance - amount

// 9. Several contracts
fun sqrt_safe(x: Float) -> Float obeys Sqrt, Logged
    ...
```

**Error cases** (all reduce to structural mismatch or an inherited rule):

```orthon
Float obeys Sqrt                             // ERROR: Float has no callable member call(x: Float) -> Float
fun f(x: Int) -> Int obeys Printable         // ERROR: function has no member format(self)
fun g(x: Float) -> Float obeys Sqrt
    requires x > 10.0                        // ERROR: stronger requires than the contract
fun h(x: Float) -> Float obeys Sqrt
    ensures root >= 1.0                      // allowed: stronger ensures
User obeys Printable                         // ERROR if format is missing or its signature differs
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
| Rule over several values | multi-instance contract (name still open, see Open Question 14) |
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

## Alternatives

| ID | Alternative | Notes |
|---|---|---|
| A0 | Status quo plus a separate contract-object construct: `trait` / `impl` stay, new `contract` / `obeys` for functions | two conformance mechanisms; no unification cost; the duplication this hypothesis targets |
| A1 | **This hypothesis**: `contract` replaces `trait`, `obeys` replaces `impl` | one mechanism; largest cascade |
| A2 | Keep the name `trait` as the single construct; `obeys` is the application keyword for types and functions | smaller cascade, no rename of EDR-019; `trait Sqrt(x: Float) -> Float` reads oddly for a function specification |
| A3 | Pattern only: express function contracts as a trait with a `Callable` member using today's `impl` | no new keywords; needs "functions have unique types" anyway; verbose per implementation |
| A4 | No contract objects: inline clauses only (EDR-056 as accepted) | loses reuse, higher-order contracts, and the specification-first workflow |

| Criterion | A0 | A1 | A2 | A3 | A4 |
|---|---|---|---|---|---|
| New primitive constructs | 1 | 0 | 0 | 0 | 0 |
| New keywords | 2 | 2 (and 2 removed) | 1 (and 1 removed) | 0 | 0 |
| Reuse of one specification across implementations | yes | yes | yes | yes | no |
| Higher-order contracts without a new feature | no | yes | yes | yes | no |
| "Wrong kind" errors | yes | no | no | no | n/a |
| Depends on "functions have unique types" | no | yes | yes | yes | no |
| Cascade over accepted documents | small | large | medium | small | none |
| LLM prior for the keywords | mixed | uncertain (`contract`, `obeys`) | mixed (`trait` kept, `obeys` new) | strong (`trait`, `impl`) | strong |

## Pre-filter notes (pipeline Stages 2–4)

- **Decision pipeline.** Language-level, not library: conformance and checking are compiler services. Not pure sugar for the function-level case only, because it changes the type story (see Open Question 1).
- **Primitive decomposition.** Function contract = `struct` + `Callable` + conformance, the composition used by constrained types. The multi-member case is today's trait.
- **Layer classification.** Candidate **Level 2 (Language Pattern)** for the function-level case, with the composition formula above. Whether "every function has a unique nominal type" is a Level 0 / Level 1 decision is part of the [`FUNCTIONS.md`](FUNCTIONS.md) review.

## Principle alignment (preview — gate verdicts are not recorded here)

| Principle | Reading |
|---|---|
| Orthogonality | one construct applied to different subjects instead of two constructs with overlapping rules |
| Explicitness | conformance is declared (`obeys`); nothing is satisfied silently except under the existing `structural` opt-in |
| Composition over addition | the function-level case decomposes into existing primitives |
| One concept → one syntax | the specification-and-conformance concept gets one spelling |
| Named before symbolic | `obeys` is a word; no new symbol is introduced |
| DOP principle 4 | the specification is separate from the implementation and applied at boundaries |
| LLM generability | risk: new keywords against strong `trait` / `impl` priors; to be measured in the LLM Generability Gate |

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

## Open Questions

1. **Unique type per function.** Does every function have its own nominal type, and how does it relate to the structural function type `Fn(A) -> B` (coercion or erasure through a callable bound)? Coupled to `FUNCTIONS.md` open question 4 and `ANONYMOUS_FUNCTION.md`. This is the main dependency.
2. **Name of the construct.** `contract` (matches TRAITS' own wording; possible Solidity association) or keep `trait` (alternative A2)?
3. **Name of multi-instance contracts.** `law` is judged too heavy. Candidates: `property` (collides with the PROPERTIES concept), `rule` (matches the business-rule use but is generic), `check`, `claim`, `holds`, `axiom` (as heavy as `law`). Undecided.
4. **Multi-instance contracts in v0.1 or later?** Needed for `Eq` and `Serialize`, but not blocking.
5. **Conformance keyword.** `obeys` is the candidate to replace `impl` (Review decisions, item 14). Word, position, multiple contracts, lambdas, and blanket conformance are tracked in [`../../../syntax/CONFORMANCE_SYNTAX.md`](../../../syntax/CONFORMANCE_SYNTAX.md); associated types remain open here.
6. **`old` (state at entry).** Result naming is settled (Review decisions, item 9) and `old` is deferred (item 10). The analysis and reintroduction triggers live in [`../deferrable/CONTRACT_OLD_STATE.md`](../deferrable/CONTRACT_OLD_STATE.md); nothing about `old` blocks this hypothesis.
7. **Generics and overloads.** A generic function is a family of anonymous types (one per instantiation); each overload has its own type. Consequences for `obeys` on generic functions are unexamined.
8. **`dyn C` on function values.** Cost and meaning of dynamic dispatch through a function contract.
9. **`structural` contracts.** What implicit (structural) satisfaction means for a function.
10. **Invariant for mutable types.** DOP treats data as immutable, so state invariants fall outside it. Keep `invariant` in v0.1 or reduce it to constrained types? Also the re-entrancy problem (see the review note).
11. **Enforcement by position.** Strictness at module boundaries versus internally, and partial disabling of checks. Deferred to the enforcement discussion.
12. **Lambda conformance.** Spelling of `obeys` on a lambda, and whether a lambda may be accepted into `dyn C` without declaring it.
13. **Deferred: external imposition of rules.** Form of the reference in a comment, how a separate module pulls the description by reference, a rule registry file, and imposing a contract on a foreign function (Racket `contract-in`), which shifts blame because the function itself promises nothing.
14. **Do not name the multi-instance kind "rule".** If every contract is a business rule, the word is too general for one kind among several. Note that `obeys` pairs naturally with *rule* and *law* as objects ("obeys the rule"); this is an input to the naming of the construct (Open Question 2) and of the multi-instance kind (Open Question 3), not a decision.

## Next steps (route through the pipeline)

1. Review this hypothesis in the Concept Design Review; the unique-type question (Open Question 1) must be settled together with the `FUNCTIONS.md` review.
2. Run the naming and conformance-keyword questions through [`SYNTAX_PIPELINE.md`](../../../SYNTAX_PIPELINE.md); the coupling is registered in the [`how/syntax/`](../../../syntax/README.md) decision queue.
3. **Do not file a contract-object EDR before this hypothesis is decided.** If A1 or A2 wins, the contract-object EDR would be superseded immediately.
4. If accepted: an EDR superseding the keyword and conformance syntax of EDR-019 and amending EDR-056; the cascade over about 52 files becomes a tracked task.

## Human Sign-off

> **Reviewed-by:** — · **Date:** — · **Verdict:** pending
> Required before any EDR may be filed (Concept Design Review Step 7). Agents and GSD flows must not self-certify this field.

---

### Affected Documents

- [ ] `what/concepts/TRAITS.md` (EDR-019)
- [ ] `what/concepts/CONTRACTS.md` (EDR-056)
- [ ] `what/concepts/CONSTRAINED_TYPES.md` (EDR-080)
- [ ] `what/concepts/EQUALITY.md` (EDR-017)
- [ ] `what/concepts/STRUCTURAL_TYPING.md` (EDR-044)
- [ ] `what/GLOSSARY.md` (Contract umbrella term, Trait, Lambda)
- [ ] `how/concepts/research/essential/FUNCTIONS.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/decision_records/INDEX.md`
- [ ] Other: the 52 files referencing EDR-019 / `TRAITS.md`

## See also

- [`TRAITS.md`](../../../../what/concepts/TRAITS.md), [`CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md), [`CONSTRAINED_TYPES.md`](../../../../what/concepts/CONSTRAINED_TYPES.md)
- [`CONCEPT_PIPELINE.md`](../../../CONCEPT_PIPELINE.md), [`SYNTAX_PIPELINE.md`](../../../SYNTAX_PIPELINE.md), [`concept-design-review.md`](../../../concept-design-review.md)
