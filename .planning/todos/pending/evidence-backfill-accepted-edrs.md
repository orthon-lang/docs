---
title: Evidence backfill for accepted EDRs (separate phase)
date: 2026-10-18
priority: low
status: pending
---

## What

Retroactively attach evidence (Programming Language Theory literature and
other languages' experience) to the accepted concepts and Architecture EDRs,
as a separate phase. It starts only after (a) the current re-verification of
accepted hypotheses is finished and (b) the Evidence Survey process EDR is
accepted (see [[draft-evidence-survey-process-edr]]). Not to be added to
`ROADMAP.md` until the author says so. Design and rationale:
[[2026-10-18-evidence-survey-step-design]].

## Why

Evidence found afterwards may change a decision, so it cannot be applied
automatically; but collecting it is cheap compared with the author's
attention. The phase separates collection (agent) from decision impact
(author).

## Working queue

About 57-76 units (counted 2026-10-18): 57 accepted concept docs and 76
accepted Architecture EDRs, largely overlapping. Excluded: 10 Process and 1
Quality EDRs (methodological evidence); 4 Rejected Architecture EDRs
(deferred). Cost per unit unknown - measure on a pilot of 2-3 concepts.

## Steps

1. **Theory base (agent, automatic).** Collect PLT literature the concept
   already relies on. Verify each cited work exists and supports what is
   claimed; mark the unverifiable as "unverified".
2. **Analogies and language experience (agent, no author).** Classified
   analysis: fit to Orthon (transfers / with caveats / does not transfer),
   points of attention, new insights. Registry entries get status `DRAFT`
   until the author marks them `VERIFIED`. Emit a non-binding impact signal
   per EDR (possible impact / no impact). Build the survey from the Step 1
   problem statement without reading the chosen solution.
3. **Decision impact (author only).** Agent pre-drafts a change pack per EDR
   (EDR claim, evidence, divergence, proposed edit) and applies nothing. The
   author reviews flagged EDRs and spot-checks unflagged ones. Outcomes as in
   `AGENTS.md` §11.2: clarify / revise concept plus EDR / not a problem.

## Suggested action

1. Wait for the re-verification to finish and the process EDR to be accepted.
2. Run a pilot on 2-3 concepts to measure cost per unit and tune the impact
   signal.
3. Re-estimate the volume; then decide whether to add a phase to `ROADMAP.md`.
