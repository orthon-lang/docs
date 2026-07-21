# Roadmap: Orthon Language Specification

## Overview

This roadmap takes the v0.1 specification from its current state — Phase 1
(Concerns Remediation) complete, architecture stubs filled, and a redesigned
8-phase architectural design pipeline (see `when/ROADMAP.md`) — through to a
frozen, self-consistent v0.1 language specification. It mirrors the
**11-level engineering design hierarchy** adopted in `when/ROADMAP.md`:
finish the remaining Foundation work — vision consolidation, locking Design
Principles, and process infrastructure (Phase 1.1) — then define the
Semantic Model (Phase 2) and the minimal Primitive Blocks that decompose
from it (Phase 3), design every outstanding concept through the Decision
Pipeline (Phase 4), derive Syntax from the semantic model (Phase 5), verify
Cross-Cutting interactions (Phase 6), define the Execution & Optimization
boundary (Phase 7), and finally define the Evolution Model and Freeze v0.1
(Phase 8). Every phase in this roadmap is documentation/design work — there
is no compiler or runtime in this repository.

**Note on Phase 1.1:** this phase was inserted to carry forward the
"Foundations, Process & Vision" work that was previously tracked as a
separate Phase 2 before the roadmap was restructured on 2026-07-21 to match
`when/ROADMAP.md`'s 8-phase pipeline. It corresponds to the remainder of
`when/ROADMAP.md`'s Phase 1 ("Foundation") not already covered by the
completed Phase 1 (Concerns Remediation) below.

## Phases

**Phase Numbering:**

- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (1.1, 2.1): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Concerns Remediation** - Fill the four empty architecture specs and resolve the tech-debt items flagged in CONCERNS.md
- [ ] **Phase 1.1: Foundation Completion** (INSERTED) - Consolidate Vision, lock Design Principles as constitutional, build process infrastructure
- [ ] **Phase 2: Semantic Model** - Define identity, ownership, mutation, evaluation, visibility, and lifetime as one unified model
- [ ] **Phase 3: Primitive Blocks** - Identify the minimal orthogonal set of primitive building blocks
- [ ] **Phase 4: Derived Features & Decision Pipeline** - Design every outstanding concept through the Decision Pipeline and accept via EDR
- [ ] **Phase 5: Syntax Design** - Derive syntax from the semantic model for every accepted concept
- [ ] **Phase 6: Cross-Cutting Review** - Build the concept interaction matrix and resolve boundary conflicts
- [ ] **Phase 7: Execution & Optimization Model** - Define the boundary between language semantics and implementation
- [ ] **Phase 8: Evolution Model & Freeze** - Define versioning/deprecation policy, produce the canonical spec, validate, and freeze v0.1

## Phase Details

### Phase 1: Concerns Remediation

**Goal**: Every finding in CONCERNS.md — not only the critical and high-severity blockers, but every tech-debt, governance, and maintenance item on its Summary Table — is resolved or has an explicit, documented remediation, so later architecture-dependent work (Freeze, and any concept design that references parsing/type/name-resolution behavior) rests on real content instead of placeholders, and no unresolved concern is silently carried past this phase.
**Depends on**: Nothing (first phase)
**Requirements**: DEBT-01, DEBT-02, DEBT-03, DEBT-04, DEBT-05, DEBT-06, DEBT-07, DEBT-08, DEBT-09, DEBT-10, DEBT-11, DEBT-12, DEBT-13, DEBT-14, DEBT-15, DEBT-16, DEBT-17
**Status**: Complete (executed and verified 2026-07-20 — see `.planning/phases/01-concerns-remediation/01-01-SUMMARY.md`)
**Success Criteria** (what must be TRUE):

  1. `docs/how/architecture/IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, and `NAME_RESOLUTION.md` are each filled with a real design (intermediate representation format, parsing/grammar strategy, type inference/checking rules, and scoping/symbol-resolution rules respectively) — none remain at 0 lines.
  2. `docs/how/templates/_adr.md` is archived or removed, and no live document references it as the current template.
  3. Zero references to the superseded `EXECUTION_IMAGE.md` remain in active documents (e.g., `docs/how/concepts/research/imperative-crutch-lazy-sequences.md`, `imperative-crutch-resource-management.md`); all such cross-references point to `EXECUTION_PROGRAM.md`, and `EXECUTION_IMAGE.md` itself is archived.
  4. Concept and architecture documents carry date-stamp/freshness metadata (e.g., a `last_updated` field), addressing the "only 2 files have date stamps" finding.
  5. The EDR-008/009 numbering gap and TDR→EDR migration are documented in `how/decision_records/INDEX.md` (or EDR-001); `docs/how/architecture/FITNESS_FUNCTIONS.md` contains a full fitness-functions catalogue (not just 2 examples); `docs/how/DOCUMENTATION_PRINCIPLES.md` is expanded beyond its current template stub.
  6. All 22 concept documents have been audited against the `_concept.md` 8-section template; the under-developed 21-35 line drafts (`EQUALITY.md`, `FUNCTIONS.md`, `MUTABILITY.md`, `ALLOCATION.md`) are either brought to compliant depth or carry an explicit, tracked remediation plan, and the audit results (a produced gap list) demonstrate the template-compliance enforcement mechanism is actually applied, not just defined.
  7. Every tracked TODO/requirement item has an assigned owner and target-completion metadata, replacing unowned freeform checkboxes — resolving the "unclear milestone ownership" finding.
  8. A repository-wide cross-reference / "See also" link audit has been performed, all broken or superseded links found are fixed, and a link-validation approach for future changes is documented.
  9. A glossary maintenance workflow (how/when new terms get registered in `GLOSSARY.md`) and a versioning/change-log policy for large monolithic documents (`EXECUTION_PROGRAM.md`, `GLOSSARY.md`, `ARCHITECTURE.md`, `DESIGN_PRINCIPLES.md`, `AGENTS.md`) are both documented, and a documentation growth / information-architecture plan exists for scaling `how/concepts/research/` and future stdlib/tooling directories.

**Plans**: `01-concerns-remediation/PLAN.md` (executed, 1 summary)

---

### Phase 1.1: Foundation Completion (INSERTED)

**Goal**: The remaining "Foundation" work from `when/ROADMAP.md` Phase 1 — beyond concerns remediation, which is already done — is complete: Vision is consolidated into one canonical block, Design Principles are locked as the project's constitution, and process infrastructure (a one-page decision authority, a collapsed review pipeline, and throughput fitness functions) exists — so Phase 2 (Semantic Model) has a stable foundation and Phase 4 (Derived Features) has a defined, lightweight acceptance gate instead of stalling on undefined process.
**Depends on**: Phase 1
**Requirements**: PROC-01, PROC-02, PROC-03, PROC-04, PROC-05, VISION-01, VISION-02, VISION-03, PRINC-01, FITNESS-01
**Success Criteria** (what must be TRUE):

  1. A concept acceptance gate is documented — the DRAFT → Accepted transition, its owner, criteria, and EDR linkage (`docs/how/concept-design-review.md` is no longer a stub).
  2. `docs/how/concept-design-review.md` is collapsed from its current 11-step procedure to the pipeline-throughput-gaps-recommended 3-5 step process for solo authorship, and every former TODO.md milestone item is represented as a tracked GSD requirement with explicit status.
  3. `docs/how/process/DECISION_PROCESS.md` moves from its current DRAFT status to Accepted (referenced from `AGENTS.md` and `README.md` as the single source of decision authority), and `docs/how/process/DECISION_PIPELINE.md` (the 10-question feature pipeline) is finalized alongside it.
  4. `why/MANIFESTO.md`, `why/VISION.md`, and `why/ZEN.md` each show evidence of review against `why/WORKING_BACKWARDS.md` rationale, and are consolidated into a compact canonical `VISION.md` (Vision · Mission · Non-goals · Target Audience · Success Criteria), keeping the existing docs as reference expansions.
  5. `docs/notes/criteria-based-algorithm-selection.md` carries an explicit validation verdict (accepted / revised / rejected), and the LLM-native language research shortlist (`docs/notes/llm-native-concept-shortlist.md` or equivalent) is complete and ready to inform Phase 4's concept prioritization.
  6. `docs/how/DESIGN_PRINCIPLES.md` is declared closed for modification — any future change requires a Tier 1 EDR — and this rule is stated in the document itself.
  7. `docs/how/architecture/FITNESS_FUNCTIONS.md` includes throughput fitness functions (at minimum: concepts-in-flight, oldest-open-proposal age), per the `docs/notes/pipeline-throughput-gaps.md` analysis.

**Plans**: TBD

---

### Phase 2: Semantic Model

**Goal**: Define *what a program means* at the foundational level — identity, ownership, mutation, evaluation, visibility, lifetime — as a single unified, internally consistent model in `what/SEMANTIC_MODEL.md`, replacing its current DRAFT placeholder. This is the most expensive part of language design to get wrong; every derived feature in Phase 4 depends on it being settled first.
**Depends on**: Phase 1.1 (Design Principles locked, Vision consolidated)
**Requirements**: SEM-01, SEM-02, SEM-03
**Success Criteria** (what must be TRUE):

  1. `what/SEMANTIC_MODEL.md` defines all 6 semantic dimensions (Identity, Ownership, Mutation, Evaluation, Visibility, Lifetime) as a coherent whole, informed by existing research (`DATA_MODEL.md`, `OWNERSHIP.md`, `MUTABILITY.md`, `ALLOCATION.md`, `EQUALITY.md`, `FOUNDATIONAL_ABSTRACTIONS.md`) — the document is no longer a placeholder.
  2. Cross-dimension conflicts are checked and resolved (e.g., ownership × mutation, evaluation × lifetime), and the model is validated against every Design Principle with no contradictions found.
  3. An EDR (Architecture category) accepts the Semantic Model.

**Plans**: TBD

---

### Phase 3: Primitive Blocks

**Goal**: Identify the minimal orthogonal set of primitive building blocks in `what/PRIMITIVE_BLOCKS.md`, replacing its current DRAFT placeholder. Every derived feature designed in Phase 4 must decompose into these; if a feature cannot be decomposed, the primitive set is incomplete and must be revisited before Phase 4 proceeds.
**Depends on**: Phase 2 (primitives are defined in terms of the Semantic Model)
**Requirements**: PRIM-01, PRIM-02, PRIM-03
**Success Criteria** (what must be TRUE):

  1. `what/PRIMITIVE_BLOCKS.md` fully specifies the minimal candidate set (identifier, literal, assignment, function, call, attribute access, scope, reference, pack, unpack, operator definition) plus composition rules, each primitive orthogonal (no overlap) and mapped to the Semantic Model.
  2. Every concept research doc in `how/concepts/research/` is verified to decompose onto the primitive set; the set is confirmed minimal — removing any primitive makes some known feature inexpressible.
  3. An EDR (Architecture category) accepts the Primitive Blocks set.

**Plans**: TBD

---

### Phase 4: Derived Features & Decision Pipeline

**Goal**: Every outstanding language concept is fully designed — run through the 10-question Decision Pipeline (`how/process/DECISION_PIPELINE.md`), decomposed onto Primitive Blocks, classified as Language/Standard Library/External, and accepted via EDR — informed by anti-pattern research into imperative crutches and Phase 1.1's LLM-native concept shortlist. Zero `DRAFT` headers remain in `how/concepts/research/` when this phase completes.
**Depends on**: Phase 3 (Primitive Blocks must exist to decompose into)
**Requirements**: DERIV-01, DERIV-02, DERIV-03, CONCEPT-01, CONCEPT-02, CONCEPT-03, CONCEPT-04, CONCEPT-05, CONCEPT-06, CONCEPT-07, CONCEPT-08, CONCEPT-09, CONCEPT-10, CONCEPT-11, CONCEPT-12, CONCEPT-13, ANTIPAT-01, ANTIPAT-02, ANTIPAT-03, ANTIPAT-04, ANTIPAT-05, ANTIPAT-06, ANTIPAT-07, ANTIPAT-08, ANTIPAT-09, ANTIPAT-10
**Success Criteria** (what must be TRUE):

  1. `how/process/DECISION_PIPELINE.md` (the 10-question pipeline) is finalized and applied to every one of the 13 outstanding concepts (`PATTERN_MATCHING.md`, `ERROR_HANDLING.md`, `OWNERSHIP.md`, `GENERICS.md`, `ASYNC_AWAIT.md`, `OBJECT_INITIALIZATION.md`, `CONCURRENCY.md`, `LITERATE_PROGRAMMING.md`, `SORTING.md`, `UNPACKING.md`, `GENERATORS.md`, `METAOBJECTS.md`, `SPAN.md`).
  2. All 10 `docs/how/concepts/research/imperative-crutch-*.md` research topics are completed with documented findings, and those implications are reflected in the relevant concept designs.
  3. `what/LIBRARY_BOUNDARY.md` documents the language-vs-stdlib-vs-external classification rationale for every feature classified as Standard Library or External.
  4. Every feature classified "Language" has a corresponding EDR (Architecture category) and moves to `what/concepts/`; zero `⚠️ DRAFT` headers remain across `how/concepts/research/`.

**Plans**: TBD

---

### Phase 5: Syntax Design

**Goal**: Design syntax as the external interface of the semantic model — derived from semantics, not designed independently — for every concept accepted in Phase 4.
**Depends on**: Phase 4 (concepts must be accepted before syntax is designed)
**Requirements**: SYNTAX-01, SYNTAX-02, SYNTAX-03, SYNTAX-04
**Success Criteria** (what must be TRUE):

  1. Syntax principles are established and documented (one concept → one syntax, one symbol → one context-independent meaning, no significant whitespace, named forms preferred over symbolic where ambiguity risk exists).
  2. `what/SYNTAX.md` documents syntax for every accepted Language concept, with all canonical forms shown together (per AGENTS.md §4.2), consistent across concepts.
  3. Syntax is validated: no ambiguous or context-dependent parses, LLM generability check passed, and a readability check confirms a human can predict the syntax from semantics alone.
  4. `how/architecture/PARSER.md` is updated with the concrete grammar.

**Plans**: TBD

---

### Phase 6: Cross-Cutting Review

**Goal**: Verify that all accepted concepts work together without conflicts — produce the full pairwise Interaction Matrix and resolve every concept-boundary conflict — finalizing the format investigation from `docs/notes/interaction-matrix-format.md`.
**Depends on**: Phase 4 (all concepts designed), Phase 5 (syntax stable enough to check interactions)
**Requirements**: XCUT-01, XCUT-02
**Success Criteria** (what must be TRUE):

  1. `docs/notes/interaction-matrix-format.md` research is finalized into a concrete, adopted Interaction Matrix format, and `what/CROSS_CUTTING.md` contains the full pairwise analysis for every concept pair Phase 4 identified as interacting (e.g., Generator × Error Handling, Pattern Matching × Types, Async/Await × Error Handling, Ownership × Mutation, Generics × Pattern Matching).
  2. `what/CONFLICT_REGISTRY.md` records every open conflict found; all are resolved or carry a documented EDR deferral rationale — no unresolvable contradictions remain.

**Plans**: TBD

---

### Phase 7: Execution & Optimization Model

**Goal**: Define the boundary between language semantics and implementation — what the language guarantees about *how* a program executes vs. what is an optimisation choice left to the Implementation Strategy.
**Depends on**: Phase 6 (stable concept set), Phase 2 (Semantic Model)
**Requirements**: EXEC-01, EXEC-02, EXEC-03
**Success Criteria** (what must be TRUE):

  1. `what/EXECUTION_MODEL.md` defines execution semantics, formalizes the Execution Program concept via EDR, and documents supported execution targets (Interpreter, AOT, JIT, LLVM, WASM, Container).
  2. `what/OPTIMIZATION_MODEL.md` catalogues optimisations (inlining, constant folding, escape analysis, dead code elimination, evaluation order, memory layout), each explicitly classified as Optimisation, Semantics, or Strategy-dependent.
  3. `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md`, and `LLM_STRATEGY.md` are all reconciled and consistent with the Execution/Optimization Model.

**Plans**: TBD

---

### Phase 8: Evolution Model & Freeze

**Goal**: Define how the language evolves over time, then freeze Orthon v0.1 as a complete, self-consistent language specification — every concept Accepted, architecture specs filled, cross-references valid, the "Orthon" name settled, and the version tagged with a whitepaper and launch plan ready to execute.
**Depends on**: Phase 7 (Execution Model part of frozen spec), Phase 6 (cross-cutting issues resolved), Phase 5 (syntax resolved), Phase 4 (concepts designed)
**Requirements**: EVOL-01, SPEC-01, VALID-01, NAME-01, FREEZE-01, FREEZE-02, FREEZE-03, FREEZE-04, FREEZE-05, FREEZE-06, FREEZE-07
**Success Criteria** (what must be TRUE):

  1. `how/EVOLUTION_MODEL.md` documents the versioning scheme, deprecation policy, experimental/feature-gate mechanism, and breaking-change policy, approved via EDR.
  2. A global consistency review passes — syntax, naming, and operator uniformity checked; no special cases; global minimality verified (no concept duplicates another's responsibility) — and the canonical `SPEC.md` plus machine-readable `SCHEMA.md` are produced.
  3. Reference examples/programs are written, LLM generation tests and schema round-trip tests pass, and any friction points are documented with new EDRs raised for issues discovered.
  4. A documented decision (EDR or equivalent) confirms "Orthon" as the final project name, or names a replacement with rationale.
  5. Every open validation issue tracked across the project (concept review notes, CONCERNS.md items, Open Questions sections) has either been resolved or carries an explicit, documented deferral rationale.
  6. A link-check of all "See also" and cross-reference links across the repository reports zero dangling references.
  7. A version tag (e.g., "Orthon 0.1 Specification") exists, marking the frozen spec.
  8. Whitepaper strategy in `docs/notes/whitepaper-strategy.md` has been revisited with at least one whitepaper commissioned or drafted, and a launch/announcement strategy (audience, channels, timing) is documented.

**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 1.1 → 2 → 3 → 4 → 5 → 6 → 7 → 8

| Phase | Plans Complete | Status | Completed |
|-------|-----------------|--------|-----------|
| 1. Concerns Remediation | 1/1 | Complete | 2026-07-20 |
| 1.1. Foundation Completion (INSERTED) | 0/TBD | Not started | - |
| 2. Semantic Model | 0/TBD | Not started | - |
| 3. Primitive Blocks | 0/TBD | Not started | - |
| 4. Derived Features & Decision Pipeline | 0/TBD | Not started | - |
| 5. Syntax Design | 0/TBD | Not started | - |
| 6. Cross-Cutting Review | 0/TBD | Not started | - |
| 7. Execution & Optimization Model | 0/TBD | Not started | - |
| 8. Evolution Model & Freeze | 0/TBD | Not started | - |
