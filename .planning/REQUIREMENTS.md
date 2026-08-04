# Requirements: Orthon Language Specification

**Defined:** 2026-07-20
**Core Value:** A complete, self-consistent Orthon v0.1 specification — every concept accepted (no DRAFT), core architecture specs filled in, all cross-references valid, and a final review confirming the result coheres as a real language specification.
**Ownership:** All requirements owned by **Solo Author** unless otherwise noted.

## v1 Requirements

Requirements for the v0.1 specification. Each maps to roadmap phases (the 8-phase pipeline in `when/ROADMAP.md`, tracked here as GSD Phases 1, 1.1, 2-8).

### Concerns Remediation (CONCERNS.md blockers & tech debt) — Phase 1, COMPLETE

- [x] **DEBT-01**: `docs/how/architecture/IR.md` filled with intermediate representation design
- [x] **DEBT-02**: `docs/how/architecture/PARSER.md` filled with parsing strategy / grammar design
- [x] **DEBT-03**: `docs/how/architecture/TYPE_SYSTEM.md` filled with type inference/checking design
- [x] **DEBT-04**: `docs/how/architecture/NAME_RESOLUTION.md` filled with scoping/symbol resolution design
- [x] **DEBT-05**: Deprecated `docs/how/templates/_adr.md` archived or removed; references cleaned up
- [x] **DEBT-06**: Superseded `EXECUTION_IMAGE.md` references updated to point to `EXECUTION_PROGRAM.md`; concept archived
- [x] **DEBT-07**: Date-stamp / freshness metadata added to concept and architecture documents
- [x] **DEBT-08**: EDR numbering gap (008-009) and TDR→EDR migration rationale documented
- [x] **DEBT-09**: `docs/how/architecture/FITNESS_FUNCTIONS.md` filled with a full fitness-functions catalogue
- [x] **DEBT-10**: `docs/how/DOCUMENTATION_PRINCIPLES.md` expanded beyond the current template stub
- [x] **DEBT-11**: Concept-document depth audited against the `_concept.md` 8-section template across all 22 concept files; under-developed drafts brought to template-compliant depth
- [x] **DEBT-12**: Ownership and target-completion metadata assigned to every tracked TODO/requirement item
- [x] **DEBT-13**: Cross-reference / "See also" link audit performed across the repository; broken or superseded links fixed
- [x] **DEBT-14**: Template-compliance enforcement mechanism applied — audit run against all concept documents, gap list produced
- [x] **DEBT-15**: Glossary maintenance workflow documented
- [x] **DEBT-16**: Versioning / change-log policy documented for large monolithic documents
- [x] **DEBT-17**: Documentation growth / information-architecture plan documented

**Status:** All 17 executed and verified 2026-07-20 (`.planning/phases/01-concerns-remediation/01-01-SUMMARY.md`).

### Foundation Completion (resolves CONCERNS.md governance/process gaps) — Phase 1.1 ✅

- [x] **PROC-01**: Concept acceptance gate defined — documents the DRAFT → Accepted transition (owner, criteria, EDR linkage)
- [x] **PROC-02**: Concept Design Review process fully documented and collapsed from its former 12-step procedure to a 5-step pipeline for solo authorship (`docs/how/concept-design-review.md`)
- [x] **PROC-03**: Milestone/TODO tracking model defined for solo authorship — every TODO.md item converted into a tracked GSD requirement with explicit status
- [x] **PROC-04**: Template-compliance process defined for the 8-section concept template (`_concept.md`) — audit checklist or equivalent
- [x] **PROC-05**: `docs/how/process/DECISION_PROCESS.md` (one-page decision authority) finalized from DRAFT to Accepted, referenced from `AGENTS.md`/`README.md`, alongside a finalized `docs/how/process/DECISION_PIPELINE.md` (10-question feature pipeline)
- [x] **VISION-01**: Manifesto, Vision, and Zen reviewed against Working Backwards rationale, and consolidated into a compact canonical `VISION.md` (Vision · Mission · Non-goals · Audience · Success Criteria)
- [x] **VISION-02**: Criteria-based algorithm selection idea validated (`docs/notes/criteria-based-algorithm-selection.md`)
- [x] **VISION-03**: LLM-native language research completed — produces a written shortlist of concrete concepts/ideas to strengthen Orthon's LLM-native design, feeding Phase 4's concept design and prioritization work
- [x] **PRINC-01**: `docs/how/DESIGN_PRINCIPLES.md` declared closed for modification — any change requires a Tier 1 EDR
- [x] **FITNESS-01**: Throughput fitness functions added to `FITNESS_FUNCTIONS.md` (concepts-in-flight, oldest-open-proposal age), per `docs/notes/pipeline-throughput-gaps.md`

### Semantic Model — Phase 2

- [x] **SEM-01**: `what/SEMANTIC_MODEL.md` synthesizes the 10 essential-tier source files identified as Semantic-Model raw material (`how/concepts/research/essential/DATA_MODEL.md`, `OWNERSHIP.md`, `OWNERSHIP_METAPROPERTY.md`, `OWNERSHIP_TRANSFER_OPERATOR.md`, `MUTABILITY.md`, `VALUE_SEMANTICS.md`, `IDENTITY_BASED_SAFETY.md`, `VISIBILITY_AND_ENCAPSULATION.md`, `SCOPED_RESOURCE_LIFECYCLE.md`, `EXPRESSION_ORIENTED_LANGUAGE.md`) into all 6 semantic dimensions (Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime) as one coherent whole, replacing the current DRAFT placeholder
- [x] **SEM-02**: Cross-dimension conflicts across the 10 source files checked and resolved; validated against every Design Principle
- [x] **SEM-03**: EDR accepting the Semantic Model, recording how each of the 10 source files' proposals were merged, modified, or superseded

### Primitive Blocks — Phase 3 ✅

- [x] **PRIM-01**: `what/PRIMITIVE_BLOCKS.md` synthesizes the 10 essential-tier source files identified as Primitive-Block raw material (`how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md`, `EXCLUSIVE_DECLARATIONS.md`, `STRUCT_AS_VALUE_TYPE.md`, `CLASS_WITH_ACT.md`, `ACT_AS_FUNCTION.md`, `FUNCTIONS.md`, `FINAL_BY_DEFAULT.md`, `NAMESPACES.md`, `DELEGATE.md`, `COMPOSITION_OVER_INHERITANCE.md`) into the minimal orthogonal primitive set plus composition rules, replacing the current DRAFT placeholder
- [x] **PRIM-02**: Every concept research doc across all tiers (`how/concepts/research/{essential,important,deferrable,reject}/`, ~132 files) verified to decompose onto the primitive set; set confirmed minimal
- [x] **PRIM-03**: EDR accepting the Primitive Blocks set, recording how each of the 10 source files' proposals were merged, modified, or superseded

### Derived Features & Decision Pipeline — Phase 4

- [ ] **DERIV-01**: `how/process/DECISION_PIPELINE.md` (10-question pipeline) finalized and applied to every outstanding concept
- [x] **CONCEPT-ESS-01**: The 22 essential-tier concepts not consumed by Phase 2 or Phase 3 pass the Decision Pipeline first, per SEED-001's essential-first sequencing, each reaching EDR-accepted or explicitly-rejected status — 17 core concepts (`how/concepts/research/essential/AST_MACROS.md`, `COMPILER_AS_STATIC_ANALYZER.md`, `COMPILE_TIME_EXECUTION.md`, `COMPOSABLE_COLLECTION_OPS.md`, `CONCURRENCY_MODEL.md`, `EQUALITY.md`, `ERROR_HANDLING.md`, `ERROR_UNION.md`, `GENERICS.md`, `ITERATOR_PROTOCOL.md`, `LAZY_SEQUENCE_GENERATORS.md`, `NULL_SAFETY.md`, `PATTERN_MATCHING.md`, `PATTERN_MATCHING_DISPATCH.md`, `TRAITS.md`, `TYPE_INFERENCE.md`, `TYPE_LEVEL_NULL_SAFETY.md`), plus 3 Policy-level pocket concepts processed here pending relocation (`ALLOCATION.md`, `REGION_BASED_MEMORY_MANAGEMENT.md`, `EXECUTION_PROGRAM.md` — see `.planning/todos/pending/move-policy-level-essential-concepts-out-of-pipeline.md`), plus 2 concepts a concurrent task moved from `important/` into `essential/` mid-cycle without a Phase assignment (`CONTEXT_PARAMETERS.md`, `REPRESENTATION_MODIFIERS.md` — commit `bf0f10a`; still pending a Phase 2 vs. Phase 3 vs. Phase 4 call, tracked here as unclassified rather than guessed)
- [x] **CONCEPT-IMP-01**: All important-tier concepts (`how/concepts/research/important/`, 36 files as of 2026-07-26) pass the Decision Pipeline as a second pass, after CONCEPT-ESS-01 completes. One borderline file flagged for a future separate judgment call rather than moved: `DECLARATION_BY_ASSIGNMENT.md` is arguably Phase 5 (Syntax) material
- [ ] **CONCEPT-DEFER-01**: All deferrable-tier concepts (`how/concepts/research/deferrable/`, 54 files as of 2026-07-26) explicitly deferred to v0.2/v0.3 with a documented one-paragraph rationale per concept, except the 4 files covered by CONCEPT-REJECT-01
- [ ] **CONCEPT-REJECT-01**: The 4 concepts flagged in SEED-001 as contradicting Orthon's stated principles — `PROTOTYPE.md`, `SIGNIFICANT_WHITESPACE.md`, `DYNAMIC_TYPING.md`, `CLASS_OR_STRUCTURE_AS_PRIMARY_COMPOSITION.md` (currently in `how/concepts/research/deferrable/`) — evaluated for formal rejection via EDR; rejected files moved to `how/concepts/research/reject/`
- [ ] **ANTIPAT-01**: Collections & Loops anti-pattern research completed
- [ ] **ANTIPAT-02**: Null handling anti-pattern research completed
- [ ] **ANTIPAT-03**: Resource management anti-pattern research completed
- [ ] **ANTIPAT-04**: Collection initialization anti-pattern research completed
- [ ] **ANTIPAT-05**: Type checking & casting anti-pattern research completed
- [ ] **ANTIPAT-06**: Date/time API anti-pattern research completed
- [ ] **ANTIPAT-07**: Lazy sequences & generators anti-pattern research completed
- [ ] **ANTIPAT-08**: Multi-key sorting anti-pattern research completed
- [ ] **ANTIPAT-09**: Metaprogramming & reflection anti-pattern research completed
- [ ] **ANTIPAT-10**: Serialization anti-pattern research completed
- [x] **DERIV-02**: `what/LIBRARY_BOUNDARY.md` documents language-vs-stdlib-vs-external classification for every feature
- [x] **DERIV-03**: Every "Language"-classified feature has an accepted EDR and moves to `what/concepts/`; zero DRAFT headers remain in `how/concepts/research/` (supersedes former ANTIPAT-11)

### Syntax Design — Phase 5

- [ ] **SYNTAX-01**: Syntax principles established (one concept → one syntax, one symbol → one meaning, no significant whitespace, named-over-symbolic)
- [ ] **SYNTAX-02**: `what/SYNTAX.md` documents syntax for every accepted Language concept, all canonical forms shown together
- [ ] **SYNTAX-03**: Syntax validated — no ambiguous/context-dependent parses, LLM generability check, human readability check
- [ ] **SYNTAX-04**: `how/architecture/PARSER.md` updated with concrete grammar

### Cross-cutting Review — Phase 6

- [ ] **XCUT-01**: Interaction Matrix format investigated, finalized, and applied — full pairwise analysis in `what/CROSS_CUTTING.md`
- [ ] **XCUT-02**: `what/CONFLICT_REGISTRY.md` records all concept-boundary conflicts; all resolved or deferred with EDR rationale

### Execution & Optimization Model — Phase 7

- [ ] **EXEC-01**: `what/EXECUTION_MODEL.md` defines execution semantics; Execution Program concept formalized via EDR
- [ ] **EXEC-02**: `what/OPTIMIZATION_MODEL.md` catalogues and classifies every optimisation (Optimisation / Semantics / Strategy-dependent)
- [ ] **EXEC-03**: All Implementation Strategies (Default, Embedded, High-Performance, LLM) reconciled with the Execution/Optimization Model

### Evolution Model & Freeze — Phase 8

- [ ] **EVOL-01**: `how/EVOLUTION_MODEL.md` documents versioning, deprecation, experimental/feature-gate, and breaking-change policy, approved via EDR
- [ ] **SPEC-01**: Global consistency review passed (uniformity, no special cases, global minimality); canonical `SPEC.md` and machine-readable `SCHEMA.md` produced
- [ ] **VALID-01**: Reference examples/programs written; LLM generation tests and schema round-trip tests pass; friction points documented with new EDRs
- [ ] **NAME-01**: "Orthon" verified as the final name, or replacement decided
- [ ] **FREEZE-01**: All validation issues resolved or explicitly deferred with documented rationale
- [ ] **FREEZE-02**: Specification verified self-consistent and complete
- [ ] **FREEZE-03**: All cross-references ("See also" links etc.) validated — no dangling links
- [ ] **FREEZE-04**: Dedicated final review confirms Orthon v0.1 coheres as a complete language specification
- [ ] **FREEZE-05**: Version tagged (e.g., "Orthon 0.1 Specification")
- [ ] **FREEZE-06**: Whitepaper strategy revisited; whitepaper(s) commissioned/drafted
- [ ] **FREEZE-07**: Launch/announcement strategy defined (audience, channels, timing)

### Onboarding Material — Phase 9

- [ ] **ONBOARD-01**: Tutorial covering every accepted concept (`docs/tutorial/GETTING_STARTED.md`)
- [ ] **ONBOARD-02**: Language tour in learnability order (`docs/tutorial/LANGUAGE_TOUR.md`)
- [ ] **ONBOARD-03**: Cookbook with idiomatic solutions for at least 5 common programming tasks (`docs/cookbook/`)
- [ ] **ONBOARD-04**: Cross-reference consistency check — all examples consistent with frozen `SPEC.md`; no undefined terms

## v2 Requirements

Deferred to post-Freeze (Milestones 8-9 and LLM tooling). Tracked but not in current roadmap.

### Milestone 8 — Standard Library & FFI

- **STDLIB-01**: Standard library architecture designed (collections, I/O, formatting)
- **STDLIB-02**: FFI & interoperability designed (C ABI, embedding API)
- **STDLIB-03**: Standard library whitepaper written, if stdlib design introduces novel concepts

### Milestone 9 — Build System & Tooling

- **BUILD-01**: Build system & package manager designed
- **BUILD-02**: Developer tooling designed (formatter, linter, LSP)
- **BUILD-03**: Build-system whitepaper benefit evaluated

### Orthon for LLM (beyond the already-shipped LLM Generability Gate, EDR-011)

- **LLMTOOL-01**: LLM Toolchain implemented (Schema Provider, Code Completer, Code Generator, Static Analyser)
- **LLMTOOL-02**: ML-friendly grammar research (tokenizer alignment, ambiguity metrics)
- **LLMTOOL-03**: Common LLM error-pattern research across languages
- **LLMTOOL-04**: Literate Programming as LLM-native interface designed
- **LLMTOOL-05**: Opinionated Code Generation mode decided (lint-level constraints for LLM output)
- **LLMTOOL-06**: Structured error format for LLM consumption decided (JSON schema)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Milestone 8 — Standard Library & FFI implementation/design | Deferred until v0.1 spec is locked at Freeze |
| Milestone 9 — Build System & Tooling design | Deferred until v0.1 spec is locked at Freeze |
| Milestone 10 — Compiler/runtime implementation | Happens in a separate repository; this project is documentation-only |
| Full LLM Toolchain implementation | Design/decisions only pre-Freeze; implementation deferred to post-Freeze (Milestone 8 area) |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DEBT-01 | Phase 1 | Done |
| DEBT-02 | Phase 1 | Done |
| DEBT-03 | Phase 1 | Done |
| DEBT-04 | Phase 1 | Done |
| DEBT-05 | Phase 1 | Done |
| DEBT-06 | Phase 1 | Done |
| DEBT-07 | Phase 1 | Done |
| DEBT-08 | Phase 1 | Done |
| DEBT-09 | Phase 1 | Done |
| DEBT-10 | Phase 1 | Done |
| DEBT-11 | Phase 1 | Done |
| DEBT-12 | Phase 1 | Done |
| DEBT-13 | Phase 1 | Done |
| DEBT-14 | Phase 1 | Done |
| DEBT-15 | Phase 1 | Done |
| DEBT-16 | Phase 1 | Done |
| DEBT-17 | Phase 1 | Done |
| PROC-01 | Phase 1.1 | Done |
| PROC-02 | Phase 1.1 | Done |
| PROC-03 | Phase 1.1 | Done |
| PROC-04 | Phase 1.1 | Done |
| PROC-05 | Phase 1.1 | Done |
| VISION-01 | Phase 1.1 | Done |
| VISION-02 | Phase 1.1 | Done |
| VISION-03 | Phase 1.1 | Done |
| PRINC-01 | Phase 1.1 | Done |
| FITNESS-01 | Phase 1.1 | Done |
| SEM-01 | Phase 2 | Complete |
| SEM-02 | Phase 2 | Complete |
| SEM-03 | Phase 2 | Complete |
| PRIM-01 | Phase 3 | Pending |
| PRIM-02 | Phase 3 | Pending |
| PRIM-03 | Phase 3 | Pending |
| DERIV-01 | Phase 4 | Pending |
| CONCEPT-ESS-01 | Phase 4 | Complete |
| CONCEPT-IMP-01 | Phase 4 | Complete |
| CONCEPT-DEFER-01 | Phase 4 | Pending |
| CONCEPT-REJECT-01 | Phase 4 | Pending |
| ANTIPAT-01 | Phase 4 | Pending |
| ANTIPAT-02 | Phase 4 | Pending |
| ANTIPAT-03 | Phase 4 | Pending |
| ANTIPAT-04 | Phase 4 | Pending |
| ANTIPAT-05 | Phase 4 | Pending |
| ANTIPAT-06 | Phase 4 | Pending |
| ANTIPAT-07 | Phase 4 | Pending |
| ANTIPAT-08 | Phase 4 | Pending |
| ANTIPAT-09 | Phase 4 | Pending |
| ANTIPAT-10 | Phase 4 | Pending |
| DERIV-02 | Phase 4 | Complete |
| DERIV-03 | Phase 4 | Complete |
| SYNTAX-01 | Phase 5 | Pending |
| SYNTAX-02 | Phase 5 | Pending |
| SYNTAX-03 | Phase 5 | Pending |
| SYNTAX-04 | Phase 5 | Pending |
| XCUT-01 | Phase 6 | Pending |
| XCUT-02 | Phase 6 | Pending |
| EXEC-01 | Phase 7 | Pending |
| EXEC-02 | Phase 7 | Pending |
| EXEC-03 | Phase 7 | Pending |
| EVOL-01 | Phase 8 | Pending |
| SPEC-01 | Phase 8 | Pending |
| VALID-01 | Phase 8 | Pending |
| NAME-01 | Phase 8 | Pending |
| FREEZE-01 | Phase 8 | Pending |
| FREEZE-02 | Phase 8 | Pending |
| FREEZE-03 | Phase 8 | Pending |
| FREEZE-04 | Phase 8 | Pending |
| FREEZE-05 | Phase 8 | Pending |
| FREEZE-06 | Phase 8 | Pending |
| FREEZE-07 | Phase 8 | Pending |
| ONBOARD-01 | Phase 9 | Pending |
| ONBOARD-02 | Phase 9 | Pending |
| ONBOARD-03 | Phase 9 | Pending |
| ONBOARD-04 | Phase 9 | Pending |

**Coverage:**

- v1 requirements: 74 total
- Mapped to phases: 74 (100%)
- Unmapped: 0

**Phase summary:**

- Phase 1 — Concerns Remediation: 17 requirements (DEBT-01..17) — **Done**
- Phase 1.1 — Foundation Completion: 10 requirements (PROC-01..05, VISION-01..03, PRINC-01, FITNESS-01)
- Phase 2 — Semantic Model: 3 requirements (SEM-01..03)
- Phase 3 — Primitive Blocks: 3 requirements (PRIM-01..03)
- Phase 4 — Derived Features & Decision Pipeline: 17 requirements (DERIV-01..03, CONCEPT-ESS-01, CONCEPT-IMP-01, CONCEPT-DEFER-01, CONCEPT-REJECT-01, ANTIPAT-01..10)
- Phase 5 — Syntax Design: 4 requirements (SYNTAX-01..04)
- Phase 6 — Cross-Cutting Review: 2 requirements (XCUT-01..02)
- Phase 7 — Execution & Optimization Model: 3 requirements (EXEC-01..03)
- Phase 8 — Evolution Model & Freeze: 11 requirements (EVOL-01, SPEC-01, VALID-01, NAME-01, FREEZE-01..07)
- Phase 9 — Onboarding Material: 4 requirements (ONBOARD-01..04)

---
*Requirements defined: 2026-07-20*
*Last updated: 2026-07-20 — Phase 2 broadened from "critical/high-severity blockers only" to all CONCERNS.md findings; added DEBT-11..17, 56/56 requirements mapped*
*Last updated: 2026-07-20 — Added VISION-03 (LLM-native language research → Phase 3 concept shortlist), 57/57 requirements mapped*
*Last updated: 2026-07-21 — Restructured to match `when/ROADMAP.md`'s 8-phase pipeline: Phase 1 (Concerns Remediation) marked Done; former Phase 2 renamed Phase 1.1 (Foundation Completion) and given PROC-05, PRINC-01, FITNESS-01; former Phase 3 split into Phase 2 (Semantic Model), Phase 3 (Primitive Blocks), Phase 4 (Derived Features & Decision Pipeline, +DERIV-01..03, ANTIPAT-11 retired in favor of DERIV-03); Phase 5 (Syntax Design) and Phase 7 (Execution & Optimization Model) added as new phases; former Phase 4 (Cross-Cutting) renumbered Phase 6 (+XCUT-02); former Phase 5 (Freeze & Naming) renumbered Phase 8 (+EVOL-01, SPEC-01, VALID-01). 66/66 requirements mapped.*
*Last updated: 2026-07-26 — Phase 2 (SEM-01..03) and Phase 3 (PRIM-01..03) now name their actual essential-tier source files (10 each, per `.planning/notes/2026-07-26-tier-vs-phase-mapping.md`); Phase 4's thirteen individually-hardcoded per-concept requirements (scoped when `how/concepts/research/` held ~22 files) replaced with 4 tier-scaled requirements — CONCEPT-ESS-01 (22 files, including 2 files a concurrent task moved into essential/ mid-cycle pending Phase classification), CONCEPT-IMP-01 (36 files), CONCEPT-DEFER-01 (54 files), CONCEPT-REJECT-01 (4 files) — covering the real 132-file inventory (42 essential / 36 important / 54 deferrable / 0 reject; 20 of the 42 essential files consumed by Phase 2/3), sequenced essential-first per SEED-001. Also corrected a pre-existing Traceability-table miscount found while editing this block: the table has always had 79 (now 70) individual rows, not the 66/57 totals a prior changelog entry stated; Coverage now reports the actual row count. Phase 4 total: 26 to 17 requirements; v1 total: 70 (previously incorrectly stated as 66).*
