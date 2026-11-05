# Dialectical Trade-offs

> Registry of Orthon's position on each axis of language design: the
> competing positions, the trade-off accepted, and the new contradiction
> each committed position creates. The axis tree itself lives in
> [`why/DESIGN_INFLUENCES.md`](../why/DESIGN_INFLUENCES.md); this document
> overlays Orthon on it. Reference index, not specification — canonical
> semantics stay in the linked EDRs and concept docs.

## Section A — Committed Axes

Each row records a position Orthon has committed to (via an EDR or an
accepted concept), the positions it declined, the trade-off it accepted,
and the new contradiction the position opens. A row is complete only when
all five cells are filled; see
[`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md)
§ Acceptance Bar.

| Axis | Orthon position | Competing positions | Trade-off accepted | New contradiction |
|------|-----------------|---------------------|--------------------|-------------------|
| Concurrency — task model | Colourless functions; execution policy is an explicit Execution Context value; the scheduler is an Implementation Strategy, not source semantics ([EDR-085](../how/decision_records/architecture/EDR-085-execution-context-invocation.md), [EDR-016](../how/decision_records/architecture/EDR-016-primitive-blocks.md)) | Coloured `async`/`await` (Python, JS, Rust); goroutines (Go); virtual threads (Java); actor isolation (Erlang, Elixir, Swift); structured concurrency (Kotlin, Swift) | Explicit policy values (`defer`, `delegate`, `spawn`, `fork`) and explicit materialisation (`await`, `take`, `next`, `gather`, `stop`) replace function-colour keywords — more surface syntax than `async`/`await` | Worker-lifecycle resolution moves from the runtime scheduler to the source: unresolved `spawn`/`fork` work cannot be silently abandoned (EDR-085), so bounded group completion (parent-child completion, automatic cancellation) becomes a recurring source-level obligation. Its general form is undecidable, so it re-appears as a policy the programmer re-declares per context |

## Section B — Open Axes

Axes where Orthon has no committed position, or only a hypothesis. Each
entry names the witness or classical problem that demands the axis, and
routes the gap to the Decision Pipeline.

| Axis | Demanded by | Orthon status | Next step |
|------|-------------|---------------|-----------|
| Evaluation strategy — strictness / totality | Haskell, Lean 4, Idris 2 (laziness, totality); Halting Problem as a limit | blank — no stated position | Decision Pipeline Q1 |
| Dependent and refinement types | Idris 2, Lean 4, Dafny, Vera; [`REFINEMENT_TYPES.md`](../how/concepts/research/deferrable/REFINEMENT_TYPES.md) | research | Decision Pipeline Q1 |
| Effect tracking — algebraic effects | Koka, Vera, Idris 2; the module `effects` header; [`EFFECT_FOOTPRINT.md`](../how/concepts/research/essential/EFFECT_FOOTPRINT.md) | research | Decision Pipeline Q1 |
| Evolution and compatibility | [`EVOLUTION_MODEL.md`](../how/EVOLUTION_MODEL.md) placeholder; Java (never break) vs Python 3 (epoch) | blank | Decision Pipeline Q1 |
| Heterogeneous execution — GPU / SIMD | Mojo, Bend, Flint vectors | blank | Decision Pipeline Q1 |

## Method

See [`how/process/DIALECTICAL_METHOD.md`](../how/process/DIALECTICAL_METHOD.md).
