# Phase 4: Derived Features & Decision Pipeline — Context

**Gathered:** 2026-07-27
**Status:** Ready for planning

<domain>
## Phase Boundary

Design every outstanding language feature through a rigorous pipeline — Decision Pipeline → Primitive Decomposition → Classification (Language/StdLib/External/Policy) → Concept Design Review → Validation Gates → EDR. Produces accepted concept documents in `what/concepts/`, finalized `DECISION_PIPELINE.md`, `LIBRARY_BOUNDARY.md` as summary table, and a Phase 4 aggregating EDR.

Covers ~58 concepts (22 essential-tier + 36 important-tier) plus 4 formal rejections and 3 Policy-level concepts. 54 deferrable-tier concepts receive documented deferral rationale. Anti-pattern (imperative-crutch) research is already integrated into concept docs — no separate processing needed.

Covers requirements: DERIV-01, DERIV-02, DERIV-03, CONCEPT-ESS-01, CONCEPT-IMP-01, CONCEPT-DEFER-01, CONCEPT-REJECT-01.
</domain>

<decisions>
## Implementation Decisions

### D-01: Processing Order — Essential-First by Tier
- **Decision:** Essential-tier concepts (22) processed first, then important-tier (36). Within essential tier: clean language features processed before Policy-level edge cases (ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM).
- **Sub-ordering within clean language features:** Follow ROADMAP.md priority table — Core (PATTERN_MATCHING, ERROR_HANDLING, GENERICS, OWNERSHIP, FUNCTIONS) → Control (ASYNC_AWAIT, CONCURRENCY, GENERATORS) → Data (SORTING, UNPACKING, OBJECT_INITIALIZATION, SPAN) → Meta (METAOBJECTS, LITERATE_PROGRAMMING, LLM_TOOLCHAIN). Remaining essential concepts (EQUALITY, NULL_SAFETY, TYPE_INFERENCE, etc.) placed in the family that fits best.
- **Borderline concepts** (CONTEXT_PARAMETERS, REPRESENTATION_MODIFIERS — moved to essential/ mid-cycle): processed in this phase alongside essential-tier; permission to correct Semantic Model or Primitive Blocks if analysis reveals inconsistency.
- **Anti-pattern research** (ANTIPAT-01 through ANTIPAT-10): no separate processing block. The imperative-crutch analyses are already integrated into the concept research docs they inform. The REQUIREMENTS.md requirements are effectively satisfied by the integrated findings.
- **Rationale:** Tier-first sequencing ensures the semantic bedrock is settled before building on it. ROADMAP priority within tier ensures the most architecturally impactful concepts (Core) are resolved first.

### D-02: Decision Pipeline Depth — Full Treatment per Concept
- **Decision:** Every concept (essential and important tiers) receives full treatment: aggregate all related research files → run through 10-question Decision Pipeline → run through all 7 DECISION_VALIDATION.md gates → full Concept Design Review (5-step: Idea → Problem → Minimal Solution → Principle Check → Examples → EDR) → concept-specific EDR documenting classification, primitive decomposition, and design decisions.
- **Pipeline results documented** explicitly — each concept's Q&A traceable in the pipeline application log.
- **Single aggregating EDR** for Phase 4 (following pattern of EDR-013 for Semantic Model, EDR-016 for Primitive Blocks), recording the overall acceptance of all processed concepts and their interactions.
- **Rationale:** User selected "deep design" approach. Full treatment ensures each concept is genuinely designed, not just classified. The aggregating EDR provides the global acceptance record.

### D-03: Classification Criteria — Both Semantic Uniqueness AND Compiler Dependency
- **Decision:** Classification uses two criteria checked in order:
  1. **Semantic uniqueness** (primary) — Does this feature add new semantics not expressible through composition of existing primitives? If yes → Language.
  2. **Compiler dependency** (secondary) — Must the compiler understand this feature for correct code generation? If yes → Language.
- **If neither:** Classify as Standard Library (composable from Language primitives) or External Library (domain-specific, out of scope for M1).
- **Classification recorded** in each concept's EDR as a dedicated field.
- **LIBRARY_BOUNDARY.md:** Created as a lightweight summary table derived from EDR classifications — not a design artifact, just a reference document. Satisfies DERIV-02.
- **Rationale:** Both criteria together prevent two failure modes: (a) classifying a library-worthy feature as Language just because the compiler team implements it, and (b) classifying a genuinely semantic feature as StdLib because it could theoretically be a library.

### D-04: Policy-Level Concepts — Pipeline with Policy Classification
- **Decision:** ALLOCATION.md, REGION_BASED_MEMORY_MANAGEMENT.md, and EXECUTION_PROGRAM.md run through Decision Pipeline but are classified as **Policy** (not Language/StdLib/External). Pipeline results document why they are Policy-level and where they belong in the Implementation Strategy hierarchy.
- **Destination:** Filed under `how/strategies/` area rather than `what/concepts/`.
- **Rationale:** These are implementation-policy decisions (how semantics are realized), not language features that users write code against. They belong with the strategy system.

### D-05: Rejection EDRs — Full EDR per Concept
- **Decision:** Each of the 4 concepts contradicting Orthon principles gets a full EDR:
  - PROTOTYPE.md
  - SIGNIFICANT_WHITESPACE.md
  - DYNAMIC_TYPING.md
  - CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md
- Each EDR includes: context (what the concept proposes), rationale (which Orthon principles it violates and why), alternatives considered (what Orthon uses instead), and formal rejection verdict.
- After EDR: move rejected files from `how/concepts/research/deferrable/` to `how/concepts/research/reject/`.
- **Rationale:** Formal rejection EDRs serve as permanent design records — years later, someone asking "why doesn't Orthon have prototypal inheritance?" finds the EDR with the reasoning.

### D-06: Deferrable-Tier Deferral Documentation
- **Decision:** Each of the ~54 deferrable-tier concepts gets a one-paragraph deferral rationale documented in a single organized reference (e.g., a table in `DECISION_PIPELINE.md` application log or in the aggregating EDR). Rationale states: why the concept is deferred (per SEED-001 tiering criteria), which phase could revisit it (v0.2/v0.3), and any dependency that must be resolved first.
- **Rationale:** Avoids losing rationale for ~50 decisions. A table with concept name + one-line rationale + target version is sufficient.

### the Agent's Discretion
- Exact ordering within concept families (e.g., which Core concept to process first among PATTERN_MATCHING, ERROR_HANDLING, GENERICS, OWNERSHIP).
- Whether closely related concepts share a combined EDR or get separate ones (e.g., CONCURRENCY_MODEL.md and ACTORS.md may be one EDR or two).
- Format and structure of the DECISION_PIPELINE.md application log.
- LIBRARY_BOUNDARY.md summary table format.
- Deferral documentation format (table in pipeline log vs. table in aggregating EDR).
- The content formatting of each concept's Canonical Form examples in the Design Review.

</decisions>

<specifics>
## Specific Ideas

- Follow the pattern established in Phases 2 and 3: concept-specific EDRs + single aggregating EDR (EDR-013 for Semantic Model, EDR-016 for Primitive Blocks → aggregating EDR for Phase 4).
- Some concepts are spread across multiple hypothesis files — aggregate related files before processing rather than treating each file as a separate concept.
- Essential-tier concept research files already in `how/concepts/research/essential/` serve as the raw material; the Design Review synthesizes from all related files.
- Prior decisions locked in Phases 2/3 CONTEXT.md carry forward: `@` Metadata Protocol (D-07), `emit` lazy default (D-06), blocks require explicit `return` (D-09), interior mutability/closure mutation/mut-vs-&mut (D-10 open items).
- The imperative-crutch anti-pattern research is already integrated into concept docs — no separate ANITPAT-01..10 processing pass needed.

</specifics>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Phase Definition & Requirements
- `when/ROADMAP.md` § Phase 4 — Full scope, steps, priority table, exit criteria
- `.planning/REQUIREMENTS.md` § Phase 4 — DERIV-01..03, CONCEPT-ESS-01, CONCEPT-IMP-01, CONCEPT-DEFER-01, CONCEPT-REJECT-01, ANTIPAT-01..10 requirement definitions
- `.planning/PROJECT.md` — Phase 4 summary in Active section
- `.planning/phases/03-primitive-blocks-identify-the-minimal-orthogonal-set-of-prim/03-CONTEXT.md` — Primitive set decisions (D-01..D-10), Metadata Protocol, emit semantics, block return rule
- `.planning/phases/02-semantic-model-define-identity-ownership-mutation-evaluation/02-CONTEXT.md` — Semantic model decisions (D-01..D-06), identity/ownership/mutation/evaluation/visibility/lifetime

### Process & Pipeline
- `how/process/DECISION_PIPELINE.md` — The 10-question pipeline (target document to finalize via DERIV-01)
- `how/process/DECISION_PROCESS.md` — Decision authority, tiers, recording rules
- `how/concept-design-review.md` — 5-step Concept Design Review procedure
- `how/gates/DECISION_VALIDATION.md` — 7 validation gates for design decisions
- `how/DESIGN_PRINCIPLES.md` — Locked constitution; all decisions verified against it
- `how/gates/_language-design.md` — Language design gate checklist

### Foundation Documents
- `what/PRIMITIVE_BLOCKS.md` — Accepted primitive set (EDR-016) for decomposition verification
- `what/SEMANTIC_MODEL.md` — Accepted semantic foundation (EDR-013); concepts must be consistent
- `what/GLOSSARY.md` — Terminology reference
- `how/IMPLEMENTATION_POLICIES.md` — Policy framework (for Policy-level concept classification)

### Concept Sources (by tier)
- `how/concepts/research/essential/` — All 22 essential-tier concept research files (primary source material)
- `how/concepts/research/important/` — All 36 important-tier concept research files
- `how/concepts/research/deferrable/` — All 54 deferrable-tier concepts (for deferral documentation)
- `how/concepts/research/reject/` — Destination for rejected concepts
- `.planning/seeds/SEED-001-concept-research-tier-triage.md` — Tier assignments and triage rationale

### Prior Phase EDRs
- `how/decision_records/architecture/EDR-013-semantic-model.md` — Semantic Model acceptance
- `how/decision_records/architecture/EDR-016-primitive-blocks.md` — Primitive Blocks acceptance

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `how/concepts/research/essential/` (22 files) — Existing concept research analyses, following the `_concept.md` template, ready for formal processing through the Decision Pipeline
- `how/concepts/research/important/` (36 files) — Secondary concept research, same template structure
- `how/process/DECISION_PIPELINE.md` — Already contains the 10-question list behind a DRAFT banner; needs finalization
- `what/PRIMITIVE_BLOCKS.md` — Contains complete primitive set with composition rules; serves as decomposition target
- `how/concept-design-review.md` — 5-step procedure ready for per-concept application
- Prior phase pattern (Phase 2: single SEMANTIC_MODEL.md + EDR-013; Phase 3: single PRIMITIVE_BLOCKS.md + EDR-016) — replicable as concept-specific docs + aggregating EDR

### Established Patterns
- Concept Design Review: 5-step procedure (Idea → Problem → Minimal Solution → Principle Check → Examples → EDR)
- Every consequential decision requires EDR
- DESIGN_PRINCIPLES.md is locked — all decisions verified against it
- CORE_CONCEPTS.md is the destination registry for accepted concepts
- SEMANTIC_MODEL.md and PRIMITIVE_BLOCKS.md are accepted — corrections permitted during Phase 4 if edge cases demand it

### Integration Points
- Accepted concepts move to `what/concepts/{NAME}.md`
- Classification records go into EDRs and feed LIBRARY_BOUNDARY.md summary table
- Phase 4 output feeds Phase 5 (Syntax Design) — every Language-classified concept needs concrete syntax
- Phase 6 (Cross-Cutting Review) depends on Phase 4's interaction annotations per concept
- Policy-level concepts (ALLOCATION, REGION_BASED_MEMORY_MANAGEMENT, EXECUTION_PROGRAM) route to `how/strategies/` area

</code_context>

<deferred>
## Deferred Ideas

- ~54 deferrable-tier concepts explicitly deferred to v0.2/v0.3 with documented rationale per concept.
- Literate Programming feature (listed in PROJECT.md scope) — if classified External, documented as out of scope for M1.
- LLM Toolchain components (Schema Provider, Code Completer) — lighter gate track per ROADMAP.md; their timing relative to concept design is a planning-level decision.
- "Orthon for LLM" research items beyond the LLM Generability Gate — deferred to post-Freeze LLM Toolchain work.

</deferred>

---

*Phase: 04-derived-features-and-decision-pipeline*
