---
phase: 02
phase_name: "semantic-model-define-identity-ownership-mutation-evaluation"
project: "Orthon Language Specification"
generated: "2026-07-27"
counts:
  decisions: 7
  lessons: 5
  patterns: 3
  surprises: 4
missing_artifacts:
  - "02-UAT.md"
---

# Phase 02 Learnings: semantic-model-define-identity-ownership-mutation-evaluation

## Decisions

### Six semantic dimensions adopted as the complete, minimal semantic contract
Identity, Ownership, Mutation, Evaluation, Visibility, and Lifetime were adopted as the full, minimal, syntax-independent semantic contract for Orthon v0.1, formally accepted via EDR-013.

**Rationale:** These six dimensions fully characterize a program's meaning; candidate 7th dimensions were tested and rejected (see Patterns/Decisions below).
**Source:** 02-02-SUMMARY.md (key-decisions)

---

### Ownership's concrete transfer syntax deferred to Phase 5
Only the semantic requirement that ownership transfer be visible is fixed in Phase 2 (`@ownership` / `$` / `move` remain undecided).

**Rationale:** Phase 2 defines semantics, not syntax; syntax choices belong to Phase 5 (Syntax Design) per the roadmap's phase separation.
**Source:** 02-02-SUMMARY.md (key-decisions, Decisions Made)

---

### IDENTITY_BASED_SAFETY.md's implicit mutability inference rejected as the foundational model
Its `.`/`!`-based implicit mutability inference was rejected as the enforcement/foundational model because it violates the Explicitness principle, but its ownership + escape-analysis reasoning was preserved as one compatible future (Phase 7) enforcement strategy.

**Rationale:** Explicit, syntactically-visible ownership transfer is a hard requirement (Semantic Invariant 6); implicit inference conflicts with it even though the underlying analysis technique is still useful.
**Source:** 02-02-SUMMARY.md (key-decisions, Decisions Made)

---

### Ownership applies wherever exclusive responsibility exists, not only "external resources"
Roughly 95% of ordinary-value code never needs to reason about ownership explicitly, because plain value semantics eliminates the question.

**Rationale:** Narrowing ownership's scope to only external resources would have been a special case; defining it generally in terms of exclusive responsibility keeps the model orthogonal and lets value semantics absorb the common case.
**Source:** 02-02-SUMMARY.md (key-decisions)

---

### Candidate 7th dimensions (Aliasing, Concurrency) rejected
Both were tested against the Minimal Core principle and found to be fully composable from the existing six dimensions, so no 7th dimension was added.

**Rationale:** Minimal Core requires that the semantic dimension set be minimal; a candidate dimension that decomposes into existing ones doesn't earn a new slot.
**Source:** 02-02-SUMMARY.md (key-decisions)

---

### New "Validation" section title chosen over the plan's example title
The plan suggested "Decision Validation Gates" as a section title, but the executor instead used "Validation" to match EDR-013's pre-existing `§ Validation` cross-references verbatim.

**Rationale:** Avoids introducing a stale/mismatched section-name reference between the EDR and the document it accepts.
**Source:** 02-02-SUMMARY.md (key-decisions, Decisions Made)

---

### Informal gate names mapped onto DECISION_VALIDATION.md's real 7-gate catalogue
02-PLAN.md's informal gate names (Correctness, Orthogonality, Learnability, Completeness, Principle Alignment) were mapped onto the actual gate catalogue (`USER_VALUE_GATE`, `LOGICAL_CONSISTENCY_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `ARCHITECTURAL_INTEGRITY_GATE`, `IMPLEMENTATION_INDEPENDENCE_GATE`, `LONG_TERM_MAINTAINABILITY_GATE`, `LLM_GENERABILITY_GATE`) rather than inventing a parallel taxonomy.

**Rationale:** The project already has one authoritative gate catalogue in `how/gates/DECISION_VALIDATION.md`; a plan-local naming scheme would have fragmented validation terminology.
**Source:** 02-02-SUMMARY.md (key-decisions, Decisions Made)

---

## Lessons

### Verification passing "8/8 truths" did not catch internal self-contradiction in a canonical example
02-VERIFICATION.md scored the phase 8/8 with no gaps, but the subsequent code review (02-REVIEW.md, CR-02) found that the Ownership section's own two canonical code examples directly contradict Semantic Invariant 6 — one example moves a named binding via bare `=` with no visible marker, the other treats the identical operation as a compile error requiring an explicit marker.

**Context:** Verification confirmed each dimension section was present and substantive but did not cross-check a section's examples against each other or against the invariant they illustrate.
**Source:** 02-VERIFICATION.md; 02-REVIEW.md (CR-02)

---

### A "15 pairwise interactions documented" completeness claim was false as written
The Cross-Dimension Consistency table's 15th row ("Lifetime ↔ All") is explicitly a cross-cutting summary, not a genuine pairwise interaction — leaving only 14 of the required 15 pairs actually analyzed, with Visibility ↔ Lifetime never addressed anywhere in the document. This survived both the write task and verification.

**Context:** Both EDR-013 and the model's own `LOGICAL_CONSISTENCY_GATE` verdict relied on the "all fifteen pairwise interactions" claim as evidence; counting the table's actual rows (rather than trusting the header's count) revealed the gap.
**Source:** 02-REVIEW.md (CR-01)

---

### EDR-to-EDR citations need index reconciliation, not just file existence checks
EDR-013 cites "EDR-011 (LLM Generability Gate)," but `INDEX.md` — the canonical journal — assigns EDR-011 to an unrelated record ("Process Inventory"). A second file genuinely about the LLM Generability Gate exists on disk under the same ID but isn't indexed at all. The collision predates Phase 2, but EDR-013 repeated the stale reference rather than catching it.

**Context:** Checking that a cited EDR ID's file exists on disk is not sufficient — the ID must also be checked against the authoritative INDEX.md to confirm it hasn't been reassigned or duplicated.
**Source:** 02-VERIFICATION.md (Notable Non-Blocking Observation); 02-REVIEW.md (CR-03)

---

### Resuming an interrupted multi-wave execution session requires verifying prior commits before continuing
A prior executor session completed Waves 1–3 (12 commits) but was interrupted before Wave 4 and before writing the Summary. The resuming session verified all 12 prior commits existed via `git log --grep="02-02"` and confirmed the target file was already complete before resuming at Wave 4, avoiding redone work.

**Context:** Without this git-log verification step, a resumed session risks either re-doing already-completed waves or skipping work that was never actually committed.
**Source:** 02-02-SUMMARY.md (Issues Encountered)

---

### Fixing a stale cross-document fact while doing unrelated terminology work is worth doing inline
While updating GLOSSARY.md with new Semantic Model terms (Task 4.2), the executor noticed two pre-existing glossary entries stated "six independent validation gates" when the actual gate catalogue in `DECISION_VALIDATION.md` has seven (missing LLM Generability Gate) — a factual inconsistency directly relevant to the terminology work already in progress, so it was fixed in the same commit.

**Context:** Small, directly-relevant factual corrections encountered mid-task are cheaper to fix immediately than to defer and re-discover later.
**Source:** 02-02-SUMMARY.md (Deviations from Plan — Auto-fixed Issues)

---

## Patterns

### Semantic dimension synthesis via source-document disposition table
Reconcile N competing/independent research documents into one accepted model via an EDR that includes an explicit source-document disposition table (merged / modified / superseded) for each source.

**When to use:** Any future phase (e.g., Phase 4 Derived Features) that must synthesize multiple prior research documents into one accepted concept — makes it traceable which parts of each source survived, were altered, or were dropped.
**Source:** 02-02-SUMMARY.md (tech-stack.patterns)

---

### Decision Validation retrospective run against a completed artifact
Apply `DECISION_VALIDATION.md`'s real 7-gate catalogue against a finished artifact as a whole, citing existing evidence sections in the document rather than re-deriving verdicts from scratch.

**When to use:** Any Wave-4-style validation task that runs formal gates after a document is otherwise complete — reuse the document's own Model/example/Source sections as gate evidence instead of duplicating analysis.
**Source:** 02-02-SUMMARY.md (tech-stack.patterns)

---

### Verify prior commits via `git log --grep` before resuming an interrupted multi-wave plan
Before continuing an interrupted execution, search git history for the plan/task identifiers already claimed complete, and confirm the target files match that state, rather than trusting session state alone.

**When to use:** Any multi-wave phase execution that spans more than one executor session — cheap insurance against duplicate or skipped work.
**Source:** 02-02-SUMMARY.md (Issues Encountered)

---

## Surprises

### A phase that scored 8/8 on verification still had 3 critical code-review findings
02-VERIFICATION.md reported "No gaps found against the phase goal" with all 8 observable truths verified, yet the subsequent 02-REVIEW.md found 3 critical, 4 warning, and 4 info issues in the same deliverables — including a false completeness claim and a self-contradicting canonical example.

**Impact:** Confirms that goal-backward verification (checking that sections/artifacts exist and are substantive) and code review (checking internal logical consistency of the content) catch different classes of defects; both are needed, and passing one does not imply passing the other.
**Source:** 02-VERIFICATION.md vs. 02-REVIEW.md

---

### A prior session's work went undetected mid-phase, requiring commit archaeology to resume safely
The Wave 4 session discovered mid-stream that Waves 1–3 (12 commits, tasks 1.1–3.4) were already fully complete from a prior interrupted session, requiring `git log` verification before it could safely skip re-doing that work.

**Impact:** Session interruption without an explicit handoff meant the resuming session had to reconstruct phase state from git history rather than reading it directly from a status artifact.
**Source:** 02-02-SUMMARY.md (Issues Encountered)

---

### A pre-existing EDR ID collision was propagated by this phase's own new EDR
The EDR-011 duplicate-ID collision (one file correctly indexed as "Process Inventory," another orphaned file genuinely about the LLM Generability Gate under the same ID but unindexed) predates Phase 2, but EDR-013 — a document newly authored in this phase — cited "EDR-011 (LLM Generability Gate)" as if it were the canonical, indexed record.

**Impact:** A pre-existing registry defect was not just left unfixed but actively re-cited as authoritative in new work, making it more entrenched; flagged for follow-up renumbering (subsequently addressed by the later "rename EDR-14 -> EDR-15" commit, per project git history).
**Source:** 02-VERIFICATION.md (Notable Non-Blocking Observation); 02-REVIEW.md (CR-03)

---

### GLOSSARY.md had accumulated unrelated structural defects discovered only via this phase's terminology work
Code review found duplicate `### Data` and `### Explicit Optimization` headings, a `## D (cont.)` section out of alphabetical order, and entries misfiled under the wrong letter sections (e.g., "Program Enricher" under `## I` instead of `## P`) — none introduced by Phase 2, all surfaced only because this phase touched the glossary for new terminology.

**Impact:** Touching a shared reference document for a narrow purpose (adding 7 new terms) exposed a broader, unrelated maintenance debt in that document that had accumulated silently across prior phases.
**Source:** 02-REVIEW.md (WR-04)

---
