---
date: "2026-10-21 00:00"
promoted: false
---

# Implementation Complexity Is a Redesign Signal, Not a Budget Exception

> **Exploratory note.** Outcome of a `/gsd-explore` session on 2026-10-21 (topic: which complexity-management techniques are used in designing Orthon). Positions below are the author's. No external research was run; every factual claim comes from this repository and was read directly.

## Starting point

Techniques already in use, by what they act on:

| Group | Techniques |
|---|---|
| Fewer concepts | Orthogonality, Minimal Core (RISC-like), Principle of Parsimony, Justified Solutions |
| No exceptions | Semantic Purity, Uniformity, Consistency, Representation Symmetry |
| Separate axes | Schema Separate from Representation, Data vs Data Modifier, Core / Pattern / Library layers |
| Move complexity to the compiler | Intent Over Implementation, Declarative With Static Guarantees |
| Make it visible | Explicitness, Transparency, Named Before Symbolic |
| Spec-level | Graded growth, Primitive Closure Audit, semantic simplicity budget (see [[2026-10-17-spec-level-complexity-management]], [[2026-10-18-graded-spec-growth]]) |

## Positions

1. **Essential elements of the language.** Three of four candidates are irreducible parts of the language itself: the orthogonal decomposition into a semantic core with the seven Representations; the Data / Data Modifier split; the Execution Program with its Policies and Strategies.
2. **Gates, EDR, Concept Design Review are design discipline, not the language.** They control how the language grows but are not part of what Orthon is. The line between essential and accidental cannot be drawn until the milestone is complete and the hypotheses are pruned; until then the whole set is hypotheses that must agree on syntax.
3. **Order of growth.** Start from the minimal core needed for implementation, then add functionality with decreasing importance and increasing complexity relative to necessity. In the repository this is realized as grades whose criterion is the cost of deferral, not importance (decision 2 of the graded-growth note).
4. **Implementation complexity is a signal to simplify, cut, or reconsider a concept.** It is not a cost to be accepted with a recorded exception. When the semantic budget vector (K, F, R, C) says a concept is too heavy while the cost of deferral keeps it in grade 1, the concept goes back to redesign.
5. **Redesign means moving to another plane** (quantity turning into quality: when gradual trimming in the same plane stops helping, the fix lies in a different plane). Two kinds are both allowed, chosen by the situation:
   - **Down:** extract a common kernel from which the constructs derive (e.g. one control-transfer primitive in place of `loop` / `break` / `continue` / `?` / `?.` / suspend-resume, the gap found in spikes 001-004).
   - **Sideways:** move the complexity to another carrier (compiler built-in instead of macro layer for named arguments; one generic mechanism instead of `comptime T: type` plus `<>`).

## What this adds to the existing notes

- The graded-growth note has the brake's measurements (K, F, R, C, G) but no decision rule for crossing a threshold. Position 4 supplies the rule's direction; the threshold values and the escalation path remain undefined.
- Position 5 is unresolved on one point: there is no criterion for choosing between "down" and "sideways". See the research question of the same date.
- Information hiding (a module hides a decision likely to change) has no counterpart among the design principles; the nearest existing item is task T7 (spec subsystems) of the 2026-10-17 note.

## Not decided

- The criterion for choosing a down-shift versus a sideways shift.
- Numeric thresholds for the budget vector.
- Whether the escalation rule belongs in the Phase 04.5 Process EDR or in a separate one.
