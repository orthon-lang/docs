# Tooling: orthon prove — Optional Contract Proof Backend (Viper-style)

> **Status:** Open · **Priority:** P3 · **Target:** M3 (Tooling) / post-v0.1
> **Source:** [notes/orthon-vs-viper.md](../../notes/orthon-vs-viper.md)

---

## Description

An optional, external proof backend for Orthon contracts — a
"verifier as a service" in the style of Viper (ETH Zurich, the
infrastructure behind Gobra/Nagini/Prusti). When attached, the
compiler maps Orthon's ownership semantics onto a permission-based
separation logic (single-owner invariant ⇒ full permission per
location; `move` ⇒ permission transfer) and discharges contract proof
obligations automatically. Designed to be invisible to the user and
the LLM: the public specification surface stays Orthon's
pure-expression `requires` / `ensures` / `invariant`; no permission
syntax is ever exposed.

## Problem

Orthon's contract model is "static where possible, dynamic where
necessary" (EDR-056): contracts the compiler can prove statically
produce compile-time errors; the rest degrade to runtime assertions
in debug/test and are elided in release. A proof backend extends the
"static" end of that ladder for contracts a lightweight static
analyzer cannot discharge — without committing v0.1 to an SMT solver.
Without it, the strongest guarantees live in the type system, and the
"we can add proof later" claim rests on an unvalidated assumption.

## Spec Impact

### If this tool exists

- Ownership semantics get a validated, proof-carrying form:
  single-owner invariant ⇒ full permission; `move` ⇒ permission
  transfer.
- Contracts beyond the static analyzer's reach become provable at
  compile time, migrating Tier-2 invariants toward Tier 1.
- The LLM toolchain gains a "proof oracle": the LLM generates
  contracts, the backend checks them, structured diagnostics feed the
  generation loop.
- The IR's verification-friendly subset ("proof IR") pays off.

### If this tool does NOT exist

- Tier-2 contracts remain runtime-checked and elided in release — the
  intended v0.1 behaviour, so nothing breaks.
- The "future formal verification" claim stays hypothetical; the
  encoding burden is discovered later rather than designed for now.
- Loop invariants and ghost state, if ever wanted, are retrofitted
  rather than reserved by the spec.

## Language Feature Dependencies

- **Ownership / move semantics** — the mapping to permissions
  (single-owner ⇒ full permission) is the encoding basis. See
  [`what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) § Ownership.
- **Contracts (`requires` / `ensures` / `invariant`)** — EDR-056; the
  pure-expression contract language is the public surface.
- **IR** — must reserve a verification-friendly subset from the start
  (see [notes/orthon-vs-viper.md](../../notes/orthon-vs-viper.md)).
- **Ecosystem boundary** — classified as **Tooling** (M3) / external
  to the v0.1 core; explicitly NOT part of the v0.1 language
  semantics ([`why/GOALS.md`](../../why/GOALS.md) non-goal: formal
  verification).

## Spec Documents Affected

- [`how/architecture/IR.md`](../architecture/IR.md) — define the
  verification-friendly subset ("proof IR") now so contracts are not
  retrofitted later.
- [`what/concepts/CONTRACTS.md`](../../what/concepts/CONTRACTS.md) —
  note that `invariant` is reserved for aggregate (class/module)
  invariants; if full functional proof (loop invariants) is ever
  wanted, a distinct form must be reserved.
- [`what/OPTIMIZATION_MODEL.md`](../../what/OPTIMIZATION_MODEL.md) —
  classify proof backends as compile-time operations, not runtime
  semantics.

## Notes

- Guardrail: never expose Viper-style permissions or ghost code to
  users/LLMs. The backend is internal; the compiler maps ownership →
  permissions silently. Leaking the permission burden would fail the
  LLM Generability Gate (EDR-014).
- Partial correctness is the baseline; termination (`decreases`) is a
  separate, later concern.
- Concurrency: Viper permissions scale to thread interactions (this
  is what powers Gobra). Relevant only if Orthon later makes
  concurrency a proof target — currently deferred to the Execution
  Program / policies level.
