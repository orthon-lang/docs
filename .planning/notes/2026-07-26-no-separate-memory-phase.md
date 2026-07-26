---
date: "2026-07-26 00:00"
promoted: false
---

## Why memory questions don't get their own phase

### The question

Should ownership / mutation / lifetime / allocation questions — "memory
work" — be pulled out of Phase 2 (Semantic Model) and Phase 7 (Execution &
Optimization Model) into a dedicated phase of its own?

### Answer: no — the split that already exists is correct, once tightened

Memory-related concerns split cleanly across exactly two places already
defined in `when/ROADMAP.md`, and neither should move:

- **Conceptual/semantic memory questions** — what does it mean to own a
  value, when can it mutate, how long does it live — are three of Phase 2's
  six semantic dimensions (Identity, **Ownership**, **Mutation**,
  Evaluation, Visibility, **Lifetime**). They must stay inside Phase 2
  because Phase 2's own exit criteria require checking *cross-dimension*
  conflicts (ownership × mutation, evaluation × lifetime) as one unified
  model. Pulling "memory" out into its own phase would re-fragment exactly
  the kind of check Phase 2 exists to perform — the same
  collapsing-two-axes mistake already diagnosed once in
  [[2026-07-26-tier-vs-phase-mapping]] (tier vs. phase), just recurring at
  phase granularity instead of tier granularity.

- **Implementation/policy memory questions** — which allocation mechanism
  (arena, heap, GC, static), region/bump-allocator mechanics — are
  Allocation Policy, deliberately deferred past the semantic model and
  reconciled in Phase 7 (Execution & Optimization Model) against
  `how/strategies/`. This is required by the project's own anti-pattern
  rule ("Strategy-Specific Language Semantics" — `AGENTS.md`) and is
  exactly how `ALLOCATION.md` already models itself.

### The real gap this session found

The reason a separate phase felt necessary wasn't that the split is
wrong — it's that one file, `REGION_BASED_MEMORY_MANAGEMENT.md`, currently
sits entirely in the "Policy pocket" (per
[[2026-07-26-tier-vs-phase-mapping]]) even though it contains semantic
claims that belong in Phase 2 (linear/affine ownership — "each value has
exactly one owning name", no reference types, closure-capture-by-move,
`mut` semantics), mixed with genuine policy mechanics (arena structure,
bump-allocator performance, region-inference implementation).

Fix: split the file, don't relocate it wholesale. See the updated
[[move-policy-level-essential-concepts-out-of-pipeline]] todo for the
concrete action.

### Decision

No new phase. `REGION_BASED_MEMORY_MANAGEMENT.md` gets split between
Phase 2 (semantic content) and the Policy pocket → `how/strategies/`
(mechanism content) when that todo executes.
