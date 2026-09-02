# Orthon vs. Viper

**Status:** *Analysis — not a decision record. Neither a language
influence nor a design target; a reference point and a candidate
external proof backend for the post-v0.1 toolchain.*

**Date:** 2026-09-02

## Purpose

Compare Orthon with Viper — the ETH Zurich verification
infrastructure (https://viper.ethz.ch/tutorial/) that backs Gobra,
Nagini, and Prusti — to decide what Orthon can borrow and what it
should deliberately leave out. The core claim: Viper is not a sibling
language but the *proof infrastructure* for the whole family Orthon
sits beside, and the deepest resonance is semantic — both ground
reasoning in separation logic, Viper explicitly via permissions and
Orthon structurally via the single-owner invariant.

## Direct Relationship

No lineage: Orthon's declared influences
([`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md)) are only
Python and Java. Viper enters Orthon's design space through two
existing references:

- `why/GOALS.md` — formal verification of semantics is a stated
  **non-goal** for v0.1.
- [`how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`](../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md)
  — *"Formal verification tooling (Dafny-style) — possible future;
  not part of v0.1 core."*

So the relationship is prospective: Viper is the canonical
permission-logic infrastructure and the natural *encoding target* if
Orthon ever attaches a proof backend — the thing that makes the
"we can bolt on proof later" claim credible.

## Viper in Brief

- Verification infrastructure (ETH Zurich, Programming Methodology
  group), not a verifier for one language: an intermediate language
  plus two back-ends — Symbolic Execution (Silicon) and Verification
  Condition Generation (Carbon) — both discharging proof obligations
  to Z3.
- Front-ends (Gobra for Go, Nagini for Python, Prusti for Rust, ...)
  translate source languages into the Viper language and share the
  tooling.
- Native, first-class support for permission logics (separation
  logic, implicit dynamic frames): permissions express ownership of
  heap locations, used for heap-manipulating programs and thread
  interactions in concurrent software.
- Verifies partial correctness: `requires` / `ensures` / loop
  `invariant`; termination handled separately (`decreases`).
- Small, precise core language that is convenient to write manually
  yet expressive enough for automatic encoding.

## Orthon by Contrast

- LLM-native general-purpose language (design docs only; no
  implementation yet). Correctness is a *means*, not the end: it
  serves human comfort and reliable LLM code generation.
- Correctness by **structure**: illegal states are made
  unrepresentable — ownership + move semantics, value semantics by
  default, immutable-by-default, ADTs + exhaustive `match`, literal
  types, `Option<T>` / `Result<T,E>`.
- Contracts (`requires` / `ensures` / `invariant`, EDR-056) are a
  Tier-2 mechanism: checked statically "where possible", degraded to
  runtime assertions in debug/test, elided in release.
- Compiler as static analyzer (EDR-030): progressive verification
  layers built into the compiler pipeline — no SMT solver, no
  proof-carrying specification logic in v0.1. Aggregate
  (class/module) invariants; loops are not proved.
- Ownership is separation logic in the type system: the single-owner
  invariant realizes the frame condition **by construction**
  (`what/SEMANTIC_MODEL.md` § Ownership).

## Common Ground

1. **Separation logic as the correctness foundation** — Viper
   explicitly (permissions); Orthon structurally (single-owner
   invariant). Same answer to the framing problem, different carrier.
2. **Partial correctness semantics** — Viper guarantees properties
   *if* the program terminates; Orthon's `ensures` applies "after
   every successful return". Same baseline, and Viper supplies the
   vocabulary (`decreases`) for the day Orthon wants total
   correctness.
3. **Design-by-Contract surface** — `requires` / `ensures` /
   `invariant` appear in both, so contract syntax is recognizable
   between the two.
4. **Automation as a goal** — Viper is fully automated (no
   interactive proofs); Orthon's contracts are compiler-checked.
   Neither assumes a proof engineer in the loop.

## What Orthon Can Borrow

### 1. One IR, many front-ends / back-ends (architecture)

Viper's architecture — define one compact, permission-aware
intermediate language; let tools share the back-ends — is the proven
shape for a verification ecosystem. Orthon already has an IR
(`how/architecture/IR.md`) and an Execution Program model. The
lesson: reserve a *verification-friendly subset* of the IR
("proof IR") from the start, so contracts do not have to be
retrofitted once a proof backend is attached. Even without
implementing verification, the IR can be designed to stay encodable
into a permission logic.

### 2. Ownership → permissions is a nearly-free encoding (semantic)

The single most valuable transfer. Orthon's single-owner invariant
maps directly onto Viper's permission model: one owner ⇒ one full
permission per location; `move` ⇒ permission transfer across call
boundaries. What Viper spends effort proving (framing), Orthon
already guarantees by type. A future Orthon → Viper translation would
need no new permission annotations. Worth recording as a hypothesis
to validate.

### 3. Verification as a service, not a monolith (tooling)

Viper is a library that front-ends attach to. This is the right shape
for Orthon's LLM toolchain (M3+): a separate proof-backend "oracle"
that the IDE, CI, test generators, and the LLM itself call. The LLM
generates contracts; the backend checks them; structured diagnostics
feed the generation loop. Directly relevant to the open question in
[`correct-by-construction-and-ai.md`](correct-by-construction-and-ai.md)
("should external prover integration be LLM-assisted?").

### 4. Partial correctness + separate termination (semantics)

Adopt Viper's clean separation now, at spec level: contracts
guarantee behaviour on successful return; termination is a separate
concern with its own vocabulary. Orthon already does this implicitly;
making it explicit keeps the door open to total correctness without
re-architecting contract semantics.

## What Orthon Should NOT Borrow (Guardrails)

- **Explicit permissions.** Never expose permission syntax to users
  or LLMs. Ownership gives framing for free; leaking Viper's
  permission/invariant burden would import the PhD barrier Orthon
  exists to avoid and would fail the LLM Generability Gate (EDR-014).
  Viper must remain an *internal* backend; the compiler maps
  ownership → permissions silently.
- **Manual loop invariants in v0.1.** Viper (like Dafny) requires
  loop invariants and `decreases`. Orthon deliberately does not prove
  loops. Caveat: `invariant` is already claimed by aggregate
  invariants, so if full functional proof is ever wanted, a distinct
  loop-invariant form must be reserved now rather than retrofitted
  into an overloaded keyword.
- **Ghost-code surface.** Viper's ghost state is powerful but heavy.
  Orthon v0.1 keeps only ghost bindings (`result`, `old`). Decide
  whether "no ghost code" is a principle or a v0.1 simplification; if
  it is relaxed later, Viper is the reference design for how to scope
  spec-only state cleanly.

## Differences

| Axis | Viper | Orthon |
|------|-------|--------|
| What it is | Verification infrastructure (IR + SE/VCG back-ends → Z3) | Language with correctness built into the type system |
| Correctness carrier | Explicit permissions (separation logic) | Structural: single-owner invariant in the type system |
| Specification surface | requires/ensures, loop invariants, ghost state, termination | Pure-expression contracts, aggregate invariants; no loop proofs, no ghost code |
| Who writes specs | Verification engineer / tool builder | Human **and** LLM; kept intentionally light |
| Position vs. language | External; front-ends translate source languages into it | The language itself is the guarantee; external proof is optional post-v0.1 |
| Maturity | Shipped research infrastructure (Gobra at CAV 2021, ...) | Specification in progress; no implementation |

## Synthesis

Viper's value to Orthon today is not code — it is *de-risking*. It
demonstrates that Orthon's ownership model encodes cheaply into a
permission logic, that partial correctness is the right baseline
semantics, and that "verifier as a service" is a working shape for
LLM-assistive tooling. Orthon keeps Viper in mind as the natural
proof backend of a post-v0.1 future — and as a reminder of the expert
load it deliberately declines to inherit.

## See Also

- [`how/tooling/ORTHON-PROVE.md`](../how/tooling/ORTHON-PROVE.md) —
  tooling requirement: optional contract proof backend (Viper-style)
- [`../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`](../how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md)
- [`what/concepts/CONTRACTS.md`](../what/concepts/CONTRACTS.md) and
  EDR-056
  ([`how/decision_records/architecture/EDR-056-contracts.md`](../how/decision_records/architecture/EDR-056-contracts.md))
- [`correct-by-construction-and-ai.md`](correct-by-construction-and-ai.md)
- [`orthon-vs-dafny.md`](orthon-vs-dafny.md)
- [`why/GOALS.md`](../why/GOALS.md) (non-goals: formal verification)
