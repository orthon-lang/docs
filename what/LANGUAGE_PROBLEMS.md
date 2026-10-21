# Language Problems — Classical Problems Orthon Solves

> Registry of classical programming-language theory (PLT) problems that
> Orthon explicitly addresses, the mechanism(s) that solve them, their
> scientific grounding, and how other languages solve them.
>
> This is a reference index, not specification: canonical semantics live
> in the linked concept docs and EDRs. The [Decision Pipeline](../how/process/DECISION_PIPELINE.md)
> (Q1) and the [Concept Design Review](../how/concept-design-review.md)
> (Step 1) point here when a proposal maps to a known problem; concept
> research uses it to ground proposals in established theory.
>
> **Conventions:** one problem per entry. Each entry names the problem,
> states the scientific grounding, lists the Orthon mechanisms and EDRs,
> and notes analogies in other languages. Add entries as new problems
> are identified and confirmed against the cited concept docs / EDRs.

## The Problems

### The Expression Problem

**Problem.** A system should be extensible along two axes — adding new
data variants and adding new operations over them — without modifying
existing code. Classic solutions specialise: functional style favours
adding operations over a fixed set of variants; OOP style favours adding
new subtypes but makes adding an operation invasive (it must be added to
every existing subtype).

**Orthon's solution.** Orthon separates data from behaviour, keeping both
extension axes open:

- `type` declares data shape only (product records, ADT variants).
  Behaviour attaches externally via traits and separate `impl` blocks;
  variants carry no inherent methods.
- Path A — one closed ADT (`type Shape = ...`) + one `impl` + `match` —
  optimises adding operations; a new variant breaks exhaustiveness
  (compile-time error).
- Path B — independent product types + per-type `impl` — optimises
  adding types; a new operation is a new trait plus impls for the types
  that need it. Path B inverts responsibility for defining operations.

**Mechanisms.** [`concepts/ALGEBRAIC_DATA_TYPES.md`](concepts/ALGEBRAIC_DATA_TYPES.md) § Behaviour,
[`concepts/TRAITS.md`](concepts/TRAITS.md),
[`concepts/PATTERN_MATCHING.md`](concepts/PATTERN_MATCHING.md)

**EDRs.** [EDR-019](../how/decision_records/architecture/EDR-019-traits.md) (traits),
[EDR-039](../how/decision_records/architecture/EDR-039-algebraic-data-types.md) (ADT, §4),
[EDR-025](../how/decision_records/architecture/EDR-025-pattern-matching.md) (pattern matching)

**Scientific grounding.** The Expression Problem was formulated by
Philip Wadler. Traits as composable units of behaviour were formalised
in *"Traits: Composable Units of Behaviour"* (Schärli, Ducasse,
Nierstrasz, Black, 2002). Orthon's trait model (explicit `impl`,
coherence / orphan rule, static dispatch by default) descends from
Haskell type classes and this traits research.

**Analogies in other languages.** Rust (`enum` ADT + external
`impl`/`trait` — closest match); Haskell / OCaml (`data` + typeclasses);
Swift (structs + protocols + extensions); Kotlin (sealed classes +
extension functions); Scala (case classes + traits / type classes);
TypeScript (discriminated unions, untagged, no exhaustiveness). Contrast:
Java / C# bind methods to data via inheritance — the rejected
alternative.

**Status:** Documented — concepts accepted (EDR-019, EDR-025, EDR-039). Entry added 2026-08-24.

### Behavioral Subtyping and Separation of Specification from Implementation

**Problem.** Code that depends on a specification (a type, an interface, a function signature) rather than on one implementation needs a rule for when another implementation can be substituted safely. A signature fixes shapes but not behaviour: an implementation can match the signature and still break the assumptions of its callers. Behavioral subtyping states the rule (an implementation may weaken preconditions and strengthen postconditions); Design by Contract makes the specification executable and attaches it to the declaration.

**Orthon's approach.** Traits (EDR-019) fix shape and conformance for types; contracts (EDR-056, amended by EDR-097) fix behaviour for functions. The hypothesis [`TRAIT_CONTRACT_UNIFICATION.md`](../how/concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md) proposes that both are one construct, a named specification that an element declares conformance to, with the substitutability rule stated once.

**Mechanisms.** [`concepts/TRAITS.md`](concepts/TRAITS.md),
[`concepts/CONTRACTS.md`](concepts/CONTRACTS.md)

**EDRs.** [EDR-019](../how/decision_records/architecture/EDR-019-traits.md) (traits),
[EDR-056](../how/decision_records/architecture/EDR-056-contracts.md) (contracts),
[EDR-097](../how/decision_records/architecture/EDR-097-contracts-result-named-old-deferred.md) (contracts amendment)

**Scientific grounding.** Liskov and Wing, *A behavioral notion of subtyping* (ACM TOPLAS, 1994); Meyer, *Applying Design by Contract* (IEEE Computer, 1992). Cited from general knowledge; to be checked against the primary sources before an EDR relies on them.

**Analogies in other languages.** Eiffel (inheritance with contract assertion rules); Dafny (abstract interface with an adherence check); Racket (first-class contracts attached at module boundaries); SPARK (pre- and postconditions on declarations); Clojure spec and OCL (specifications bound from outside). Contrast: Rust (traits for shape, no behavioural contract) and Haskell (typeclass laws in documentation), both from general knowledge.

**Status:** Partly documented — traits (EDR-019) and contracts (EDR-056, EDR-097) are accepted; their unification is a hypothesis under Concept Design Review. Entry added 2026-10-21.
