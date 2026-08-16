---
title: Draft Tier 1 EDR to amend Semantic Purity from "one meaning" to "one semantic role"
date: 2026-08-15
priority: high
status: pending
---

## What

`how/DESIGN_PRINCIPLES.md` is closed for modification; any change requires a
Tier 1 Architecture EDR. Draft that EDR to amend **Semantic Purity**:

- Old: "Each symbol or syntactic construct has exactly one meaning,
  independent of context. No exceptions."
- New (proposed): "Each symbol has exactly one **semantic role**. A role may
  appear in multiple grammatical positions, but must never change with
  context. Roles are enumerated in a table."

The EDR must also close the `:` question: `:` = "association" (type-of and
key-value are two positions of one role), consistent with EDR-083's range
rejection (step ≠ association) and EDR-041's map literal.

## Why

EDR-041 (`{key: value}`) and EDR-083 (range `:` rejected) apply Semantic
Purity inconsistently, and `:` currently violates both Semantic Purity and
DRY (it duplicates the `->` pair constructor). See
[[2026-08-15-colon-overload-analysis]].

## Suggested action

1. Run the symbol-role inventory (see the research question).
2. Draft `EDR-NNN-semantic-purity-one-semantic-role.md` in
   `how/decision_records/architecture/` using
   `how/templates/_edr-architecture.md`.
3. Reconcile the Semantic Purity table's `*` = pack/unpack entry with POLA.
4. Update `how/DESIGN_PRINCIPLES.md` only after the EDR is Accepted.
