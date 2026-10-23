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
   unification. Only types declare invariants in v0.1: module invariants were
   removed on 2026-10-23 (Tier 4, inline, in `CONTRACTS.md`), so the words "type
   or module" in Review decision 18 of the hypothesis read as "type".
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
     a separate contract. Conformance of a function is always explicit. **The
     `structural` mode itself is deferred out of v0.1 (author, 2026-10-23; Tier 4
     in `STRUCTURAL_TYPING.md`) and returns in v0.3 with the module boundary;
     this rule applies when it returns.**
   - **Violation (22).** When a check is enabled by the Contract Enforcement
     Policy, a violated condition is a `PROGRAM_ERROR` with a structural
     diagnostic (contract, clause, values, guilty side: `requires` the caller,
     `ensures` the implementation); termination is defined and there is no
     recovery in code. Under elision (release builds without `--enforce-contracts`)
     a violated contract is not detected and is an undetected program error, as in
     EDR-056 (R9). A condition that cannot be evaluated, when its check runs, is a
     `PROGRAM_ERROR` with a separate code and the fault lies with the contract
     author. Conditions
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
   merged concept document; it takes effect when the structural mode returns in
   v0.3.
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
10. **Clarifications from the logical-consistency resolutions R1–R4
    (2026-10-22; proposed text, awaiting sign-off).** They sharpen the
    hypothesis without contradicting it; the hypothesis is signed and is not
    edited, so where wording differs, this item governs.
    - **Umbrella term (R1).** "Contract" is the umbrella term for a named
      specification of any subject. The accepted entries Contract (Design by
      Contract) and Trait stay as the two kinds until this EDR is accepted; the
      proposed entry is in [`GLOSSARY.md`](../../../what/GLOSSARY.md) and is marked as
      not accepted.
    - **Base case of F2 (R2).** The callable member of the anonymous type is the
      `function` primitive of a type (a method), not a free-function declaration.
      The sugar of item 3 applies only to free-function declarations, so the
      definition ends after one step and has no regress.
    - **Matching rule by subject (R3).** For a **type**, each member of the
      contract is matched by name and signature; a type obeys a short-form contract
      by providing a callable member `call`. For a **function**, the contract must
      have exactly one member, a callable member named `call` (the short form
      produces it), matched by call signature (parameters and result); the name of
      the function is irrelevant. A contract with another member name, or with more
      than one member, cannot be obeyed by a function: this is an error with a hint
      to use the short form or to name the member `call`.
    - **Overloads and associated types (R4).** `obeys` on a function binds the
      declaration it belongs to: each overload obeys separately, and an overload
      set as a whole does not obey. A callable member has no associated types in
      v0.1; type contracts keep them (Open Question 5 of the hypothesis stays
      open for types).

    ```orthon
    // provisional spelling
    contract Sqrt(x: Float) -> result: Float
    fun root(x: Float) -> Float obeys Sqrt          // ok: the function name is irrelevant (R3)

    contract Calc
        fun calc(x: Float) -> result: Float
    fun root(x: Float) -> Float obeys Calc          // ERROR (R3): the member is `calc`; use the short form or name it `call`

    contract Pair
        fun first(self) -> Int
        fun second(self) -> Int
    fun make() -> Int obeys Pair                    // ERROR (R3): a function obeys a contract with exactly one member

    type Squarer(k: Float)
    Squarer obeys Sqrt                              // ok if Squarer provides `fun call(x: Float) -> Float` (R3)

    fun scale(x: Int) -> Int obeys Scaler           // (R4) binds this overload only
    fun scale(x: Float) -> Float                    // does not obey Scaler
    ```

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

Selected examples for decisions 19–24. The full catalog, with all contexts and
error cases, is in the hypothesis:
[Examples for Review decisions 19–24](../../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md#examples-for-review-decisions-1924).

```orthon
Frozen obeys Counter
    fun increment() ...                         // ERROR (19): the member is proc, the implementation is fun

FileConfig obeys Config
    fun load(path: String) -> !Config           // ERROR (20): the contract does not allow failure

structural contract Bounded
    fun len(self) -> result: Int
        ensures result >= 0                     // ERROR (21), when structural returns in v0.3: signatures only

withdraw(100, -5)                               // (22) PROGRAM_ERROR, requires false, the caller is at fault

fun hashsort<Ordered + Hash as T>(items: [T]) -> [T] obeys Sort<T>   // ERROR (23): stronger bound

contract Sqrt
    fun calc(self, x: Float) -> result: Float   // (24) matched by name `calc`
contract Root(x: Float) -> result: Float        // (24) short form: matched by call signature, any function name
fun calc(self, x: Float) -> Float obeys Positive   // ERROR (24): Positive describes a member `call(self) -> Bool`
```

11. **Clarifications from the simplicity and architecture resolutions R5–R8
    (2026-10-22; proposed text, awaiting sign-off).**
    - **Level 2 pattern and why the sugar is kept (R5).** Function-level
      conformance is a Level 2 Language Pattern: `pack{captures}` + `function(call)`
      + conformance. In today's language the same result is a hand-written `struct`
      with `impl Callable` and a conformance statement, the shape of EDR-080
      (alternative A3, which stays the fallback if F2 is rejected). The sugar is
      kept because A3 needs, for every implementation of a specification, a
      nameable wrapper type, its own `impl` block and a construction of the value;
      a specification with ten implementations costs ten wrappers. The sugar
      removes the wrapper and leaves the semantics identical.
    - **Separately reviewable rule (R6).** The `invariant` check rule of item 4
      (after creation and after each `proc` exit; not for `fun` or `new`;
      re-entry rule) amends CONTRACTS only and no other item of this record depends
      on it. It is recorded as its own entry in the Decision History of the merged
      concept so that it can be reviewed apart. This EDR does not split it; the
      author may move it into a separate EDR at sign-off.
    - **Deferrals and `structural` (R7).** The deferrals of item 7 are kept in a
      register with the reason and the trigger for reopening each:
      [`CONTRACT_DEFERRALS_V02.md`](../../concepts/research/deferrable/CONTRACT_DEFERRALS_V02.md).
      The rule that `structural` contracts carry signatures only stays in item 5
      (decision 21), with the EDR-044 clarification of item 6.
    - **Interactions (R8).** The per-concept interactions of the unified construct
      are drafted in
      [`notes/trait-contract-unification-interactions.md`](../../../notes/trait-contract-unification-interactions.md)
      as input to the Cross-Cutting Review (Stage 11, Phase 6). The formal
      pair-wise matrix in `what/CROSS_CUTTING.md` is still to be made; rows marked
      undefined there are examined before final acceptance.

12. **Clarifications from the implementation-independence and maintainability
    resolutions R9–R12 (2026-10-22; proposed text, awaiting sign-off).**
    - **Violation under elision (R9).** Item 5, "Violation (22)", is qualified in
      place: termination is defined when the check is enabled; under elision a
      violated contract is an undetected program error.
    - **Strategies (R10).** The strategy documents carry a note on contracts,
      marked as proposed: no strategy gets a new default, the rule of `CONTRACTS.md`
      applies to all, and the embedded strategy uses a minimal diagnostic payload
      (diagnostic code and source location). See
      [`IMPLEMENTATION_STRATEGIES.md`](../../strategies/IMPLEMENTATION_STRATEGIES.md)
      § Contracts Across Strategies.
    - **Precedents and F2 (R11).** The cited precedents were checked against
      primary documentation; the result is in Gate Validation, "Verification of
      precedents". The plan for surfacing F2 in v0.2 is in
      [`CONTRACT_DEFERRALS_V02.md`](../../concepts/research/deferrable/CONTRACT_DEFERRALS_V02.md).
    - **Words and rename cascade (R12).** The choice between `contract` and
      `trait` and between `obeys` and `impl ... for` is routed through the syntax
      queue ([`how/syntax/README.md`](../../syntax/README.md): the conformance row
      and the contract-keyword row). The cascade is sized for both options in Gate
      Validation, "Rename cascade sizing".

13. **Clarifications from the LLM-generability resolutions R13–R15
    (2026-10-22; proposed text, awaiting sign-off).**
    - **Measurement (R13).** The measurement protocol is written, in
      [`notes/trait-contract-unification-llm-measurement.md`](../../../notes/trait-contract-unification-llm-measurement.md):
      12 tasks, at least two models, 10 samples per task, a control run in the
      current form, an error taxonomy and the 90 percent threshold. **The gate
      measurement has not been run.** A small pilot of the protocol was made on
      2026-10-22 (10 runs: contract variants 88/96, control 14/14, one root cause,
      task T6, a gap in the schema; a pilot is not a gate measurement and changes no
      verdict). It found nine defects of the protocol, which were fixed on
      2026-10-23 (the task set stays at 12); see the pilot and the list of defects
      in the notes file.
    - **Blocking items of the syntax pipeline (R14).** The position of the
      conformance statement (P1, P2, P3) and the receiver spelling in members of a
      type contract are registered as open questions 2 and 8 of
      [`CONFORMANCE_SYNTAX.md`](../../syntax/CONFORMANCE_SYNTAX.md) and in its row of
      the syntax queue. They block the final grammar and the measurement of a single
      variant; they do not block the semantic decisions of this record. Nothing is
      decided here.
    - **Diagnostics (R15).** The following errors are compile errors with a
      dedicated message and a hint. The wording is provisional; the structure
      (message, then hint) is the requirement.

      | Error | Message | Hint |
      |-------|---------|------|
      | Conditions-only header, `contract Positive -> result: Int` | a contract header needs a parameter list; a contract with conditions only is not part of v0.1 | write the parameter list (`contract Positive(n: Int) -> result: Int`), or put the condition in a named predicate and call it in `requires` / `ensures` |
      | `self` in the short form, `contract Area(self) -> result: Float` | the short form of a contract has no receiver | describe a method with the member-list form: `contract Shape` with `fun area(self) -> result: Float` |
      | A function obeys a contract whose single member is not named `call`, `fun root(...) obeys Calc` with the member `calc` | a function can obey only a contract whose single member is named `call`; `Calc` has a member named `calc` | use the short form (`contract Calc(x: Float) -> result: Float`) or name the member `call` |
      | A function obeys a contract with several members, `fun make() obeys Pair` | a function can obey only a contract with exactly one member; `Pair` has 2 | split the contract, or make a type obey it |

      The diagnostics for the other errors of items 5 and 10 (member kind, result
      type, `structural` with conditions, stronger bound, violation report) are
      specified in structure by the rules of those items; their wording is written
      with the merged concept.

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
  - A rename cascade over documents: 56 files under `what/` and `how/` reference
    EDR-019 or `TRAITS.md`, 41 `impl ... for` lines in 20 files and 48 `trait`
    declaration lines in 24 files appear in code blocks (counts as of 2026-10-22,
    including the unification's own documents); smaller if `trait` is kept. See
    Gate Validation, "Rename cascade sizing".
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
   Lambda) and [`INDEX.md`](../INDEX.md) updated on acceptance. The umbrella
   entry is already in the glossary as **proposed** (R1); the accepted entries are
   unchanged until acceptance.
4. **Gate validation** (see below): all seven gates were run on 2026-10-22 and
   recorded in [`DECISION_LOG.md`](../../gates/DECISION_LOG.md) (one Pass, six
   Flags, no Fail). The resolutions R1–R15 are proposed and await the author's
   decision; R1–R8 are applied as proposed text; the Flags are to be resolved
   before acceptance.
5. **LLM Generability measurement**: the protocol is written (R13,
   [`notes/trait-contract-unification-llm-measurement.md`](../../../notes/trait-contract-unification-llm-measurement.md)):
   a fixed task set over the contract forms, the gate's own 90 percent threshold,
   an error taxonomy. **The run is not done** and is deferred until the open
   variants of the syntax pipeline are decided, or is run as a comparison of
   them, and is recorded before the final gate record, as for EDR-097 (R7).
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

All seven gates apply: the decision adds, changes and removes language rules
(`DECISION_VALIDATION.md` § Gate Selection, "New language construct"). The
Decision Pipeline was run on 2026-10-19 on the hypothesis and is not re-run. The
verdicts below come from the run of 2026-10-22 made by the drafting agent
(analysis only, no LLM sampling). They are **proposed** verdicts: the author
confirms or overrides them at sign-off of this EDR. Gate preconditions
(`DECISION_VALIDATION.md` § Gate Preconditions): the Problem statement and the
classical problem exist (Step 1 of the hypothesis); the interactions with existing
concepts are documented in the hypothesis (Impact and cascade, decisions 19–24,
Step 7 edge cases), but the pair-wise interaction matrix of the Cross-Cutting
Review (`CONCEPT_PIPELINE.md` Stage 11, Phase 6) has not been made, so that
precondition is met in part (R8).

| Gate | Method | Verdict | Summary |
|------|--------|---------|---------|
| `USER_VALUE_GATE` | Working Backwards | **Pass** | The story is stated in user terms and maps to a classical problem; the payoff of the function-level part in v0.1 is narrower than the full vision (observation, not a flag) |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | **Flag** | The core rules hold; four resolvable gaps (R1–R4): the umbrella use of "contract" against the current glossary, F2 read literally is circular, the matching rule depends on the subject, associated types and overloads on `obeys` are unexamined. R1–R4 applied as proposed text on 2026-10-22 |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | **Flag** | Fewer mechanisms than the alternatives, but the function-level part is expressible by composition (it is a Level 2 pattern by design), and the record bundles a change to `invariant` checking (R5, R6). R5 and R6 applied as proposed text on 2026-10-22 |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | **Flag** | No new primitive, no layer violation; two accepted concepts are modified, `structural` cannot carry conditions, named functions are privileged over lambdas until v0.2, and the Stage 11 matrix is not made (R7, R8). R7 and R8 applied as a deferrals register and an interactions draft on 2026-10-22; the formal matrix is Phase 6 work |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | **Flag** | Semantics are strategy-agnostic; "defined termination" on a violation holds only when the check is enabled, and no strategy document mentions contracts (R9, R10). R9 and R10 applied as proposed text on 2026-10-22 |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | **Flag** | Evolution path and reversibility are documented; minor conceptual debt (F2, two forms) and no verified precedent for the unification (R11, R12). R11 and R12 applied on 2026-10-22: precedents checked, cascade sized |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | **Flag** | Not measured; the grammar position of the conformance statement and the receiver spelling are open, and the diagnostics for the new error forms are not specified (R13–R15). R14 and R15 applied as proposed text and R13 as a protocol on 2026-10-22; the gate measurement is not run (a pilot of the protocol was made, Decision item 13) |

**Overall.** No Fail: one Pass, six Flags. Under the Gate Flow a Flag may
proceed but must be resolved before the final gate record. The root causes and the
proposed resolutions R1–R15 are below; R1–R15 were applied on 2026-10-22 as
proposed text (Decision items 10 to 13, the proposed glossary entry, the register
of deferrals, the interactions draft, the strategy notes, the precedent
verification and the measurement protocol). **R13 is applied as a protocol only:
the gate measurement has not been run** (a pilot of the protocol was made on
2026-10-22 and the protocol was revised on 2026-10-23; see Decision item 13).

#### Run of 2026-10-22

**1. `USER_VALUE_GATE` — Working Backwards — Pass**

*User story.* A programmer writes a rule once as a named specification, several
implementations declare that they obey it, and the compiler reports the
implementation that does not fit. *Press release (one line).* "Specifications
are named objects: write the rule once, check every implementation against it."
*FAQ.* Why not a library? Conformance checking and the Liskov rules for
conditions are compiler services (EDR-019 and EDR-056 already classify both as
Language). Why a rename? Only if `contract` replaces `trait`; with the name kept
(alternative A2) there is none. What does v0.1 get for functions? The shape check,
reuse across implementations, contract-derived tests and generated documentation;
higher-order use waits for v0.2.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| The problem is stated in user terms | Pass | The user story above; the `Withdraw` rule with `withdraw` and `withdraw_with_fee` is the concrete case |
| Justified by the Vision | Pass | `VISION.md`: "the language defines contracts; implementations fulfill them" |
| A realistic example shows the pain point | Pass | The business-rule example and `Sort<T>` with several implementations |
| The benefit outweighs the cognitive cost | Pass | For types, one mechanism instead of two with the same rules written twice is a net gain. For named functions the v0.1 payoff is narrower (see the FAQ) and the cost is one conformance statement, one hidden definition and a possible rename |
| Grounded in a classical problem | Pass | Behavioral subtyping and separation of a specification from its implementations, registered in `LANGUAGE_PROBLEMS.md` on 2026-10-21 |

*Fail conditions checked.* The problem exists outside the implementer's
experience; it cannot be a library (compiler-recognised conditions and
conformance); it is not speculative.

**2. `LOGICAL_CONSISTENCY_GATE` — Socratic Method — Flag**

*Terms.* Contract: the umbrella term for a named specification (type-level or
callable). Conformance: an element declares that it obeys a contract. F2: a
function declaration is sugar for a unique anonymous type with one callable
member. Successful exit and named result: as in EDR-097.

*Counterexamples tried.* A function that obeys a contract whose single member is
not named `call`; a type that obeys a short-form contract; a short-form and a
long-form single-member contract applied to the same function; a stronger
`requires` in the implementation (error by the inherited rule); a conditions-only
header (error, decision 24); a `structural` contract with a condition (error,
decision 21); a condition that cannot be evaluated (decision 22); the callable
member of the anonymous type, which is itself a function. *Devil's advocate:* a
function that obeys a contract gets only a shape check, since the body is not
verified; the same holds for inline clauses and is a Contract Enforcement Policy
matter, not a contradiction.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| All terms precisely defined | Flag | `GLOSSARY.md` and `CONTRACTS.md` define a contract as a verifiable assertion in a function signature; `TRAITS.md` calls a trait "a behavioural contract". The umbrella use changes both definitions and the glossary entry is not updated yet (R1) |
| No self-referential paradoxes | Flag | F2 read literally is circular: the callable member of the anonymous type is a function, which is again sugar for an anonymous type. Step 0 ends the regress at the `function` primitive of a type, but the definition does not say so (R2) |
| Behaviour deterministic across contexts | Flag | The matching rule depends on the subject: members by name and signature for a type, the single callable member by call signature for a function. Whether a single-member contract whose member is not `call` can be obeyed by a function is not stated (R3). Deterministic once stated |
| Composition with existing concepts is defined | Flag | Defined: generics (23), `structural` (21), invariants (18), member kinds (19), error sets (20), violations (22), markers and forms (24), `Callable` (EDR-080). Not examined: associated types on a contract obeyed by a function (Open Question 5) and which overload an `obeys` on a function binds (R4) |

*Fail conditions checked.* No contradiction of an established definition
remains once R1 is applied; the one restriction added to guard against a
mismatch (the conditions-only header) is a recorded deferral, not a patch.

**3. `CONCEPTUAL_SIMPLICITY_GATE` — Scientific Method — Flag**

*Hypothesis.* One construct with conformance for two kinds of subject is simpler
than a type-level mechanism plus a function-level mechanism. *Known.* Traits and
inline clauses exist; the constrained-type desugaring (EDR-080) already shows
`struct` + `Callable` + conformance. *Unknown.* The final words; how often
function-level contracts are reused. *Experiment.* Rewrite `Printable` and
`Withdraw` in the unified form: the rules for signature match, Liskov, coherence
and the orphan rule are stated once and not twice, and the third mechanism
(contract objects beside traits) never appears. The experiment is shorter than
the proposal. *Simpler alternatives checked.* A4 (inline clauses only) is simpler
and leaves the Context problems open; A3 (a pattern over `impl Callable`) adds no
keyword but needs a nameable wrapper type per implementation.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Expressible through composition | Flag | Function-level conformance is expressible today with a hand-written struct and `impl Callable` (A3), and the pipeline classifies it as a Level 2 pattern with no new primitive. The sugar is justified by the wrapper per implementation that A3 needs; the gate records the overlap (R5) |
| Single responsibility | Flag | The main responsibility is one conformance mechanism. The record also changes when `invariant` is checked (decision 18) and fixes the violation class (decision 22); these are separable from the unification (R6) |
| No keyword or syntax bloat | Pass | No keyword is added if `obeys` replaces `impl` and `contract` replaces `trait`; A2 keeps `trait`. `obeys` beside `impl` would be bloat and is excluded (decision 14) |
| Minimal learning surface | Pass | Learnable in isolation: "a specification is a named object; elements declare they obey it". F2 is not visible in v0.1 |

*Fail conditions checked.* It is not a library function; it does not duplicate an
existing concept, it removes a duplication; it is not sugar that a named function
could replace.

**4. `ARCHITECTURAL_INTEGRITY_GATE` — Logical Analysis — Flag**

*Premises.* Orthon has a Language Layer with Level 0 primitives, Level 1
operations and Level 2 patterns; enforcement is a Policy; implementation
architecture does not enter the language definition. *Deductions.* Type-level
conformance keeps its classification; function-level conformance is a Level 2
pattern (`pack` + `function` + conformance) and adds no primitive; enforcement
stays in the Contract Enforcement Policy and dispatch in the Dispatch Policy.
*Contradiction test.* None found against `ARCHITECTURE.md`. *Hidden premises.*
That the IR keeps contracts and conformance as named, queryable entities (Step 6,
dependency 1); that the Stage 11 interactions matrix will find no further
incompatibility.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Respects layer boundaries | Pass | Entirely within the Language Layer; the tooling and IR needs are consequences recorded in Step 6, not inputs to the definition |
| Composes orthogonally | Flag | One known incompatibility, deliberate: `structural` contracts carry no conditions (decision 21). The other restrictions are absent combinations deferred to v0.2 (function bounds in generics, `dyn` on functions, mutating short form), not exceptions to existing rules (R7) |
| No privileged position | Flag | Minor preferential treatment: named functions can obey a contract and lambdas cannot until v0.2 (R7) |
| Consistent with `ARCHITECTURE.md` | Pass | Level 2 pattern, no new primitive, no policy moved |

*Fail conditions checked.* The Standard Library does not need compiler internals.
**An existing concept must be modified:** TRAITS (keyword and conformance syntax),
CONTRACTS (invariant check, "Where Contracts Apply"), EDR-044 (clarified), the
glossary. This is the purpose of an amending EDR and is stated in Decision item 9
and Compliance; it is recorded here as the reason for the Flag. The pair-wise
interaction matrix is Phase 6 work (R8).

**5. `IMPLEMENTATION_INDEPENDENCE_GATE` — TRIZ — Flag**

*Contradiction.* One construct must serve every subject (generality) while
dispatch and checking stay cheap and independent of the strategy (an embedded
target cannot afford a vtable or a formatted diagnostic by default). *Separation
applied.* By condition: static dispatch is the default and `dyn` is opt-in and
visible. By time and policy: shape is checked at compile time; condition checks
are chosen by the Contract Enforcement Policy and may be elided in release
builds. *Known patterns.* Rust's traits and `Fn` family (general knowledge, not
verified); Dafny's abstract modules.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Semantic definition is strategy-agnostic | Flag | Conformance and conditions are strategy-independent. Decision 22 says termination on a violation is defined, but this holds only when the check is enabled; under elision a violated contract goes undetected, as in EDR-056 (R9) |
| All strategies can support it | Flag | At the time of the run no strategy document mentioned contracts (a search of `DEFAULT`, `EMBEDDED`, `HIGH_PERFORMANCE` and `LLM` strategies finds no occurrence). The diagnostic payload (contract, clause, values, guilty side) may need formatting or allocation that an embedded strategy cannot afford by default; the payload per strategy is unspecified (R10) |
| Performance is a strategy concern | Pass | The cost of checks is a Contract Enforcement Policy matter; the invariant check after each `proc` is part of the same policy; conformance has no run-time cost under static dispatch |
| Consistent with `IMPLEMENTATION_STRATEGIES.md` | Pass | No policy is moved; the Policy Footprint section of the hypothesis lists Contract Enforcement and Dispatch |

*Fail conditions checked.* The semantics are not expressed in terms of compiler
internals; no strategy is shown unable to implement the concept (the Flag is
unspecified coverage, not an impossibility).

**6. `LONG_TERM_MAINTAINABILITY_GATE` — Einstein's Method — Flag**

*One sentence.* A contract is a named specification, and whatever obeys it must
keep its promises. *To a non-expert.* The same rule can be written once and
checked against many implementations. *Remove one thing.* F2 (the hidden
definition) can be dropped from v0.1 text without changing what users write,
which decision 24 already allows; the short form can be dropped (then only the
long form remains); the `invariant` change can be split out. *Obviousness.* The
need for one specification mechanism is obvious; the exact words and the position
of the statement are not, which is why they are left to the syntax pipeline.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Evolution path documented | Pass | Decision item 7 lists what returns in v0.2 (function bounds, `dyn`, lambdas, multi-instance contracts, a condition contract with its own spelling, external imposition); `old` has its own deferral record |
| Conceptual debt assessed | Flag | Minor debt with documented mitigation: F2 is a hidden definition whose v0.1 role is decided after the syntax; the two forms of a contract are accepted as sugar; the kinds of methods are named differently in accepted and proposed documents (todo of 2026-10-21) (R11) |
| Reversible decision | Pass | Pre-v1.0 there are no external users; the merge can be undone by splitting the concept documents; the words are not fixed |
| Compatibility impact is bounded | Flag | The rename touches about 52 files and 35 `impl ... for` lines if A1 is chosen (fresh counts of 2026-10-22: 56 files and 41 lines, see the sizing below); minimal if A2 is chosen. Bounded by the words decision, which is open (R12) |
| Precedent in other languages | Flag | Contracts on interface members have production precedent (Eiffel, SPARK, Dafny; not verified from primary sources). One construct for type-level and function-level conformance has no verified precedent (R11) |

*Fail conditions checked.* The F2 definition could constrain the future
function-type notation; the risk is recorded in Open Question 1 and is the reason
F2 is kept as a definition only. No known corner is closed.

**7. `LLM_GENERABILITY_GATE` — Empirical Analysis — Flag**

*Structural scan.*

| Sub-construct | Result |
|---------------|--------|
| `contract` as a keyword | Possible association with Solidity, where it means a class-like unit (an assumption to test, not a finding); decided by the syntax pipeline |
| `obeys` in the signature tail | The tail already carries `require`, bounds and `->`; the position (P1, P2, P3) is open |
| `require` and `requires` | One letter apart in the same position, different meaning: an anticipated systematic error |
| Short form and long form of a contract | Two spellings of one thing; explicit sugar, but a source of mixed use |
| Mandatory parentheses of the short form | A model may drop them (`contract Positive -> result: Int`); the error is specified (decision 24) |
| Receiver spelling in type members | `fun size()` against `fun size(self)` is inconsistent across the documents; open |
| Habit from other languages | `trait` / `impl ... for` where `contract` / `obeys` is meant; `old(...)` (diagnostic of EDR-097) |

*Schema round-trip (grammar fragment, provisional).*

```text
ContractDecl  := 'contract' Name [TypeParams] ( '(' Params ')' '->' ResultSlot | Member+ ) ClauseBlock?
Member        := Kind Name '(' [Receiver ','] Params ')' [ '->' ResultSlot ] ClauseBlock?
Conformance   := Subject 'obeys' ContractRef { ',' ContractRef }        ; position open: P1, P2, P3
ClauseBlock   := ( 'requires' Expr | 'ensures' Expr )*                  ; the result name is bound in 'ensures' only
```

The fragment round-trips except for the position of `Conformance`, the receiver
spelling and the kind names.

| Criterion | Score | Reasoning |
|-----------|-------|-----------|
| Schema-serializable | Flag | Fully representable apart from the open position of the conformance statement, the receiver spelling and the kind names (R14) |
| Predictable generation | Flag | **Not measured** (no gate measurement was run; the pilot of Decision item 13 is not one). Anticipated systematic errors: habitual `trait` / `impl`, `require` for `requires`, the receiver spelling, the conditions-only header (R13) |
| No hallucination surface | Flag | The short and the long form have one meaning; under P2 a conformance could be written in two places. Each is resolved by choosing the position (R14) |
| Strategy-aware default | Pass | Contract semantics and naming do not depend on the strategy; enforcement is a Policy |
| Self-correctable | Flag | Rules and error cases are specified for kinds, error sets, `structural` conditions, bounds and violations; the diagnostic messages and hints for the conditions-only header, `self` in the short form and a member-name mismatch are not specified (R15) |

#### Resolutions (proposed; the author decides at sign-off of this EDR)

| # | Proposed resolution | Closes |
|---|---------------------|--------|
| R1 | Define "contract" as the umbrella term in `GLOSSARY.md`, keeping Trait and Function Contract as kinds, and say so in the merged concept (Compliance item 3) | Logical Consistency: terms (applied 2026-10-22, proposed) |
| R2 | State the base case of F2: the callable member is the `function` primitive of a type, not a free-function declaration; the sugar applies only to free declarations | Logical Consistency: paradox (applied 2026-10-22, proposed) |
| R3 | State the matching rule by subject: for a type, members match by name and signature; for a function, the contract must have exactly one callable member and it matches by call signature. Decide whether a single member with another name than `call` is an error with a hint (recommended) | Logical Consistency: determinism (applied 2026-10-22, proposed) |
| R4 | State that `obeys` on a function binds that specific declaration (each overload separately) and that a callable member has no associated types in v0.1; Open Question 5 stays for types | Logical Consistency: composition (applied 2026-10-22, proposed) |
| R5 | Record in the merged concept that function-level conformance is a Level 2 pattern over `struct` + `Callable` + conformance (A3 as the fallback) and why the sugar is kept: no wrapper type per implementation | Simplicity: composition (applied 2026-10-22, proposed) |
| R6 | Keep the `invariant` check rule (decision 18) as a separate item in the Decision History of the merged concept so it can be reviewed apart; the author may split it into its own EDR | Simplicity: single responsibility (applied 2026-10-22, proposed) |
| R7 | Keep the v0.2 deferrals of item 7 in the roadmap backlog, and keep decision 21 with the EDR-044 clarification | Architectural: orthogonality, privilege (applied 2026-10-22, proposed; the roadmap has no v0.2 backlog, so the register is `deferrable/CONTRACT_DEFERRALS_V02.md`) |
| R8 | Make the pair-wise interaction matrix for the merged concept in the Cross-Cutting Review (Stage 11) before final acceptance | Architectural: precondition (applied 2026-10-22, proposed as a draft in `notes/`; the formal matrix is Phase 6 work) |
| R9 | Qualify decision 22: termination on a violation is defined when the check is enabled by the Contract Enforcement Policy; under elision a violated contract is an undetected program error (as in EDR-056) | Implementation Independence: semantics  (applied 2026-10-22, proposed; Decision item 5 qualified) |
| R10 | Add to the strategy documents a note on contracts: which strategies enable checks by default and the minimal diagnostic payload for the embedded strategy (code and location) | Implementation Independence: strategy coverage  (applied 2026-10-22, proposed; notes in the strategy documents) |
| R11 | Verify the cited precedents from primary sources (Eiffel, SPARK, Dafny) and record the surfacing plan for F2 in v0.2 | Maintainability: precedent, debt  (applied 2026-10-22; verification below, partly verified; F2 plan in the deferrals register) |
| R12 | Decide the words (A1 or A2) in the syntax pipeline; the rename cascade is sized for the chosen option | Maintainability: compatibility  (routing confirmed 2026-10-22; cascade sized below; the words stay with the syntax pipeline) |
| R13 | Before the final gate record, run a measurement: a fixed set of generation tasks over the contract forms, success threshold 90 percent, recording failures from habitual `trait` / `impl`, `require` / `requires` and the receiver spelling (Compliance item 5) | LLM: predictable generation (protocol written 2026-10-22, pilot of the protocol 2026-10-22, protocol revised 2026-10-23; **gate measurement not run**) |
| R14 | Fix the position of the conformance statement (P1, P2, P3) and the receiver spelling in the syntax pipeline before the grammar is final | LLM: schema, hallucination surface (registered 2026-10-22 as blocking items in the syntax pipeline; nothing decided) |
| R15 | Specify the diagnostics with hints for the conditions-only header, `self` in the short form and a member-name mismatch | LLM: self-correctable (applied 2026-10-22, proposed; Decision item 13) |

R13: the protocol is written and was revised after a pilot; the gate measurement is not run. The Decision Journal table in
`DECISION_VALIDATION.md` is not updated, as for EDR-097.

#### Verification of precedents (R11, 2026-10-22)

Checked against primary documentation where it could be reached. "Verified" means
the statement was found in the cited primary source on the date above.

| Claim | Source | Result |
|-------|--------|--------|
| Eiffel: a redeclared routine combines its precondition with the original by "or" (`require else`, can only weaken) and its postcondition by "and" (`ensure then`, can only strengthen); invariants accumulate through inheritance | Eiffel documentation, "ET: Inheritance", section "Inheritance and contracts" | Verified from the language's own documentation. The ECMA-367 standard text and Meyer's *Object-Oriented Software Construction* were not read |
| That a precondition of a public routine may mention only names visible to callers (Eiffel) | Not found | Not verified |
| SPARK: an overriding operation's class-wide precondition must be weaker and its class-wide postcondition stronger; GNATprove checks Liskov substitution on every overriding operation, including operations inherited from an interface | AdaCore SPARK User's Guide, section "Object Oriented Programming and Liskov Substitution Principle" | Verified |
| Dafny: traits may declare methods with `requires` and `ensures`; only body-less methods may be redeclared in an extending class, and the class gives its own specification | Dafny Quick Reference; Ahmadi, "Automatic Verification of Dafny Programs with Traits" (Microsoft Research) | Partly verified. The reference manual could not be fetched and the direction of the override rule (weaker or stronger) was not confirmed |
| Rust: every function item has its own zero-sized type, distinct for different functions and for the same function with different generics; function items implement `Copy`, `Clone`, `Send`, `Sync`, and `Fn`, `FnMut`, `FnOnce` except for `unsafe`, `target_feature` or a non-Rust ABI; there is a coercion from a function item to a function pointer | The Rust Reference, "Function item types" | Verified |
| Liskov and Wing, "A behavioral notion of subtyping", ACM TOPLAS 16(6), 1994, pp. 1811–1841 (DOI 10.1145/197320.197383) | Bibliographic data from several indexes; a copy of the paper is hosted by Columbia University | Citation verified; the paper was not read in full |
| Meyer, "Applying Design by Contract", IEEE Computer 25(10), 1992, pp. 40–51 | Bibliographic data from several lecture notes; full-text copies exist | Citation verified; the paper was not read in full |
| One construct for type-level and function-level conformance has a production precedent | Searched; none found | **No verified precedent.** Contracts on interface members have precedent (Eiffel, SPARK, and Dafny partly); the unification across both subjects does not |

The Racket, Clojure, OCL and Haskell rows of the hypothesis were not re-checked.
The Flag of `LONG_TERM_MAINTAINABILITY_GATE` on precedent stays.

#### Rename cascade sizing (R12, 2026-10-22)

Counted in the `.md` files under `what/` and `how/` (code blocks only for the
declaration and `impl` lines). The counts include the unification's own documents.
The words are decided by the syntax pipeline; this sizing is an input.

| Measure | All | Accepted spec (`what/`, `how/` outside research and records) | Research and syntax hypotheses | Decision records |
|---------|-----|------|------|------|
| Files referencing EDR-019 or `TRAITS.md` | 56 | 18 | 20 | 18 |
| Files containing the word *trait* | 160 | 40 | 77 | 43 |
| Occurrences of the word *trait* | 1836 | 723 | 683 | 430 |
| `trait` declaration lines in code blocks | 48 in 24 files | 25 in 9 files | 21 in 14 files | 2 in 1 file |
| `impl ... for` lines in code blocks | 41 in 20 files | 19 in 9 files | 21 in 10 files | 1 in 1 file |

| Option | What changes | Size |
|--------|--------------|------|
| A1: `contract` replaces `trait`, `obeys` replaces `impl` | declaration lines, `impl` lines, the word in prose, the concept name | the whole table |
| A2: `trait` is kept, `obeys` replaces `impl` | the `impl ... for` lines and the conformance wording | the `impl` row only; the word *trait* stays |

Decision records are historical: a rename there is a note, not an edit. The
routing is confirmed: the conformance word and position are the row "Conformance
word and position" of the syntax queue, and the contract keyword is the row
"Contract keyword (`contract` vs `trait`)" coupled by pointer.

---

> **Human Sign-off:** pending. Required before this EDR is Accepted. Agents and
> GSD flows must not self-certify this field.
