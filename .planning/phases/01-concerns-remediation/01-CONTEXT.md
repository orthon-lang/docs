# Phase 1: Concerns Remediation — Context

**Gathered:** 2026-07-20 (assumptions mode)
**Status:** Ready for planning

<domain>
## Phase Boundary

Resolve every finding in CONCERNS.md — all 17 tech-debt, governance, and maintenance items (DEBT-01 through DEBT-17) — so that later architecture-dependent work (Phase 3 concept design, Phase 5 Freeze) rests on real content instead of placeholders. Pure documentation/design work — no compiler or runtime code in this repo.
</domain>

<decisions>
## Implementation Decisions

### Technical Approach — Content Depth for Empty Architecture Specs
- **D-01:** The four empty architecture specs (`IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`) shall be filled with **design specifications** (interface contracts, architectural invariants, key design decisions) rather than implementation-level detail (concrete algorithms, data structures, code). This is consistent with the documentation-only repo constraint and the pattern set by `ARCHITECTURE.md` and `EDR-010`. The depth bar: sufficient to guide Phase 3 concept design and Phase 5 Freeze validation, without drifting into Milestone 10 compiler-implementation territory.

### Technical Approach — Fitness Functions Catalogue Scope
- **D-02:** The fitness functions catalogue shall be sourced primarily from existing fitness-function patterns already latent in the codebase: `DESIGN_PRINCIPLES.md` (27 principles), `EDR-005-fitness-functions.md`, `_language-design.md` gate checklist, `EDR-004-language-design-checklist.md`. Each principle should map to at least one measurable fitness function, following the pattern of the 3 existing entries (Concept Stability, Layered Isolation, Composition Surface).

### Scope Boundaries — Template Structure (8 Sections)
- **D-03:** The actual concept template has **8 sections** (Issue, Principles, Policy Footprint, Model, Default Strategy, Alternative Strategies, Open Questions, Decision History). The DEBT-11 audit and DEBT-14 enforcement mechanism must use this 8-section structure. The "7-section" references in ROADMAP.md and CONCERNS.md are counting errors and should be corrected during this phase.

### Sequencing — Dependency Order
- **D-04:** DEBT items shall be sequenced in three waves:
  - **Wave A** (independent content creation): DEBT-01..04 (fill arch specs), DEBT-09 (expand fitness functions), DEBT-10 (expand documentation principles)
  - **Wave B** (cleanup & metadata): DEBT-05 (archive deprecated template), DEBT-06 (update EXECUTION_IMAGE refs), DEBT-07 (add date stamps), DEBT-08 (document EDR gap)
  - **Wave C** (audits & process docs): DEBT-11/14 (concept audit & enforcement), DEBT-12 (ownership assignment), DEBT-13 (link audit), DEBT-15 (glossary workflow), DEBT-16 (versioning policy), DEBT-17 (IA plan)
  
  Wave C depends on Waves A/B being in place. Within each wave, items are independent and can be parallelized.

### Quality Bar — Under-Developed Concepts
- **D-05:** The four under-developed concepts (`EQUALITY.md`, `FUNCTIONS.md`, `MUTABILITY.md`, `ALLOCATION.md`) shall be brought to **structurally complete drafts** — all 8 template sections present with real draft content. Full concept design is deferred to Phase 3. Where structural completion would require content that properly belongs to Phase 3's design work, a tracked remediation plan is acceptable per the ROADMAP success criteria.

### the Agent's Discretion
No items left to agent discretion — all assumptions were confirmed.

### Folded Todos
None — no pending todos matched Phase 1 scope.
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

- `docs/.planning/ROADMAP.md` — Phase 1 goal, success criteria, requirements
- `docs/.planning/REQUIREMENTS.md` — DEBT-01..17 requirement definitions
- `docs/.planning/codebase/CONCERNS.md` — Full concern analysis driving this phase
- `docs/.planning/codebase/STRUCTURE.md` — Directory layout and file organization
- `docs/.planning/codebase/CONVENTIONS.md` — Naming, cross-reference, template conventions
- `docs/how/templates/_concept.md` — Concept template (8 sections, not 7)
- `docs/how/architecture/ARCHITECTURE.md` — Layered architecture overview
- `docs/how/decision_records/architecture/EDR-010-layered-architecture.md` — Architecture decision
- `docs/how/decision_records/architecture/EDR-011-llm-generability-gate.md` — LLM Generability Gate
- `docs/how/decision_records/process/EDR-005-fitness-functions.md` — Fitness functions concept
- `docs/how/gates/_language-design.md` — Language design gate checklist
- `docs/what/DESIGN_PRINCIPLES.md` — 27 design principles (fitness function sources)

**Target files to be modified:**
- `docs/how/architecture/IR.md` — currently empty
- `docs/how/architecture/PARSER.md` — currently empty
- `docs/how/architecture/TYPE_SYSTEM.md` — currently empty
- `docs/how/architecture/NAME_RESOLUTION.md` — currently empty
- `docs/how/architecture/FITNESS_FUNCTIONS.md` — currently sparse (3 functions)
- `docs/how/DOCUMENTATION_PRINCIPLES.md` — currently ~25 line stub
- `docs/how/templates/_adr.md` — deprecated, to be archived
- `docs/what/concepts/EXECUTION_IMAGE.md` — superseded, refs to update
- `docs/notes/imperative-crutch-lazy-sequences.md` — contains stale EXECUTION_IMAGE ref
- `docs/notes/imperative-crutch-resource-management.md` — contains stale EXECUTION_IMAGE ref
- `docs/how/decision_records/INDEX.md` — EDR gap documentation target
- `docs/how/decision_records/process/EDR-001-edr-system.md` — EDR gap documentation target
- `docs/what/concepts/EQUALITY.md` — under-developed concept
- `docs/what/concepts/FUNCTIONS.md` — under-developed concept
- `docs/what/concepts/MUTABILITY.md` — under-developed concept
- `docs/what/concepts/ALLOCATION.md` — under-developed concept
- `docs/what/GLOSSARY.md` — glossary maintenance workflow target
- All concept files — date-stamp addition target
- `docs/.planning/ROADMAP.md` — fix "7-section" → "8-section" references

No external specs — requirements fully captured in decisions above.
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- **Template system:** `docs/how/templates/_concept.md` (8-section), `_edr.md`, `_edr-architecture.md`, `_edr-process.md` — all established and usable
- **Gate system:** `docs/how/gates/DECISION_VALIDATION.md`, `_language-design.md` — validation framework for concept design
- **EDR system:** Sequential numbering across categories, master index at `decision_records/INDEX.md`
- **Architecture pattern:** `ARCHITECTURE.md` defines layered architecture (Core Language, Standard Library, Implementation Strategy) — any new arch spec must respect these boundaries

### Established Patterns
- **Document status markers:** DRAFT banner at top of concept docs, with creation milestone, review process reference, and acceptance criteria
- **Cross-referencing:** Relative paths in markdown link syntax, "See also" sections at bottom of concept docs, affected-documents checklist in concept template
- **Layer organization:** Why → How → What → When (Golden Circle framework)
- **Naming:** `UPPER_CASE.md` for content, `_kebab-case.md` for templates, `EDR-NNN-title.md` for decisions

### Integration Points
- Phase 1 outputs feed directly into Phase 2 (process docs reference filled arch specs) and Phase 3 (concept depth audit enables design work)
- Fitness functions catalogue from DEBT-09 will be used by Phase 3's Concept Design Review gate
- Date-stamp metadata (DEBT-07) establishes freshness tracking for the entire docs lifecycle
</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches. All decisions follow from codebase evidence and confirmed assumptions.
</specifics>

<deferred>
## Deferred Ideas

None — analysis stayed within phase scope.

### Reviewed Todos (not folded)
No pending todos matched Phase 1 scope.
</deferred>
