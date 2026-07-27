# EDR-014: Decision Log

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Process

**Scope:** Project

---

### Context

`DECISION_VALIDATION.md` defines seven gates and, for each, a
specific reasoning **method** (`gates/methods/*.md`, established by
EDR-003). When Phase 2 ran all seven gates against the completed
Semantic Model, the result recorded in `what/SEMANTIC_MODEL.md` §
Validation was a compact table: one verdict and a two-to-three
sentence justification per gate, citing evidence already established
elsewhere in the document.

That compact table is the right shape for the artifact itself — a
733-line specification document should not carry seven full worked
derivations inline. But the record of *how* each gate's verdict was
reached — which questions the method asked, what evidence answered
them, where a counterargument was tested and survived — did not exist
anywhere. `DECISION_VALIDATION.md`'s own Decision Journal (established
alongside the gate system) is a one-row-per-decision summary table
(date, gates applied, verdict, notes) — equally terse, by design.
Neither location was ever intended to hold the reasoning trail itself.

Without a defined home for that trail, the reasoning either (a) never
gets written down at all — the verdict appears without the analysis
that produced it, indistinguishable from an asserted opinion — or (b)
gets stuffed into the artifact being validated, bloating specification
documents with process detail that belongs one level removed from the
content programmers and future phases actually need to read.

### Decision

Establish **`how/gates/DECISION_LOG.md`** as the canonical location
for the detailed, per-gate reasoning trail behind Tier 1–2 decisions
(per `DECISION_PROCESS.md`'s Decision Tiers).

Structure: one entry per validated decision (linked to its EDR and
the artifact it validates), containing one subsection per gate
applied. Each subsection names the method used (linking
`gates/methods/*.md`), works through that method's actual steps
against the specific proposal, and states the resulting verdict —
not just the verdict, the work that produced it.

This is a third, distinct layer alongside two that already exist:

| Layer | Location | Granularity | Audience |
|---|---|---|---|
| Verdict + citation | The artifact itself (e.g. `SEMANTIC_MODEL.md` § Validation) | One row per gate, evidence-cited | Anyone reading the spec |
| Summary record | `DECISION_VALIDATION.md` § Decision Journal | One row per decision | Anyone auditing what was validated when |
| Reasoning trail | `gates/DECISION_LOG.md` | One worked derivation per gate per decision | Anyone auditing *how* a verdict was reached, or applying the same method to a future decision |

An artifact's Validation section should link to its Decision Log
entry (and vice versa); the Decision Journal row for the same decision
should link to the same entry, so all three layers are navigable from
any one of them.

### Without It

| Risk | Severity | Manifestation |
|------|----------|----------------|
| Unverifiable verdicts | High | A "Pass" or "Flag" is asserted with a one-line citation; no way to check whether the method was actually applied or the verdict was intuited |
| Method drift | Medium | Without a record of how a method was applied last time, future applications of the same method to new decisions have no worked example to calibrate against |
| Artifact bloat | Medium | The alternative — writing full reasoning inline in the artifact — inflates specification documents with process detail that belongs one level removed |

### Consequences

- **Positive:**
  - Every gate verdict is traceable to an actual worked argument, not
    an assertion
  - Future gate applications have a worked precedent per method to
    follow
  - Specification artifacts (`SEMANTIC_MODEL.md`, future
    `PRIMITIVE_BLOCKS.md`, etc.) stay focused on content; process
    detail lives in the Decision Log instead
- **Negative:**
  - One more file to update per Tier 1–2 decision, alongside the EDR,
    the artifact's own Validation section, and the Decision Journal
    row
  - Risk of the four locations (artifact, EDR, Decision Journal,
    Decision Log) drifting out of sync if only some are updated

### Evolution

- The Decision Log grows by one entry per Tier 1–2 EDR going forward;
  it is never rewritten, only appended to.
- If a gate's method file is later added or changed (e.g. the missing
  `gates/methods/EMPIRICAL_ANALYSIS_METHOD.md`, flagged separately),
  existing Decision Log entries are not retroactively rewritten —
  only new entries use the updated method.

### Compliance

`DECISION_PROCESS.md` § Recording is updated to name `DECISION_LOG.md`
as the reasoning-trail record for Tier 1–2 decisions, alongside the
existing EDR and Decision Journal requirements. A future Language
Design Gate checklist pass (EDR-004) can add "Decision Log entry
written" as an explicit completion criterion; not added retroactively
to avoid re-litigating EDR-004 as part of this decision.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|--------------------------|
| Inline the full reasoning in the artifact's own Validation section | Bloats specification documents that should stay focused on the language, not the process that validated it |
| Expand the Decision Journal table to hold full reasoning per cell | The Journal's value is being scannable at a glance (one row per decision); embedding paragraphs of reasoning defeats that purpose |
| No separate record — trust the verdict table alone | Leaves every "Pass" unverifiable and gives future decisions no worked precedent to follow, per the Without It risks above |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| EDR-002 (Gate System) | The Decision Log records reasoning produced by gates EDR-002 established |
| EDR-003 (Validation Methods) | The Decision Log's per-gate subsections are worked applications of the six (soon seven) methods EDR-003 established |
| EDR-013 (Semantic Model) | First decision recorded in the Decision Log — see `gates/DECISION_LOG.md` § Semantic Model entry |
