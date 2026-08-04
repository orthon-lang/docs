# Correct-by-Construction and AI — Orthon's Pragmatic Position

> Exploratory note: a meta-analysis of how the Correct-by-Construction
> paradigm is already realized in Orthon, how it could be extended, and
> where it sits relative to the AI-assisted formal methods shift
> (Specware → LLM theorem proving). Not a language feature proposal.

## The Article's Thesis

Formal methods (Dijkstra, Specware, refinement chains) promised
mathematically provable correctness but stayed elite: high-order logic,
category theory, manual proof construction — a PhD barrier that confined
CbC to safety-critical niches.

The paradigm shift: AI agents redistribute the cognitive load.
- **Strategic work** (decomposition, invariant selection, refinement
  architecture) stays human.
- **Tactical work** (requirements → logical formulae, proof construction,
  consistency checking) shifts to AI (Baldur 2023: 65.7% of Isabelle/HOL
  theorems auto-proved; FVEL 2024: code → Isabelle → theorem proving).

The economics change: verified software moves from "months + PhD team"
to "weeks + engineer + AI assistant". Threshold drops for consensus
protocols, distributed systems, robotics controllers.

## How Orthon Already Applies CbC

Orthon is a **CbC language by design** — CbC emerges from composing
orthogonal accepted mechanisms, not from a single feature. See
`how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`.

### Three-Tier Invariant Classification

1. **Tier 1 — Type-level:** proven by the compiler (ownership, ADTs +
   exhaustive `match`, literal types, immutable-by-default).
2. **Tier 2 — Contract-level:** verified at debug/test time
   (`requires`/`ensures`/`invariant`, EDR-056). Constrained Types
   (EDR-080) straddle Tiers 1–2.
3. **Tier 3 — External:** outside the language ("this list is sorted").

CbC is strongest at Tier 1. Strengthening = migrating invariants
Tier 2/3 → Tier 1.

### The Seven Mechanisms

| # | Mechanism | Prevents | EDR |
|---|-----------|----------|-----|
| 1 | Ownership + move semantics | Aliasing → local, provable invariants (framing problem solved) | EDR-013 |
| 2 | Value semantics by default | Accidental shared mutable state | EDR-013 |
| 3 | Immutable-by-default (`val`) | Accidental mutation | EDR-013 |
| 4 | ADTs + exhaustive `match` | Missing variant handling (compile error) | EDR-039, 025 |
| 5 | Literal types | Typos in closed constants (`"opne"`) | EDR-043 |
| 6 | `Option<T>` / `Result<T,E>` | Null deref; unhandled absence/failure | EDR-018, 028 |
| 7 | Contracts | Tier-2 invariants, elided in release | EDR-056 |

### The Deepest Form: Separation Logic in the Type System

Ownership grounds CbC in Separation Logic (Reynolds 2002, O'Hearn 2019):
the spatial separation operator $P * Q$ is realized via the
single-owner invariant. If every value has exactly one owner, no alias
can invalidate a local invariant — the frame condition holds **by
construction**. This is Orthon's answer to the framing problem
(`what/SEMANTIC_MODEL.md` § Ownership, "Formal foundation").

Fortress (structural) vs. chain-link fence (guarded).

### Supporting Idioms

- **Constrained Types** (EDR-080): `type Age = Int requires v >= 0 && v <= 150`
  — nominal, runtime-enforced at entry boundaries. `Age(200)` = compile-time
  warning + runtime error (NOT compile-time error).
- **Parse, Don't Validate** (`notes/parse-dont-validate-idiom.md`):
  `parseEmail -> Option<Email>` over `isValidEmail -> Bool` — the type
  becomes the witness (antidote to Boolean Blindness).
- **Frame Conditions** (`deferrable/FRAME_CONDITIONS.md`): `fun`/`proc`/`new`
  are already a coarse frame condition on `self`; `@modifies` doc annotation
  proposed for free functions / non-`self` state.

## How CbC Could Be Extended

1. **Static Refinement Types** (`deferrable/REFINEMENT_TYPES.md`):
   for a decidable predicate subset (ranges, positivity, non-empty),
   reject invalid literals at compile time — turning EDR-080's warning
   into an error. No SMT solver. Migrates the most common value-range
   invariants Tier 2 → Tier 1.
2. **CbC as an explicit design principle** — open question; current
   recommendation is pattern, not principle (`DESIGN_PRINCIPLES.md` is
   locked, requires Tier 1 EDR).
3. **LLM Generability Gate scoring Tier-1 coverage impact** — require
   concepts to *increase* Tier-1 coverage, not erode it.
4. **External formal verification tooling** (Dafny-style) — possible
   future; explicitly NOT part of v0.1 core.

## The Key Insight: Positioning

| Aspect | Classical CbC (Specware) | Orthon | Article's AI + CbC |
|--------|--------------------------|--------|---------------------|
| Specification | Higher-order logic, category theory | Types + constrained types + contracts | Formal spec with AI assistant |
| Proof | Manual refinement steps | Compile-time (Tier 1) + runtime (Tier 2) | AI-generated proofs (Baldur, FVEL) |
| Entry barrier | PhD + months | Engineer + compiler | Engineer + AI assistant |
| Application | Crypto libs, avionics | Broad (LLM-generated code) | Critical distributed components |

Orthon is NOT trying to be Specware. It takes the **pragmatic CbC**
path: maximize what the type system proves structurally (Tier 1), use
contracts for the rest (Tier 2), and keep the door open for future
AI-assisted formal verification (Tier 3 → external tools). This is the
intermediate point the article describes as the goal — not replacing the
engineer, but redistributing cognitive load.

For an LLM-native language, CbC is a **safety net for nondeterministic
generation**: an LLM sampling tokens will eventually produce a typo,
missing case, or invalid state. If unrepresentable in the type system,
generated code fails at compile time — fast, cheap feedback — rather
than at runtime.

## Open Questions

1. Where do static refinement types sit on the roadmap?
2. Should the LLM Generability Gate score Tier-1 coverage impact?
3. Does Orthon's tooling (Milestone 10+) include any external prover
   integration, and should it be LLM-assisted?

## Related Documents

- `how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md` — the CbC pattern
- `how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md` — invariant classification
- `how/concepts/research/deferrable/REFINEMENT_TYPES.md` — static refinement hypothesis
- `how/concepts/research/deferrable/FRAME_CONDITIONS.md` — frame conditions
- `what/SEMANTIC_MODEL.md` § Ownership — Separation Logic grounding
- `what/concepts/CONSTRAINED_TYPES.md` — EDR-080
- `what/concepts/CONTRACTS.md` — EDR-056
- `notes/parse-dont-validate-idiom.md` — Boolean Blindness antidote
