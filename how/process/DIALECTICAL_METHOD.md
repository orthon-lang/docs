# Dialectical Method

> Method for reconstructing the dialectical path of programming-language
> decisions, overlaying them on Orthon, and naming the trade-off and the
> new contradiction each decision accepts. This is a confirmation
> discipline on the existing hypothesis → concept pipeline, not a new
> pipeline.

## Purpose

Orthon declares it inherits seven decades of high-level language design.
The dialectical method makes that inheritance explicit: for each axis of
language design, it reconstructs which contradiction a language resolved
(thesis → antithesis → synthesis) and which new contradiction that
synthesis created. The output is not a feature list but a map of tensions
plus hypotheses about Orthon's next synthesis, which then route through
the [Decision Pipeline](DECISION_PIPELINE.md).

## The Three Sources

The axis tree is built by merging three sources already in the
repository, not by fresh research of every donor language:

1. **The generational chain** — [`why/DESIGN_INFLUENCES.md`](../../why/DESIGN_INFLUENCES.md)
   records what each language solved; each row names one pole of an axis.
2. **The classical problems** — [`what/LANGUAGE_PROBLEMS.md`](../../what/LANGUAGE_PROBLEMS.md)
   and [`notes/plt-classical-problems.md`](../../notes/plt-classical-problems.md)
   record the mandatory decision points every language must answer.
3. **The donor set** — each donor language's raison d'être names an axis.

Merging the three collapses duplicates into a tree; whatever does not
collapse is either a new axis or a non-axis.

## Completeness Test

The axis set is complete, relative to its sources, when three conditions
hold:

- every classical problem maps to at least one axis, or is declared a
  **limit** (an undecidability result) rather than an axis;
- every axis is reachable from at least one problem and witnessed by at
  least two opposed donor positions;
- every donor's raison d'être maps to a position on an axis.

Completeness is a fixpoint, not a count: it is relative to the declared
problem catalog, which is finite and canonical.

## The Three Steps

### Step 1 — Reconstruct the tree

Build the axis tree from the three external sources. This step is
external: no Orthon positions. Its product is the tree in
[`why/DESIGN_INFLUENCES.md`](../../why/DESIGN_INFLUENCES.md).

### Step 2 — Overlay Orthon

For each axis, mark Orthon's position: **committed** (an EDR or accepted
concept), **research** (a hypothesis), or **blank** (no position). A
blank is a gap — an axis Orthon has never decided — and is the primary
output of the overlay.

### Step 3 — Name the trade-off and the new contradiction

For every committed axis, record the position, the competing positions,
the trade-off accepted, and the new contradiction the position creates.
This is the deliberate generation of the next tension.

## Acceptance Bar

A synthesis entry is accepted only if it names:

1. **both poles** — the opposed positions being reconciled;
2. **the holding mechanism** — how the language holds both at once;
3. **the cost** — the new contradiction the synthesis creates.

Without the third, the entry is a timeline, not a dialectic.

## The Relocation Hypothesis

A working hypothesis guides Step 3: every synthesis relocates a decision
across a boundary — compile time vs runtime, programmer vs compiler,
language vs library. Its cost is exactly the class of programs for which
the relocated decision becomes undecidable, and so is approximated
conservatively or checked at runtime. Examples: GC relocates liveness to
the runtime (statically undecidable → tracing and pauses); Rust relocates
aliasing to the compiler (conservative, incomplete borrow checking); Zig
relocates execution to `comptime` (halting at build time → eval quotas).

## Stop Rule

The method is usable after one axis. Do not block on exhaustive coverage:
the ledger fills incrementally, and each completed axis is immediately
useful. Research only the axis in front of you.

## Outputs

- **Axis tree** — [`why/DESIGN_INFLUENCES.md`](../../why/DESIGN_INFLUENCES.md)
  (external view).
- **Ledger** — [`what/DIALECTICAL_TRADEOFFS.md`](../../what/DIALECTICAL_TRADEOFFS.md):
  committed axes with trade-off and new contradiction (Section A), and
  open axes (Section B).
- **Hypotheses** — open axes route to the Decision Pipeline, Q1.
- **New-contradiction field** — every EDR names the tension it creates,
  per [EDR-102](../decision_records/process/EDR-102-new-contradiction-field.md).
