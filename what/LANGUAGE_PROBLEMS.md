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
