# Roadmap: Orthon Language Specification

## Overview

This roadmap takes the v0.1 specification from its current state — a strong
vision layer, 22 concept documents (18 still `DRAFT`), and an undefined
acceptance process — through to a frozen, self-consistent v0.1 language
specification. It follows the natural dependency order surfaced in
`.planning/codebase/CONCERNS.md` and the existing `TODO.md` milestone
breakdown: first fix the *process* gap that would otherwise stall concept
review (Phase 1), clear the flagged architecture/tech-debt blockers (Phase 2),
do the bulk of language-concept design and anti-pattern research and run it
through the review gate (Phase 3), settle the smaller cross-cutting
Interaction Matrix question (Phase 4), and finally settle the "Orthon"
naming question and freeze the spec (Phase 5). Every
phase in this roadmap is documentation/design work — there is no compiler or
runtime in this repository.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundations, Process & Vision** - Define the concept acceptance gate, review process, and requirement-tracking model; review the Why-layer documents (Manifesto, Vision, Zen)
- [ ] **Phase 2: Concerns Remediation** - Fill the four empty architecture specs and resolve the tech-debt items flagged in CONCERNS.md
- [ ] **Phase 3: Language Inventory, Anti-Pattern Research & Concept Design Review** - Design the 13 outstanding language concepts, complete the 10 anti-pattern research topics, and run all 22 concepts through the Concept Design Review gate
- [ ] **Phase 4: Cross-Cutting Review** - Finalize the Interaction Matrix format for documenting concept interactions
- [ ] **Phase 5: Freeze & Naming** - Settle the "Orthon" naming question, resolve all outstanding validation issues, verify consistency, tag the version, and define whitepaper/launch strategy

## Phase Details

### Phase 1: Foundations, Process & Vision
**Goal**: A documented, self-imposed process exists for accepting concepts and tracking work, and the foundational Why-layer documents have been critically reviewed — so that Milestone 1-2 content work (Phase 3) has a defined gate to pass through instead of stalling on undefined process, per CONCERNS.md's "No Acceptance Gate for DRAFT Concepts" and "Concept Design Review Process Undefined" findings.
**Depends on**: Nothing (first phase)
**Requirements**: PROC-01, PROC-02, PROC-03, PROC-04, VISION-01, VISION-02, VISION-03
**Success Criteria** (what must be TRUE):
  1. `docs/how/concept-design-review.md` documents a complete DRAFT → Accepted transition process — owner, acceptance criteria, and EDR linkage — and is no longer a stub.
  2. Every line item from the former `TODO.md` milestone breakdown (Milestones 0-7) is represented as a tracked GSD requirement with an explicit status, replacing freeform TODO checkboxes as the tracking model for solo authorship.
  3. A template-compliance audit checklist for the 7-section `_concept.md` template exists and is referenced from the Concept Design Review process doc, so Phase 3's review gate has a concrete checklist to apply.
  4. `why/MANIFESTO.md`, `why/VISION.md`, and `why/ZEN.md` each show evidence of review against `why/WORKING_BACKWARDS.md` rationale (a review note, EDR, or committed edits resulting from the review).
  5. `docs/notes/criteria-based-algorithm-selection.md` carries an explicit validation verdict (accepted / revised / rejected) rather than remaining an open research question.
  6. A dedicated research pass into what makes a programming language LLM-native/LLM-generable — informed by `why/VISION.md`'s "Designed for the LLM Era" pillar, the LLM Generability Gate (`how/gates/_language-design.md`, `how/decision_records/architecture/EDR-011-llm-generability-gate.md`), and the existing `docs/notes/llm-generability-gate.md` / `docs/notes/language-llm-comparison.md` analyses — produces a written shortlist of concrete candidate concepts, design patterns, or process refinements worth exploring to strengthen Orthon's LLM-native design, at `docs/notes/llm-native-concept-shortlist.md`. This shortlist is a direct input to Phase 3's concept design and prioritization work across the 13 outstanding language concepts (CONCEPT-01..13).
**Plans**: TBD

### Phase 2: Concerns Remediation
**Goal**: Every finding in CONCERNS.md — not only the critical and high-severity blockers, but every tech-debt, governance, and maintenance item on its Summary Table — is resolved or has an explicit, documented remediation, so later architecture-dependent work (Freeze, and any concept design that references parsing/type/name-resolution behavior) rests on real content instead of placeholders, and no unresolved concern is silently carried past this phase.
**Depends on**: Phase 1
**Requirements**: DEBT-01, DEBT-02, DEBT-03, DEBT-04, DEBT-05, DEBT-06, DEBT-07, DEBT-08, DEBT-09, DEBT-10, DEBT-11, DEBT-12, DEBT-13, DEBT-14, DEBT-15, DEBT-16, DEBT-17
**Success Criteria** (what must be TRUE):
  1. `docs/how/architecture/IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, and `NAME_RESOLUTION.md` are each filled with a real design (intermediate representation format, parsing/grammar strategy, type inference/checking rules, and scoping/symbol-resolution rules respectively) — none remain at 0 lines.
  2. `docs/how/templates/_adr.md` is archived or removed, and no live document references it as the current template.
  3. Zero references to the superseded `EXECUTION_IMAGE.md` remain in active documents (e.g., `docs/notes/imperative-crutch-lazy-sequences.md`, `imperative-crutch-resource-management.md`); all such cross-references point to `EXECUTION_PROGRAM.md`, and `EXECUTION_IMAGE.md` itself is archived.
  4. Concept and architecture documents carry date-stamp/freshness metadata (e.g., a `last_updated` field), addressing the "only 2 files have date stamps" finding.
  5. The EDR-008/009 numbering gap and TDR→EDR migration are documented in `how/decision_records/INDEX.md` (or EDR-001); `docs/how/architecture/FITNESS_FUNCTIONS.md` contains a full fitness-functions catalogue (not just 2 examples); `docs/how/DOCUMENTATION_PRINCIPLES.md` is expanded beyond its current template stub.
  6. All 22 concept documents have been audited against the `_concept.md` 7-section template; the under-developed 25-35 line drafts (`EQUALITY.md`, `FUNCTIONS.md`, `MUTABILITY.md`, `ALLOCATION.md`) are either brought to compliant depth or carry an explicit, tracked remediation plan, and the audit results (a produced gap list) demonstrate the template-compliance enforcement mechanism is actually applied, not just defined.
  7. Every tracked TODO/requirement item has an assigned owner and target-completion metadata, replacing unowned freeform checkboxes — resolving the "unclear milestone ownership" finding.
  8. A repository-wide cross-reference / "See also" link audit has been performed, all broken or superseded links found are fixed, and a link-validation approach for future changes is documented.
  9. A glossary maintenance workflow (how/when new terms get registered in `GLOSSARY.md`) and a versioning/change-log policy for large monolithic documents (`EXECUTION_PROGRAM.md`, `GLOSSARY.md`, `ARCHITECTURE.md`, `DESIGN_PRINCIPLES.md`, `AGENTS.md`) are both documented, and a documentation growth / information-architecture plan exists for scaling `what/concepts/` and future stdlib/tooling directories.
**Plans**: TBD

### Phase 3: Language Inventory, Anti-Pattern Research & Concept Design Review
**Goal**: Every outstanding language concept is fully designed — informed by anti-pattern research into imperative crutches and Phase 1's LLM-native concept shortlist (`docs/notes/llm-native-concept-shortlist.md`) — and every concept document in the repository has passed a formal Concept Design Review, leaving zero `DRAFT` headers in `docs/what/concepts/`.
**Depends on**: Phase 1 (requires the acceptance gate and review process from PROC-01/02)
**Requirements**: CONCEPT-01, CONCEPT-02, CONCEPT-03, CONCEPT-04, CONCEPT-05, CONCEPT-06, CONCEPT-07, CONCEPT-08, CONCEPT-09, CONCEPT-10, CONCEPT-11, CONCEPT-12, CONCEPT-13, ANTIPAT-01, ANTIPAT-02, ANTIPAT-03, ANTIPAT-04, ANTIPAT-05, ANTIPAT-06, ANTIPAT-07, ANTIPAT-08, ANTIPAT-09, ANTIPAT-10, ANTIPAT-11
**Success Criteria** (what must be TRUE):
  1. All 13 concept documents in scope (`PATTERN_MATCHING.md`, `ERROR_HANDLING.md`, `OWNERSHIP.md`, `GENERICS.md`, `ASYNC_AWAIT.md`, `OBJECT_INITIALIZATION.md`, `CONCURRENCY.md`, `LITERATE_PROGRAMMING.md`, `SORTING.md`, `UNPACKING.md`, `GENERATORS.md`, `METAOBJECTS.md`, `SPAN.md`) contain real content in all 7 template sections (Issue, Principles, Model, Default Strategy, Alternative Strategies, Open Questions, Decision History) — none remain thin stubs.
  2. All 10 `docs/notes/imperative-crutch-*.md` research topics are completed with documented findings and explicit implications for Orthon's concept designs, and those implications are reflected in the relevant concept documents (e.g., sorting research reflected in `SORTING.md`, null-handling research reflected in `ERROR_HANDLING.md` / `CORE_CONCEPTS.md`).
  3. Zero `⚠️ DRAFT` headers remain across all 22 files in `docs/what/concepts/`; every file has an explicit Accepted, Deferred, or Rejected status.
  4. Every Accepted concept has a corresponding EDR in `how/decision_records/architecture/` recording the acceptance rationale, satisfying the Concept Design Review gate defined in Phase 1.
**Plans**: TBD

### Phase 4: Cross-Cutting Review
**Goal**: The remaining smaller cross-cutting question — how feature interactions are documented — is settled once the bulk of concept design has stabilized, so Freeze isn't blocked on an open-ended research note.
**Depends on**: Phase 3
**Requirements**: XCUT-01
**Success Criteria** (what must be TRUE):
  1. `docs/notes/interaction-matrix-format.md` research is finalized into a concrete, adopted Interaction Matrix format (no longer an open investigation), with a clear statement of where/how it will be applied to concept interactions.
**Plans**: TBD

### Phase 5: Freeze & Naming
**Goal**: Orthon v0.1 coheres as a complete, self-consistent language specification — every concept Accepted, architecture specs filled, cross-references valid, the "Orthon" name settled — and is versioned with a whitepaper and launch plan ready to execute.
**Depends on**: Phase 1, Phase 2, Phase 3, Phase 4 (all — Freeze validation requires the process gate, filled architecture specs, fully reviewed concepts, and settled cross-cutting decisions)
**Requirements**: NAME-01, FREEZE-01, FREEZE-02, FREEZE-03, FREEZE-04, FREEZE-05, FREEZE-06, FREEZE-07
**Success Criteria** (what must be TRUE):
  1. A documented decision (EDR or equivalent record) confirms "Orthon" as the final project name, or names a replacement with rationale — resolving Milestone 6.
  2. Every open validation issue tracked across the project (concept review notes, CONCERNS.md items, Open Questions sections) has either been resolved or carries an explicit, documented deferral rationale.
  3. A dedicated final review record confirms the specification is self-consistent and complete — no contradictions between concept documents, architecture specs, and design principles.
  4. A link-check of all "See also" and cross-reference links across the repository reports zero dangling references.
  5. A version tag (e.g., "Orthon 0.1 Specification") exists, marking the frozen spec.
  6. Whitepaper strategy in `docs/notes/whitepaper-strategy.md` has been revisited with at least one whitepaper commissioned or drafted, and a launch/announcement strategy (audience, channels, timing) is documented.
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|-----------------|--------|-----------|
| 1. Foundations, Process & Vision | 0/TBD | Not started | - |
| 2. Concerns Remediation | 0/TBD | Not started | - |
| 3. Language Inventory, Anti-Pattern Research & Concept Design Review | 0/TBD | Not started | - |
| 4. Cross-Cutting Review | 0/TBD | Not started | - |
| 5. Freeze & Naming | 0/TBD | Not started | - |
