# EDR-087: Syntax Acceptance Process — `how/syntax/` Working Entity

**Status:** Accepted

**Date:** 2026-08-15

**Category:** Process

**Scope:** Project

---

### Context

Phase 5 (Syntax Design) works from accepted concepts, but the deferred
**syntax decisions** recorded across research docs had no unified home and
no acceptance process:

- **Scattered.** Syntax hypotheses lived in three places with no index:
  the `how/concepts/research/` root (`TYPE_ANNOTATION_SYNTAX.md`,
  `SHADOWING_SYNTAX.md`) and the `important/` tier (`FUNCTION_ARGUMENT_SYNTAX.md`,
  `FUNCTION_RETURN_SYNTAX.md`, `FUNCTION_RETURNING_FUNCTION.md`,
  `BOUNDS_IN_ANGLE_BRACKETS.md`). `research/README.md` did not list them, so
  they were discoverable only by browsing the directory.
- **Loss risk.** ROADMAP Phase 5's exit criteria are per-concept ("every
  language concept has defined syntax"); nothing forced a sweep of open
  syntax hypotheses. A deferred decision (for example `: Type` from
  `EDR-027`/`EDR-074`) could be silently skipped.
- **No unified input flow.** The intended flow — semantic research →
  accepted concept (EDR) → deferred syntax decision → accepted syntax
  (EDR) — was implicit, not formalized.
- **Gates already exist.** `DECISION_VALIDATION.md` § Gate Selection already
  defines a "Syntax change" row, and `what/SYNTAX.md` defines five Syntax
  Principles. No parallel gate system was needed — only a thin process that
  reuses them.

### Decision

1. **`how/syntax/` is the syntax-hypothesis inbox** (How layer): one file
   per open syntax question, indexed in `how/syntax/README.md` with a
   **decision queue** (deferral source, coupling, status). All
   syntax-hypothesis documents are moved here; coupled questions living in
   concept research docs are registered by pointer, not relocated.
2. **`what/syntax/` is the accepted-syntax home** (What layer): one file per
   accepted syntax decision. **`what/SYNTAX.md` becomes a hub only** —
   Syntax Principles plus pointers — to avoid an unmaintainable single file.
3. **`how/SYNTAX_PIPELINE.md` formalizes acceptance**, reusing existing
   machinery: Decision Pipeline (pre-filter) → **Coupling & Overload Check**
   (the one syntax-specific gate — symbol/keyword collision sweep across all
   concepts) → 5 Syntax Principles → Gate Selection "Syntax change" row
   (`LOGICAL_CONSISTENCY`, `CONCEPTUAL_SIMPLICITY`, `ARCHITECTURAL_INTEGRITY`,
   `LLM_GENERABILITY`; optional `USER_VALUE`, `IMPLEMENTATION_INDEPENDENCE`,
   `LONG_TERM_MAINTAINABILITY`) → relevant `_language-design.md` criteria →
   Architecture EDR → `what/syntax/` record → hub + "Resolved" mark.
4. **Unified Phase 5 input flow**:
   `research semantic concepts → EDR + concepts → deferred syntax decisions → EDR + resolved syntax`.
5. **ROADMAP Phase 5** runs in two passes: first **inventory and resolve all
   open syntax hypotheses in `how/syntax/`** (the deferred-items pass), then
   produce syntax per accepted concept. Exit criteria include "all open
   syntax hypotheses in `how/syntax/` resolved and folded into `SYNTAX.md`".
6. **Per syntax decision:** Architecture EDR + `what/syntax/` record + hub
   pointer; the hypothesis doc is marked **"Resolved — see `SYNTAX.md` § …"**
   per the `RANGE_STEP.md` precedent (reasoning trail remains provenance).
7. **Pipeline runs are provenance, not EDRs.** A Decision Pipeline run on a
   hypothesis is recorded as a **«Decision Pipeline Run»** section inside the
   hypothesis document; the EDR records the eventual decision.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Deferred syntax decisions silently dropped in Phase 5 | High | A concept's syntax decided ad hoc without the recorded comparison (`x: Type` vs `Type x`, `let`/`var`, `->` overload) |
| Syntax hypotheses undiscoverable | Medium | Files in three unindexed locations; Phase 5 agents never open them |
| `what/SYNTAX.md` grows unbounded | Medium | A single huge reference file becomes unmaintainable and hard to review |
| Ad-hoc syntax acceptance | Medium | Syntax accepted without the existing gates; overloaded symbols (`,` `:` `<>` `->`) recur |

### Consequences

- **Positive:**
  - One inventory (`how/syntax/` decision queue) → nothing deferred is lost.
  - Clean layer mirror: `how/syntax/` (hypotheses) → `what/syntax/` + hub (accepted), parallel to `how/concepts/research/` → `what/concepts/`.
  - Reuses existing gates; no parallel gate system to maintain.
  - Accepted syntax gets durable records (`what/syntax/*`) with EDR provenance.
  - Phase 5 has an explicit first pass over deferred items, then per-concept work.
- **Negative:**
  - New directory + two READMEs + a pipeline doc to keep current (bit-rot risk mitigated by the decision-queue index and the Phase 5 exit criterion).
  - Six hypothesis documents were relocated; relative links in them and their referrers needed updating.
  - `what/SYNTAX.md` must be rewritten as a hub; readers looking for full Invocation/Range/Generics syntax now follow a pointer.

### Evolution

Deprecate or simplify when Phase 5 completes (all hypotheses resolved): the
inbox becomes an archive, `what/syntax/` remains the canonical syntax home,
and `SYNTAX_PIPELINE.md` is only re-invoked for post-v1.0 syntax changes.
If the number of open hypotheses stays small after Phase 5, the queue could
collapse into `SYNTAX.md`; keep the split only while it earns its keep.

### Compliance

1. Every open syntax question is registered in `how/syntax/README.md` § Decision Queue.
2. Every accepted syntax decision has an Architecture EDR and a `what/syntax/` record, and the hypothesis is marked Resolved.
3. `what/SYNTAX.md` contains only principles + pointers, never full per-construct syntax.
4. ROADMAP Phase 5 exit criteria include the deferred-items resolution.
5. `PROCESS_INVENTORY.md` lists the Syntax Pipeline.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Record the pipeline run as an EDR | EDRs record accepted decisions; a pipeline run on a deferred hypothesis has no binding verdict. The run belongs in the hypothesis doc as provenance. |
| Keep syntax hypotheses in `research/` root | They were unindexed and undiscoverable; mixing syntax with semantic research confuses both inboxes. |
| Fold everything into `what/SYNTAX.md` | Single-file hub grows unbounded and mixes open/decided state; contradicts one-file-one-topic. |
| Invent dedicated syntax gates | `DECISION_VALIDATION.md` § Gate Selection already covers "Syntax change"; a parallel gate system adds overhead without new coverage. |
| Move concept-tied syntax docs (`CLOSURE_CAPTURE.md`, `TRAIT_BLANKET_IMPLEMENTATION.md`) into `how/syntax/` | They are concept research with a syntax dimension; relocating would break tier status. Registered by pointer instead. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | The user (Phase 5 designer/agent) needs a single inventory of deferred syntax decisions and a defined acceptance path — otherwise decisions are dropped or decided ad hoc. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | The split is consistent: hypotheses (How) → accepted (What); hub holds only pointers; queue statuses are disjoint (Open / Resolved / Rejected). No contradiction with the existing concept pipeline. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Reuses existing gates; adds exactly one syntax-specific step (Coupling & Overload Check) and one inventory — the minimal structure that prevents loss. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | One-sentence test: "deferred syntax is inventoried in `how/syntax/`, accepted syntax lands in `what/syntax/`, and the hub links both." No "but/except". The Phase 5 exit criterion keeps the queue from bit-rotting. |

**Gates not applied:**
| Gate | Rationale |
|------|-----------|
| `ARCHITECTURAL_INTEGRITY_GATE` | Process decision; does not change language architecture. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Process decision about project operations, not implementation strategies. |
| `LLM_GENERABILITY_GATE` | Process decision produces no code. |

**Detailed reasoning:** See `DECISION_LOG.md` § Entry: Syntax Acceptance Process (EDR-087) for the per-gate reasoning trail.
