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

## Scope decision (locked in exploration, 2026-10-18)

**Sequencing.** Nothing starts - including the collection steps below - until the current re-verification of accepted hypotheses is finished, so the yardstick does not move mid-review. (Background: hypotheses were first accepted without the Human Sign-off gate; the gate was later enforced and re-verification began but is not complete.)

**Retroactive application is a separate phase, not an automatic rollout.** Evidence can change a decision and trigger an EDR revision, which needs author review. The phase is split by who holds authority:

1. **Theory base** - Programming Language Theory literature the accepted concepts already rely on. Added automatically. The agent verifies that each cited work exists and says what is claimed of it (citations in the repo were largely LLM-written, so a fabricated or distorted reference is the typical failure). Unverifiable items stay marked "unverified".
2. **Analogies and language experience** - classified analysis (fit to Orthon, points of attention, new insights) of other languages' mechanisms and accumulated experience. Added without the author, but registry entries carry status `DRAFT` (the pattern from `what/THESES.md`) until the author marks them `VERIFIED`. The agent also emits a non-binding impact signal per EDR (possible impact / no impact); it routes author attention and decides nothing.
3. **Decision impact** - what to change in the EDR. Author only. The agent pre-drafts a change pack per EDR (EDR claim, evidence, divergence, proposed edit) and applies nothing. The author reviews flagged EDRs and spot-checks unflagged ones to catch signal bias. Outcomes are those of `AGENTS.md` §11.2: clarify the document / revise the concept plus a new EDR / not a problem, skip.

**Bias control.** The survey is built from the problem statement (Step 1) without reading the chosen solution; the comparison with the decision comes afterwards. A model handed a hypothesis tends to confirm it.

**Volume (counted 2026-10-18).** 92 EDR files: Architecture 81 (76 Accepted, 4 Rejected, 1 Superseded), Process 10, Quality 1; 57 accepted concept docs; 11 EDRs carry a recorded `Reviewed-by`. Working queue is about 57-76 units (concepts and Architecture EDRs largely coincide). Process and Quality EDRs are excluded (their evidence is methodological, not language experience); the 4 rejected EDRs are deferred. Cost per unit is unknown - measure with a pilot on 2-3 concepts. Unreconciled: `2026-10-17-spec-level-complexity-management.md` reports 11 superseded EDRs, the Status field shows 1.

Tracked in the todo "Evidence backfill for accepted EDRs".

## Source grading (locked in exploration, 2026-10-18)

Sources are graded by provenance, not by format: a language designer's talk about why a decision was made is first-hand material, while a peer-reviewed survey retelling someone else's work is second-hand.

| Grade | What | May support a claim in a concept or EDR? |
|-------|------|------------------------------------------|
| Evidence | Peer-reviewed works; normative and design documents of a language (specifications, RFCs, design documents); first-hand accounts by the people who made the decision (talks, posts) | Yes |
| Lead | Third-party blogs, talks, retellings, tutorials | No - a pointer to the primary source only; cite the primary once found |
| Not counted | Forum threads, unsourced summaries, model memory | No |

Each Evidence entry also records its **kind**: formal result / empirical study / experience report. First-hand accounts by language authors are experience reports and carry the label "opinion" (authors tend to justify their own decisions). Grade and kind are stored separately: an experience report stays weaker than a measurement even inside the Evidence grade.

## Registry structure (locked in exploration, 2026-10-18)

- **Location:** `how/evidence/` (How layer: evidence justifies decisions and is not specification; depth complies with `AGENTS.md` §4.4 rule 6). Accepted concepts in `what/` may link to it.
- **Name:** Evidence Registry (matches the Evidence Survey step). Per-mechanism files use the `_EVIDENCE` suffix (precedent: the `_SYNTAX` suffix in `how/syntax/`), which also avoids name clashes with same-named concept docs in `how/concepts/research/`.
- **Contents:**
  - `README.md` - conventions, source grading, statuses, pointer index. No second table of sources (the duplicated-index pitfall of `AGENTS.md` §10.12).
  - `SOURCES.md` - bibliography: one entry per source with grade, kind, status (`DRAFT` / `VERIFIED`) and a verification note. A source is verified once, independent of any concept.
  - `{MECHANISM}_EVIDENCE.md` - precedents per mechanism: language, mechanism, lineage, lesson, source references. No applicability verdicts: those are concept-specific and live in the concept's reference table.
  - `how/templates/_evidence.md` - entry template (templates live in `how/templates/`, `AGENTS.md` §4.4 rule 3).
- **Not placed elsewhere:** `what/` (justification, not specification); `how/concepts/research/` (110+ concept drafts by tier would swamp it); `notes/` (exploratory, not durable). `why/DESIGN_INFLUENCES.md` stays the vision-level overview of language influences and is linked from the registry; it was judged from its headings only, so check for overlap when the registry is created.
- **Housekeeping:** the new document type must be registered in `AGENTS.md` §3 (Document Map) and §4.3 / §4.4 (rule 7).

## Open questions

1. **Forward scope.** For new concepts, assumed Tier 1-2 only (matches the Human Sign-off gate); not yet confirmed.

## Unresolved (could not stand behind)

- That empirical or theoretical studies exist for each of Orthon's core mechanisms (ADT, traits, error handling, ownership) — *unverifiable*: stated from model memory, not checked against any primary source. Tracked as a research question; the "Where They Diverge" claim in `notes/evidence-based-software-engineering.md` ("no external empirical studies of its constructs exist") should be revisited once that is answered. For Orthon as a whole the claim holds; for individual mechanisms it is unconfirmed.

## Next

See the pending todo "Draft Process EDR: Evidence Survey (Step 2a)" and the research question "Per-mechanism evidence base for core constructs".
