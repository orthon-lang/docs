# Evidence-Based Software Engineering and Orthon

> **Status:** Exploratory note
> **Date:** 2026-08-04
> **Source question:** How does Evidence-Based Software Engineering (EBSE) relate to Orthon?
> **Reference:** Kitchenham, Dybå, Jørgensen — "Evidence-Based Software Engineering", ICSE 2004 (IEEE 1377125)

## What EBSE Is

EBSE adapts evidence-based medicine to software engineering. Five steps:

1. Convert a relevant problem into an answerable question.
2. Find the best evidence with which to answer it (systematic literature reviews, empirical studies).
3. Critically appraise the evidence for validity, impact, and applicability.
4. Integrate the appraisal with practical experience and the customer's requirements.
5. Evaluate the outcome and seek ways to improve.

Core thesis: software engineering decisions should rest on empirical evidence,
not on expert opinion or intuition alone.

## Relationship to Orthon

Orthon does not explicitly cite EBSE, but its design process is EBSE applied to
language design. The overlap is at the level of spirit, not letter.

| EBSE principle | Orthon instantiation |
|---|---|
| Decisions from evidence, not intuition | Empirical Analysis Method: "LLM-friendliness cannot be assumed from intuition — it must be proven" |
| Systematic review of prior evidence | `DESIGN_INFLUENCES.md` — systematic analysis of the language evolution chain (Assembler → Zig) |
| Critical appraisal | Seven independent validation gates, each with its own method |
| Integrate evidence + experience + requirements | Decision Pipeline (10 questions) + Working Backwards Method |
| Transparent decision recording | EDR with full context: alternatives considered, rationale, consequences |

## Three Levels of Correspondence

**1. Philosophical.** Zen of Orthon ("Just enough. Nothing less, nothing more.") and
EBSE share the same intellectual tradition — parsimony, decisions based on what is
proven rather than what seems right.

**2. Methodological.** The validation gates are EBSE decomposed into seven
independent lenses:

- `USER_VALUE_GATE` → EBSE steps 1, 4 (user problem + integration with requirements)
- `LOGICAL_CONSISTENCY_GATE` → EBSE step 3 (internal validity)
- `LLM_GENERABILITY_GATE` with Empirical Analysis → EBSE steps 2–3 (structural scan + schema round-trip as empirical verification)
- Fitness Functions → evidence-based metrics: measurable checks, not subjective opinions

**3. Structural.** The EDR is a direct analogue of an EBSE record: Context (problem →
evidence question), Decision + Rationale (appraisal), Alternatives Considered (review
of evidence), Consequences (integration). `DECISION_PROCESS.md` formalizes this in a
four-tier system (Tier 1 principles → Tier 4 inline decisions).

## Where Orthon Goes Beyond EBSE

1. **EBSE is a meta-methodology for the whole field; Orthon instantiates it in one
   discipline** (language design). EBSE says "one should"; Orthon says "here is how."
2. **Orthon adds the LLM Generability gate.** EBSE predates LLMs. Orthon extends the
   evidence-based approach to a new dimension: "is this construct proven LLM-generable?"
   — via structural scan + schema round-trip.
3. **Fitness Functions as automated evidence.** EBSE assumes manual appraisal. Orthon
   introduces measurable metrics (Composition Surface: canonical-form count diff;
   Layered Isolation: impact analysis per EDR) that can be automated.

## Where They Diverge

EBSE emphasizes external empirical data — literature reviews, meta-analyses,
controlled experiments. Orthon designs a *new* language: no external empirical
studies of its constructs exist. Its "evidence" is therefore:

- Comparative analysis of predecessor languages (`DESIGN_INFLUENCES.md`)
- Internal consistency (validation gates)
- Structural/schema analysis for LLM generability
- Decomposability onto Primitive Blocks

This is not a contradiction but an adaptation — EBSE applied to a domain where RCTs
are infeasible. Orthon does the best available: systematic comparative analysis plus
internal consistency proofs instead of unavailable external experiments.

## Summary

Orthon relates to EBSE as an engineering project to its scientific methodology.
EBSE supplies the general principle — decisions must be evidence-based. Orthon
embeds that principle at every level of language design, from Zen aphorisms to the
concrete `EMPIRICAL_ANALYSIS_METHOD.md`, from philosophy to measurable Fitness
Functions.
