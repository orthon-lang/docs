---
created: 2026-10-21T00:00:00.000Z
title: Add complexity-signal escalation rule to the grading Process EDR
area: planning
severity: minor
files:
  - .planning/notes/2026-10-21-complexity-as-redesign-signal.md
  - .planning/notes/2026-10-18-graded-spec-growth.md
---

## Problem

The Process EDR planned at the start of Phase 04.5 (decision 6 of the graded-growth note) defines a grade, the cost-of-deferral question, the status format, the pre-release gate and the promotion rule. It has no rule for the case where the cost of deferral keeps a concept in grade 1 while the semantic budget vector (K, F, R, C) marks it as too heavy.

## Solution

Add a rule: exceeding the budget threshold returns the concept to redesign through a plane shift (down: common kernel; sideways: another carrier of the complexity). It never admits the concept with a recorded exception. Resolve first, or state as open, the thresholds and the criterion for choosing between the two kinds of shift (see the research question of 2026-10-21 in `.planning/research/questions.md`). Decide whether the rule lives in the grading EDR or in a separate one.
