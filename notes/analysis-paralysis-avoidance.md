# Analysis Paralysis Avoidance

> Practical guardrails against over-analysis in the Orthon design process.
> **Status:** DRAFT — exploratory note.
> **See also:** [`DECISION_PROCESS.md`](../how/process/DECISION_PROCESS.md),
> [`DECISION_PIPELINE.md`](../how/process/DECISION_PIPELINE.md),
> [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md),
> [`concept-design-review.md`](../how/concept-design-review.md)

## Why this note exists

Orthon has many validation layers (Pipeline → Gates → Design Review →
Human sign-off → EDR). The risk is not too little thinking but
misallocated effort: spending full-EDR rigour on small decisions while
deferring consequential ones. This note collects guardrails for
deciding at the right depth and closing decisions instead of
re-analysing them.

## General principles

1. **One-way vs two-way doors.** Most decisions are reversible. If a
   decision can be undone cheaply, decide fast and revisit only when
   contradicting evidence appears. Reserve deep analysis for
   irreversible decisions.

2. **Satisficing over maximizing.** Pick the first option that passes
   the gates, not the hypothetical "optimal" option. The optimum for a
   language is unknowable in advance.

3. **Timeboxing.** Bound analysis by time or by number of alternatives
   (two to three is usually enough). More than three alternatives is a
   sign the problem is under-specified, not that more analysis is
   needed.

4. **Decide-and-record.** Paralysis ends with the act of writing: a
   decision plus rationale plus alternatives considered. Once recorded,
   stop re-chewing it.

5. **Separate what from how.** Do not block a semantics decision on
   performance or implementation questions — defer those to the
   strategy layer.

## Mechanisms already built into Orthon

- **Decision Pipeline is a quick-reject machine, not a deepener.**
  Q1–Q3 reject "library problem" / "already in primitives" cheaply;
  Q4 rejects principle violations; Q5–Q7 mark sugar/composition; Q8
  routes optimisation questions to
  [`OPTIMIZATION_MODEL.md`](../what/OPTIMIZATION_MODEL.md) instead of
  semantics. Use these as kill-switches on the first pass.

- **Decision Tiers bound the effort.** Tier 4 (small decisions) is an
  inline `> **Decision:**` note — not an EDR. A common cause of
  paralysis is upgrading a small decision to a full EDR with six gates.

- **Core Language vs Implementation Strategy.** Semantics are locked
  once; disputed "how" questions (allocation, lifetime, mutability) are
  deferred to swappable strategies. Do not wait for the "best" policy
  before fixing the data model.

- **Human sign-off.** A single-author convergence gate (Reviewed-by →
  LOCKED) instead of an open-ended committee review.

- **Lock-and-move review.** During concept review, iterate one question
  at a time, lock the outcome explicitly, then move to the next —
  never try to resolve everything at once.

## Working order when stuck

1. Classify the decision into its tier. If Tier 4, write the inline
   note and close it.
2. Run the Pipeline for speed: expect an early REJECT, not a
   justification of "why yes".
3. Limit alternatives to two or three; pick the first one that passes
   the gates.
4. Record the decision (inline / gate entry / EDR) and stop revisiting
   it until new evidence appears.
