---
date: "2026-10-18 09:00"
promoted: false
---

# Evidence Survey Step in Concept Design Review

> **Exploratory note.** Output of a `/gsd-explore` session. The direction below was agreed in conversation with the author; it is **not** a Human Sign-off and **no EDR exists yet**. Canonical process text stays in `how/concept-design-review.md` until a Process EDR changes it.

## Trigger

The author observed that analysis of external literature (what exists, how other languages already implement the topic a concept covers) is useful both when creating and when reviewing an EDR, and asked where to embed it: `AGENTS.md` §11.2, `how/process/DECISION_PIPELINE.md`, or somewhere else.

## Finding: the idea already exists, but is not enforceable

`how/concept-design-review.md` Step 2 contains one paragraph ("Before scoring, enumerate how other languages solve this problem"). Step 1 asks for "scientific grounding" and points to `what/LANGUAGE_PROBLEMS.md`. Four gaps make this advisory rather than binding:

1. **No defined output.** The paragraph does not say what artifact results, where it lives, or what counts as sufficient.
2. **Step 7 (Convergence Check) does not test for it.** A Human Sign-off can be recorded with zero external sources.
3. **`_edr-architecture.md` carries only the alternatives table.** There is no field for sources, so an EDR review has nothing to check.
4. **`AGENTS.md` has no rule for agents.** Nothing forbids asserting "in Rust it works like X" without a source.

## Why not §11.2 or the Decision Pipeline

- `AGENTS.md` §11 regulates *interaction* with the author (one question at a time, connected prose, closing the branch), not artifact content. A content requirement there mixes conversation protocol with design process.
- `DECISION_PIPELINE.md` is a cheap 10-question pre-filter answering "should the feature exist". A deep survey would bloat it. At most, Q1 gets a pointer to the survey result.

## Method: decision making + EBSE

Author's frame (decision making, aligned with EBSE per `notes/evidence-based-software-engineering.md`):

1. Formulate the problem or goal.
2. Collect and analyse information.
3. Find alternative solutions.
4. Evaluate the options (pros and cons).

In Concept Design Review today, Step 1 is the problem and Step 2 jumps straight to alternatives. Step "collect and analyse" is missing between them.

Author's four goals for the step:

| # | Goal | Output |
|---|------|--------|
| 1 | Identify precedents and established traditions; gain insights | Precedent table: language, mechanism, lineage, lesson |
| 2 | Identify possible implementation variants | Input candidate list for the alternatives step |
| 3 | Bind claims to sources (evidence) | Rule: an external claim carries a source or is marked unverified |
| 4 | Obtain a base to confirm hypotheses from scientific literature | Per hypothesis: supporting / contradicting works, or an explicit "no evidence found" |

## Agreed direction

- **Placement.** Split Step 2 into **2a Evidence Survey** and **2b Alternatives**. Numbering of Steps 3-8 does not change (Step 7 / Step 8 are referenced from `AGENTS.md` §7.4, templates, `SYNTAX_PIPELINE.md` and EDRs; renumbering would cascade).
- **What counts as evidence.** There are no studies of Orthon itself (it is being designed). Evidence is Programming Language Theory (PLT) literature plus the experience of other programming languages. Each source is labelled with its class: theory/literature vs. language experience (specifications, RFCs, design documents, retrospectives).
- **Applicability judgement.** Because Orthon's conditions differ from the source language's (LLM audience, Execution Program model, explicit semantics), every precedent carries a verdict: *transfers*, *transfers with caveats*, or *does not transfer*. This is EBSE critical appraisal and applicability.
- **Storage (hybrid).** A shared registry of verified sources and precedents (verification is the expensive part and is concept-independent), plus a thin reference table in each concept with the applicability verdict (concept-specific).
- **Reliability rule.** An external claim carries a source or is marked "unverified". A negative result ("no evidence found") is recorded.
- **Hooks.** Step 7 sub-check ("evidence surveyed, claims sourced"); a reference field in `_edr-architecture.md`; a pointer line in `AGENTS.md` §5.2; a reliability rule in `AGENTS.md` §11. Optional one-line pointer in `DECISION_PIPELINE.md` Q1.
- **Process level.** Tier 3 change: needs a Process-category EDR.

The registry follows an existing repository pattern: detailed reasoning trail in `gates/DECISION_LOG.md`, terse record in the EDR (`DECISION_PROCESS.md` § Recording).

## Open questions

1. **Scope.** Which concepts must run the survey (Tier 1-2 only, or all) and is it applied retroactively to the existing accepted EDRs? This is a cost question for a solo author.
2. **Secondary sources.** Are blogs, talks and second-hand retellings a lead only (never evidence), or excluded entirely? Only the two evidence classes above were agreed.
3. **Registry name and location.** A new document type requires a category directory, a template, and updates to `AGENTS.md` §3 and §4.4 (rule 7).

## Unresolved (could not stand behind)

- That empirical or theoretical studies exist for each of Orthon's core mechanisms (ADT, traits, error handling, ownership) — *unverifiable*: stated from model memory, not checked against any primary source. Tracked as a research question; the "Where They Diverge" claim in `notes/evidence-based-software-engineering.md` ("no external empirical studies of its constructs exist") should be revisited once that is answered. For Orthon as a whole the claim holds; for individual mechanisms it is unconfirmed.

## Next

See the pending todo "Draft Process EDR: Evidence Survey (Step 2a)" and the research question "Per-mechanism evidence base for core constructs".
