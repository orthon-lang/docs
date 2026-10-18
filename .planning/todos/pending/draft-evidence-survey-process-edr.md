---
title: Draft Process EDR - Evidence Survey (Step 2a) in Concept Design Review
date: 2026-10-18
priority: medium
status: pending
---

## What

Draft a Process-category EDR that splits Concept Design Review Step 2 into
**2a Evidence Survey** and **2b Alternatives** (numbering of Steps 3-8
unchanged), and apply the resulting edits. Direction and rationale:
[[2026-10-18-evidence-survey-step-design]].

Files expected to change once the EDR is Accepted:

- `how/concept-design-review.md` - define 2a (inputs, outputs, sufficiency
  criterion, source classes, applicability verdict); add the Step 7 sub-check
  "evidence surveyed, claims sourced".
- `how/templates/_edr-architecture.md` - add a reference field for the
  evidence registry entries and the applicability verdicts.
- `how/templates/_concept.md` - add the thin reference table (concept ->
  registry entry -> applicability verdict).
- `AGENTS.md` - one pointer line in §5.2 (survey prior art first); a
  reliability rule in §11 (external claims carry a source or are marked
  unverified); new document type registered in §3 and §4.4.
- New evidence registry: category directory, template, and entries
  (name and location undecided).
- Optional: one-line pointer in `how/process/DECISION_PIPELINE.md` Q1.

## Why

Step 2 already says "enumerate how other languages solve this problem", but
it has no defined output, Step 7 does not test for it, the EDR template has no
field for sources, and agents have no rule against unsourced claims about
other languages. The author wants precedents, implementation variants,
sourced claims and a literature base for hypotheses to be part of both EDR
creation and EDR review.

## Blocked by open decisions (resolve first, one at a time)

**Sequencing (decided 2026-10-18):** do not start until the current
re-verification of accepted hypotheses is finished. Retroactive application
to accepted EDRs is out of scope here; it is the separate task
[[evidence-backfill-accepted-edrs]].

1. Forward scope for new concepts: assumed Tier 1-2 only (matches the Human
   Sign-off gate); confirm.
2. Source grading: decided 2026-10-18 (Evidence / Lead / Not counted, by
   provenance; first-hand author accounts are Evidence of kind "experience
   report", labelled opinion) - see the design note, "Source grading".
3. Registry name and location (new doc type: directory + template + AGENTS.md
   §3 / §4.4 updates).

## Suggested action

1. Resolve the three open decisions in a review session.
2. Run the research question "Per-mechanism evidence base for core
   constructs" on one pilot concept to test whether 2a is workable and what
   a sufficiency criterion should look like.
3. Obtain the author's Human Sign-off on the design (not self-certified).
4. Draft `EDR-NNN-evidence-survey-step.md` in
   `how/decision_records/process/` using `how/templates/_edr-process.md`.
5. Apply the file edits above; add the INDEX.md row in both tables.
