# Orthon vs. Dafny

**Status:** *Analysis — not a decision record. Neither a language
influence nor a design target; a reference point Orthon positions
itself against.*

**Date:** 2026-09-02

## Purpose

Compare Orthon with Dafny — Microsoft Research's verification-aware
programming language — to make explicit what the two share and where
they diverge. The core claim: there is **no direct lineage** between
the languages, yet they belong to the same family (Correct-by-
Construction / Design-by-Contract) while making opposite bets on
*where correctness comes from*.

## Direct Relationship

Dafny does not appear in Orthon's declared influences
([`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md) lists only
Python and Java). The project references Dafny only as an explicit
boundary of what Orthon deliberately is *not* in v0.1:

- `how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`
  — *"Formal verification tooling (Dafny-style) — possible future; not
  part of v0.1 core."*
- `why/GOALS.md` — formal verification of semantics is a stated
  **non-goal** for v0.1 (prose-based spec; Coq/Lean formalization
  deferred).

So the relationship is conceptual, not genealogical: Dafny is the
canonical representative of the CbC family, and Orthon positions itself
within that family while choosing a different correctness mechanism.

## Dafny in Brief

- Verification-aware language (K. Rustan M. Leino, Microsoft Research,
  2009). Compiles/transpiles to C#, Java, JavaScript, Go, Python.
- Core is **proof**, not execution: programs carry a formal
  specification (`requires` / `ensures` / loop `invariant` /
  `decreases` for termination) and the verifier discharges proof
  obligations automatically via **Dafny → Boogie (intermediate
  language) → Z3 (SMT solver)**. Framework: Hoare logic.
- Rich mathematical toolbox: mathematical integers/reals, bit-vectors,
  sequences, sets, multisets, infinite sequences/sets, induction,
  co-induction, calculational proofs, lemmas as ghost methods.
- Side-effect reasoning via `modifies` clauses and a separation-logic
  variant called *implicit dynamic frames*.
- OOP support: generic classes, dynamic allocation, inductive datatypes.

## Orthon by Contrast

- LLM-native general-purpose language (design docs only; no
  implementation yet). Correctness is a *means*, not the end: it serves
  human comfort and reliable LLM code generation.
- Correctness by **structure**: illegal states are made unrepresentable
  — ownership + move semantics, value semantics by default,
  immutable-by-default, ADTs + exhaustive `match`, literal types,
  `Option<T>` / `Result<T,E>`.
- Contracts (`requires` / `ensures` / `invariant`, EDR-056) exist but
  are a Tier-2 mechanism: checked statically "where possible", degraded
  to runtime assertions in debug/test, elided in release.
- Compiler as static analyzer (EDR-030): progressive verification
  layers built into the compiler pipeline — no SMT solver, no
  proof-carrying specification logic in v0.1.

## Common Ground

1. **Design by Contract** — nearly identical keyword surface:
   `requires` / `ensures` / `invariant`, so specification syntax is
   recognizable between the two languages.
2. **Correct-by-Construction philosophy** — Dafny is the canonical CbC
   language; Orthon is CbC *by design through composition*
   (CORRECTNESS_BY_CONSTRUCTION.md).
3. **Immutable collections + imperative/functional blend** — Dafny's
   `seq`/`set`/`map` vs. Orthon's Sequence/Set and immutable
   collections by default.

## Differences

| Axis | Dafny | Orthon |
|------|-------|--------|
| Correctness mechanism | Automatic proof (SMT/Z3, Hoare logic) | Structural impossibility of illegal states (types + ownership) |
| Invariants | Loop invariants + `decreases` (termination) | Aggregate invariants (class/module); loops are not proved |
| Specification logic | Mathematical types, lemmas, ghost methods, induction | Pure expressions; no lemmas, no ghost code |
| Side effects | `modifies` + implicit dynamic frames | Ownership/move (separation logic in the type system) + coarse frame conditions (`fun`/`proc`/`new`) |
| Contracts | Proven statically | Static "where possible", else runtime assertions; elided in release |
| Execution | Transpilation to C#/Java/JS/Go/Python | Execution Program / Engine (interpret, AOT, container, WASM from one artifact) |
| Audience | Verification engineer; safety-critical | Human programmer **and** LLM as code generator |
| Maturity | Shipped tool since 2009 (compiler + verifier + IDE) | Specification in progress; no implementation |

## Synthesis

Dafny answers *"prove that this specific program is correct"*; Orthon
answers *"make it structurally awkward to write an incorrect program."*
For Dafny correctness is the product; for Orthon it is a side effect of
orthogonality serving a pragmatic goal — humans and LLMs fail cheaply
(compile error), not expensively (runtime error).

These are not competitors and not relatives; they are two poles of one
tradition. Orthon keeps Dafny in mind precisely as what it consciously
chose not to be in v0.1 — a formal-proof language.

## See Also

- [`how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`](../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md)
- [`what/concepts/CONTRACTS.md`](../what/concepts/CONTRACTS.md) and
  EDR-056 (`how/decision_records/architecture/EDR-056-contracts.md`)
- [`notes/correct-by-construction-and-ai.md`](correct-by-construction-and-ai.md)
- [`why/GOALS.md`](../why/GOALS.md) (non-goals: formal verification)
