# EDR-098: Trait and Function Contract Unification — One Specification Construct, Conformance for Types and Functions

**Status:** Proposed

**Date:** 2026-10-22

**Category:** Architecture

**Scope:** Subsystem (Type System, Contracts)

**Human Sign-off:** pending. The Concept Design Review sign-off recorded in the hypothesis (`Reviewed-by: mniedre · Date: 2026-10-22 · Verdict: LOCKED`, Step 7) permits this EDR to be filed. It is **not** an acceptance of this EDR. The field is left empty and must be set by the author; agents and GSD flows must not self-certify it.

**Amends:** [EDR-019](./EDR-019-traits.md) (Traits) and [EDR-056](./EDR-056-contracts.md) (Contracts, as already amended by [EDR-097](./EDR-097-contracts-result-named-old-deferred.md)). Neither is superseded as a whole; the relation is stated under Decision, item 9.

**Related:** [EDR-044](./EDR-044-structural-typing.md) (clarified, item 6), [EDR-080](./EDR-080-constrained-types.md) (the decomposition pattern reused), [EDR-081](./EDR-081-require-using-dependency-slots.md) (`require`, a syntax remark), [EDR-090](./EDR-090-data-oriented-programming.md) (specification separate from implementation), [EDR-089](./EDR-089-system-error-taxonomy.md) (`PROGRAM_ERROR`).

**Source hypothesis:** [`TRAIT_CONTRACT_UNIFICATION.md`](../../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md) (Concept Design Review Steps 0–7 complete; Review decisions 1–24).

> **Proposed 2026-10-22.** This record states what the hypothesis decided. It does
> not accept anything: the accepted concepts `TRAITS.md` and `CONTRACTS.md` are
> unchanged until the author signs this EDR
> ([`DECISION_PROCESS.md`](../../process/DECISION_PROCESS.md): no EDR without
> human sign-off). The spellings in the examples are **provisional**; the words
> and the position of the conformance statement belong to the syntax pipeline.

---

### Context

The specification treats one idea in two places.

1. **Traits** ([`TRAITS.md`](../../../what/concepts/TRAITS.md), EDR-019): a named
   set of operations a type promises to provide, with explicit conformance
   (`impl Trait for Type`), coherence, the orphan rule, and static or dynamic
   dispatch.
2. **Function contracts** ([`CONTRACTS.md`](../../../what/concepts/CONTRACTS.md),
   EDR-056): `requires` / `ensures` / `invariant` clauses written inside a
   function signature.

The review of 2026-10-18 accepted that contracts should become **named objects**
that a function refers to and that can be reused across implementations. Taken
alone, that adds a third mechanism with its own conformance check, its own Liskov
rules and its own coherence story, parallel to traits. Three costs follow:

- **Duplicate machinery.** "This element declares that it satisfies a named
  specification" would exist twice, with the same rules written twice (signature
  match, weaken `requires` and strengthen `ensures`, at most one declaration per
  pair, orphan rule).
- **Contracts on function arguments have no home.** Higher-order contracts were
  deferred in EDR-056 because nothing in the language can say "this argument is a
  function that satisfies this specification".
- **Properties of a whole trait have no home.** The Transitivity Invariant in
  [`EQUALITY.md`](../../../what/concepts/EQUALITY.md) quantifies over three values
  and cannot be a `requires` / `ensures` or an `invariant`; it lives in prose.

The pipeline requires a new construct to decompose into existing primitives
([`CONCEPT_PIPELINE.md`](../../CONCEPT_PIPELINE.md) Stage 3); a separate
contract-object construct has no such decomposition.

**Classical problem.** Behavioral subtyping and the separation of a specification
from its implementations (Liskov and Wing, 1994; Meyer, 1992). The entry was
added to [`LANGUAGE_PROBLEMS.md`](../../../what/LANGUAGE_PROBLEMS.md) on
2026-10-21. The citations are from general knowledge and are listed as not
verified in the hypothesis (Evidence).

**Process.** Decision Pipeline run 2026-10-19 (PASS for the unified form,
conditional on F2 as a definition only; F4 DEFER); Concept Design Review Steps
0–7 completed 2026-10-22; the author signed Step 7 (LOCKED) on 2026-10-22.

**Transparency architecture metric.** The accepted CONTRACTS concept has changed
once after acceptance (the deferral of `old`) and a second time by EDR-097; this
record would be a further change to CONTRACTS and the first to TRAITS. The
recommended handling is to treat the Concept Design Review of the source
hypothesis as the architectural review. This is the author's call and is to be
confirmed at sign-off of this EDR. None of the four rejected proposals
(EDR-075 to EDR-078) is similar to this one.

### Decision

The author's decisions in the Concept Design Review, grouped. The numbers refer to
the **Review decisions** of the source hypothesis.

1. **One construct for specifications (decisions 1, 6, 7, 14, 16).** A
   specification is a **named object**: a list of members (operations with
   signatures and optional conditions). A program element declares that it
   satisfies it. Type-level and function-level specifications are one construct;
   the kind is decided by the application site, not by the declaration. There is
   no rule "contracts of kind K apply only to subject S"; the only error is
   structural mismatch. Contracts are allowed on any function or method; the
   specification is separate from the implementation, as in DOP principle 4
   (EDR-090). If the unification is accepted, `TRAITS.md` and `CONTRACTS.md`
   become **one concept document**.
2. **Conformance applies to a second kind of subject (decisions 14, 15).** A
   named function declares conformance to a named contract with one callable
   member, under the rules already used for types: the signature matches; an
   implementation may weaken `requires` and strengthen `ensures`; at most one
   declaration per (element, contract) pair; the orphan rule; several contracts
   may be named and the conditions of all apply; dispatch is static by default and
   `dyn` is opt-in.
3. **F2 is a definition only (decision 15).** A function declaration is sugar for
   a unique anonymous type with one callable member. It explains conformance of a
   function and decomposes it into existing primitives (`pack` + `function` +
   conformance; Level 2 Language Pattern, the shape of EDR-080). It adds no
   surface for programmers, and no Level 0 or Level 1 construct. Its consequences
   for the structural function type, generics, `dyn` and lambdas are deferred to
   v0.2. The callable member has no receiver. How much of the expansion remains in
   the v0.1 text is decided after the syntax is accepted (decision 24).
4. **Conditions (decisions 2, 3, 5, 9, 10, 11, 12, 18).** Parameters are bound by
   name. `self` exists only for types. The result is named in the signature
   (EDR-097); `old` stays deferred (EDR-097). A violation report names the failed
   predicate; the description of a rule comes from the predicate name or its
   comment, and no `as` label is introduced. `invariant` stays in v0.1 with a
   narrower check: after creation and after each `proc` exits; not for `fun` or
   `new`; nested calls on the same object do not check on entry; for an immutable
   value the rule is a constrained type (EDR-080). `CONTRACTS.md`, which says
   "entry and exit of each public method", is amended together with the
   unification.
5. **Member kind, result types, violation, generics, forms (decisions 19–24).**
   - **Kind (19).** The declaration kind of an implementing method equals the kind
     of the contract member. The callable member of the short form is always the
     reading kind (`fun`). `@modifies` is not part of conformance in v0.1. The
     names of the kinds (`new` or `trans`, `into`) are a separate open question.
   - **Types (20).** The result of an implementation converts to the result of the
     contract member under the existing conversion rules (a narrower error set
     converts to a wider one; an implementation that cannot fail is allowed).
     Parameters match exactly in v0.1. An error-union result in a contract without
     an explicit set means any set.
   - **Structural (21).** `structural` applies only to types and carries member
     signatures only, without conditions; conditions are applied independently by
     a separate contract. Conformance of a function is always explicit.
   - **Violation (22).** A violated condition is a `PROGRAM_ERROR` with a
     structural diagnostic (contract, clause, values, guilty side: `requires` the
     caller, `ensures` the implementation); termination is defined and there is no
     recovery in code. A condition that cannot be evaluated is a `PROGRAM_ERROR`
     with a separate code and the fault lies with the contract author. Conditions
     are evaluated in the order written and the first violated one is reported.
     Contract-derived tests count a violation as a failed case.
   - **Generics (23).** `obeys` (provisional) on a generic function is allowed
     when the type parameters of the contract and the function are linked by name
     and the bounds of the function are not stronger than those of the contract;
     checked once on the generic declaration. A non-generic function may obey a
     contract with concrete type arguments.
   - **Forms (24).** The general form is a list of members, matched by name and
     signature. The short form `contract Name(params) -> result` is sugar for a
     contract with one callable member: parentheses are mandatory, no receiver,
     kind `fun`, matched by call signature under any function name. Marker
     contracts (no members) are allowed. A conditions-only header without
     parentheses is an error in v0.1; a contract that attaches conditions to a
     position is deferred to v0.2 with its own spelling.
6. **Clarification of EDR-044.** Structural satisfaction applies to types and to
   member signatures only; conditions are never matched structurally (item 5,
   decision 21). A wording clarification of EDR-044 is to be applied in the
   merged concept document.
7. **Deferred to v0.2 (decisions 13, 15, 17).** Using contracts through generics
   and function values (a function contract as a generic bound, `dyn` on function
   values, collections of contract-bearing functions, `obeys` on lambdas);
   multi-instance contracts (the properties over several values, for example the
   Transitivity Invariant of `Eq`); imposing a rule from outside on a foreign
   function and a rule registry file.
8. **Not decided here.** The words and positions, which go through the syntax
   pipeline ([`SYNTAX_PIPELINE.md`](../../SYNTAX_PIPELINE.md)):
   - `contract` versus keeping `trait` (alternatives A1 and A2). They are
     semantically equivalent; the choice is deferred until the syntax is accepted
     (author, 2026-10-22). Until then no document is renamed.
   - `obeys` versus `impl ... for` (`obeys` is the candidate to replace `impl`;
     adding it beside `impl` would give one concept two spellings).
   - The position of the conformance statement (candidates P1 separate
     statement, P2 both, P3 tail for functions and statement for types), tracked
     in [`CONFORMANCE_SYNTAX.md`](../../syntax/CONFORMANCE_SYNTAX.md).
   - The remark about `require` and `requires` differing by one letter in the same
     tail position, and the receiver spelling in type contracts.
9. **Relation to EDR-019 and EDR-056.** This EDR **amends** both and supersedes
   neither as a whole. The semantics of EDR-019 (nominal traits, explicit
   conformance, coherence, orphan rule, dispatch) and of EDR-056 (`requires`,
   `ensures`, `invariant`, purity, the Contract Enforcement Policy) are carried
   over. What changes: they are described as one concept; conformance may be
   declared by a named function; the keyword and conformance syntax of EDR-019
   (`trait`, `impl ... for`) are superseded **in part**, once the syntax pipeline
   fixes the words; `invariant` is checked as in item 4. If the author prefers a
   formal supersession, that is a change to this item, not to the rest.

```orthon
// Provisional spelling throughout.

contract Sqrt(x: Float) -> result: Float        // short form: one callable member
    requires x >= 0.0
    ensures result >= 0.0

fun sqrt_newton(x: Float) -> Float obeys Sqrt   // a named function obeys a contract
    ...

contract Stack<T>                               // list of members: type-level
    proc push(item: T)
        requires self.size < self.capacity
        ensures self.top() == item
    proc pop() -> item: T
        requires not self.is_empty()

contract Positive -> result: Int                // ERROR in v0.1: conditions-only header
Float obeys Sqrt                                // ERROR: Float has no callable member call(x: Float) -> Float
fun g(x: Float) -> Float obeys Sqrt
    requires x > 10.0                           // ERROR: stronger requires than the contract
```

### Consequences

- **Positive:**
  - One conformance mechanism, one set of Liskov and coherence rules, for types
    and named functions; no third mechanism beside traits and inline clauses.
  - A specification is a named, reusable object: many implementations, tests
    derived once, and a place for business rules in domain vocabulary.
  - No new primitive and no new surface for programmers beyond the conformance
    statement; the function-level part is a Level 2 pattern.
  - A path to higher-order contracts and to multi-instance properties in v0.2
    without a new construct.
  - Violations, failing conditions, member kinds, result types, generics and
    `structural` all have a stated rule.
- **Negative:**
  - A rename cascade over accepted documents: about 52 files reference EDR-019 or
    `TRAITS.md` and 35 `impl ... for` lines in 17 files (counts as of 2026-10-18,
    sized for the largest option); smaller if `trait` is kept.
  - Users of `trait` and `impl` meet a rename; "a function has a type" is new,
    although hidden in v0.1 (POLA caution).
  - The short and the long form of a contract are two spellings of one thing
    (Syntax Principle 1), accepted as explicit sugar.
  - The keyword `contract` may evoke Solidity for LLMs (an assumption to test, not
    a finding); the LLM Generability Gate is not measured.
  - The accepted concept documents are inconsistent with this record until the
    merged concept document replaces them.
  - Several consequences are deferred (function values in generics, `dyn`,
    lambdas, multi-instance contracts), so the v0.1 benefit is the shape check,
    contract-derived tests and documentation for functions, not yet higher-order
    contracts.

### Compliance

Checks and follow-up work, none of which is done by this record:

1. **Sign-off.** The author's acceptance is required; the Status stays Proposed
   until then.
2. **Merged concept document** in `what/concepts/` replacing
   [`TRAITS.md`](../../../what/concepts/TRAITS.md) and
   [`CONTRACTS.md`](../../../what/concepts/CONTRACTS.md), written from the
   8-section [`_concept.md`](../../templates/_concept.md) template, with the
   invariant rule of item 4 and the clarification of EDR-044.
3. [`what/CORE_CONCEPTS.md`](../../../what/CORE_CONCEPTS.md),
   [`what/GLOSSARY.md`](../../../what/GLOSSARY.md) (Contract umbrella term, Trait,
   Lambda) and [`INDEX.md`](../INDEX.md) updated on acceptance.
4. **Gate validation** (see below): all seven gates with entries in
   [`DECISION_LOG.md`](../../gates/DECISION_LOG.md); none is written yet.
5. **LLM Generability measurement protocol**: a fixed task set over the contract
   forms (the gate's own 90 percent threshold), recording failures from habitual
   `trait` / `impl`; deferred until after sign-off, as for EDR-097 (R7).
6. **Syntax pipeline** for the words and positions of item 8, through
   [`CONFORMANCE_SYNTAX.md`](../../syntax/CONFORMANCE_SYNTAX.md) and the coupled
   entries in [`how/syntax/README.md`](../../syntax/README.md).
7. **Rename cascade** over about 52 files as a tracked task after the syntax is
   accepted.
8. [`FUNCTIONS.md`](../../concepts/research/essential/FUNCTIONS.md) review points
   to F2 as the v0.1 answer to its open question 4 (function-type notation stays
   undecided).
9. **Open follow-ups, not decided here:** the naming of method declaration kinds
   (todo of 2026-10-21), the receiver spelling in type contracts, the notation of
   error results if
   [`ERROR_REPRESENTATION_UNIFICATION.md`](../../concepts/research/essential/ERROR_REPRESENTATION_UNIFICATION.md)
   is accepted, generation of values for contract-derived tests (Open Question 15
   of the hypothesis).
10. **Check on acceptance:** no accepted document describes a trait and a function
    contract as separate conformance mechanisms.

### Alternatives Considered

The tables below are carried verbatim from Step 2 of the source hypothesis.

Candidates in the canonical set of [`concept-design-review.md`](../../concept-design-review.md): a language construct, syntactic sugar, a library or composition, do nothing, and at most one wild alternative. The IDs A0–A4 are kept from earlier sections; A5 is the wild alternative.

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

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| A0 separate contract-object construct beside `trait` / `impl` | Exits at pipeline Q3: it decomposes into existing primitives, so it is not a new language feature; two conformance mechanisms |
| A3 pattern over `trait` and `Callable` with today's `impl` | Fallback only if F2 is rejected; needs nameable unique function types and is verbose per implementation |
| A4 inline clauses only | The reject baseline: it leaves the problems of Context open (no reuse, no specification-first workflow, no home for higher-order contracts) |
| A5 contracts bound from outside by name (registry) | Action at a distance; drift between the registry and the declaration; fails Explicitness |
| A1 versus A2 | Not rejected; equivalent in semantics, decided by the syntax pipeline |

### Gate Validation

**Not run yet.** The Decision Pipeline was run on 2026-10-19 on the hypothesis
(provenance for the Concept Design Review, not a gate record). The seven gates
are to be run and recorded in `DECISION_LOG.md` before this EDR is accepted; no
verdict is stated here, and none is borrowed from EDR-097.

| Gate | Planned method | Status |
|------|----------------|--------|
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | pending |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method (TRIZ for the contradiction of one construct versus two subjects) | pending |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis (layers, the Level 2 classification, no new primitive) | pending |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Logical Analysis | pending |
| `USER_VALUE_GATE` | Working Backwards Method | pending |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein Method | pending |
| `LLM_GENERABILITY_GATE` | Empirical Analysis (measurement protocol of Compliance item 5) | pending |

Inputs already prepared: the Step 4 principle check of the hypothesis (no
violation of a principle; two cautions, Transparency and POLA; no Tier 1 EDR is
required) and the Step 0 primitive decomposition (no new primitive; Level 2 for
the function-level part).

---

> **Human Sign-off:** pending. Required before this EDR is Accepted. Agents and
> GSD flows must not self-certify this field.
