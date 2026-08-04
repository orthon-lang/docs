# Correctness by Construction

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-08-04
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

Traditional defensive programming treats correctness as a *runtime*
concern: validate inputs, throw exceptions, check assertions, and hope
the checks hold everywhere. Correctness by Construction (CbC) inverts
this: the language itself makes invalid states **structurally
impossible to express**, so a program that compiles is already
correct with respect to the invariants the type system can capture.

The classical framing — *"the aggregate is a fortress, not a
chain-link fence"* — exposes the weakness of runtime-only defense. In
languages with shared mutable state, an alias anywhere in the heap can
break a local invariant in ways no guard or contract can prevent. The
aggregate continues to *believe* it is valid while its internals are
already destroyed.

This document argues that Orthon is a **Correctness-by-Construction
language by design**, and catalogues the mechanisms that already realize
it, plus the interaction between them. It is a *pattern*, not a new
design principle: `DESIGN_PRINCIPLES.md` is locked and requires a Tier 1
EDR to modify; this document records the pattern without proposing such
a change.

## Principles

1. **Correctness at compile time over runtime checks.** An invariant
   the type system can prove is strictly stronger than one a guard
   checks at runtime — it cannot be missed on any code path.
2. **Structural guarantees over declared intent.** A type that *cannot
   represent* an invalid state beats documentation saying "callers must
   not pass null."
3. **Composition, not special cases.** Orthon does not add a single
   CbC feature; it achieves CbC through the *composition* of orthogonal
   mechanisms (ownership, ADTs, contracts, immutable-by-default).
4. **CbC is a lens, not a checklist.** When evaluating a new concept,
   ask: *"Does this make more illegal states representable, or fewer?"*

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Single-owner + move semantics make invariants local and provable (answers the framing problem) |
| Mutability Policy | Immutable-by-default prevents accidental mutation of shared data |
| Exhaustiveness Policy | `match` must cover all variants — missing cases are compile-time errors |
| Contract Enforcement Policy | Tier-2 invariants checked at debug/test time, elided in release |
| Allocation Policy | No GC/RC by default → deterministic lifetime, no hidden state |

## Model (What)

### Mechanisms Already in Orthon

CbC emerges from the composition of accepted mechanisms:

| Mechanism | What it guarantees | EDR / Source |
|-----------|--------------------|--------------|
| **Ownership + move semantics** | No aliasing → local, provable invariants (framing problem solved) | EDR-013 `SEMANTIC_MODEL.md` § Ownership |
| **Value semantics by default** | Assignment copies; no shared mutable state by accident | EDR-013 § Identity |
| **Immutable-by-default** | `val` bindings; mutation is explicit (`var`, `proc`) | EDR-013 § Mutation |
| **ADTs + exhaustive `match`** | Missing variant handling is a compile error | EDR-039, EDR-025 |
| **Literal types** | Typos in closed constants cannot compile | EDR-043 |
| **`Option<T>` / `Result<T,E>`** | Absence and failure are first-class, must be handled | EDR-018, EDR-028 |
| **Contracts** | Tier-2 invariants: `requires`/`ensures`/`invariant` | EDR-056 `CONTRACTS.md` |

### The Framing Problem and Ownership

The deepest form of CbC is **spatial separation of mutable state**. In a
language with shared mutable state, a "local" invariant is not local:
any alias can mutate the aggregate's internals in violation of its
guards. Orthon's ownership model embeds Separation Logic's spatial
separation operator ($P * Q$) in the type system: a value with exactly
one owner cannot be mutated by anyone else, so the invariant is provably
local. This is the difference between a fortress (structural) and a
chain-link fence (guarded).

See [`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Ownership
(Formal foundation) for the Separation Logic grounding.

### Invariant Classification

CbC applies only to what the language can prove. Orthon distinguishes
three tiers (see
[`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md)
§ Invariant Classification):

1. **Type-level invariants** — proven by the compiler (Tier 1).
2. **Contract-level invariants** — verified at debug/test time (Tier 2).
3. **External invariants** — outside the language; verified by tests or
   external tools (Tier 3).

CbC is strongest at Tier 1. Moving an invariant from Tier 2/3 to Tier 1
(refinement types, phantom types) strengthens the posture.

### LLM Generability Synergy

For an LLM-native language, CbC is a *safety net for nondeterministic
generation*. An LLM sampling tokens will eventually produce a typo, a
missing case, or an invalid state. If that state is unrepresentable in
the type system, the generated code fails at compile time — fast,
cheap feedback — rather than at runtime. This is measurable: the LLM
Generability Gate (`how/gates/DECISION_VALIDATION.md`) can require that
concepts *increase* Tier-1 coverage rather than erode it.

## Default Strategy

Adopt CbC as a documented cross-cutting pattern (this document), not as
a new entry in `DESIGN_PRINCIPLES.md` (which is locked). Guide future
concept evaluation with the question: *"Does this make more illegal
states representable, or fewer?"*

## Alternative Strategies

| Strategy | Description | Assessment |
|----------|-------------|-----------|
| **Explicit CbC principle** | Add CbC as a design principle in `DESIGN_PRINCIPLES.md` | Requires Tier 1 EDR; high ceremony, low marginal value (mechanisms already exist) |
| **Dependent types** | Prove arbitrary invariants in the type system | Powerful but complex; deferred (see `REFINEMENT_TYPES.md` for the mild form) |
| **Runtime-only (traditional)** | Guards, assertions, exceptions | Rejected — the fortress/fence argument; runtime checks do not compose |
| **Formal verification tooling** | External provers (Dafny-style) | Possible future; not part of v0.1 core |

## Open Questions

1. Should CbC be added as an explicit sub-principle under "Declarative
   With Static Guarantees" in `DESIGN_PRINCIPLES.md` (Tier 1 EDR), or
   kept as a documented pattern (this file)? Current recommendation:
   pattern, not principle.
2. Should the LLM Generability Gate be extended to score concepts on
   Tier-1 coverage impact?
3. Where do refinement types (Tier 3 → Tier 1 migration) sit on the
   roadmap? See [`REFINEMENT_TYPES.md`](../deferrable/REFINEMENT_TYPES.md).

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

## See also

- [`MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`](../essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md) — the cross-cutting pattern this document generalizes
- [`CONTRACTS.md`](../../../what/concepts/CONTRACTS.md) — Tier-2 invariants (EDR-056)
- [`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) — Ownership, Mutation, Identity dimensions
- [`REFINEMENT_TYPES.md`](../deferrable/REFINEMENT_TYPES.md) — the mild form of dependent types
- [`FRAME_CONDITIONS.md`](../deferrable/FRAME_CONDITIONS.md) — documenting what a function does not modify
