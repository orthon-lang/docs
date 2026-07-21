---
phase: 01-concerns-remediation
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - docs/how/architecture/IR.md
  - docs/how/architecture/PARSER.md
  - docs/how/architecture/TYPE_SYSTEM.md
  - docs/how/architecture/NAME_RESOLUTION.md
  - docs/how/architecture/FITNESS_FUNCTIONS.md
  - docs/how/DOCUMENTATION_PRINCIPLES.md
  - docs/how/templates/_adr.md
  - docs/how/concepts/research/EXECUTION_IMAGE.md
  - docs/how/concepts/research/imperative-crutch-lazy-sequences.md
  - docs/how/concepts/research/imperative-crutch-resource-management.md
  - docs/how/decision_records/INDEX.md
  - docs/how/decision_records/process/EDR-001-edr-system.md
  - docs/how/concepts/research/EQUALITY.md
  - docs/how/concepts/research/FUNCTIONS.md
  - docs/how/concepts/research/MUTABILITY.md
  - docs/how/concepts/research/ALLOCATION.md
  - docs/how/concepts/research/DATA_MODEL.md
  - docs/how/concepts/research/CORE_CONCEPTS.md
  - docs/how/concepts/research/OWNERSHIP.md
  - docs/how/concepts/research/ERROR_HANDLING.md
  - docs/how/concepts/research/GENERICS.md
  - docs/how/concepts/research/SPAN.md
  - docs/how/concepts/research/METAOBJECTS.md
  - docs/how/concepts/research/EXECUTION_PROGRAM.md
  - docs/how/concepts/research/GENERATORS.md
  - docs/how/concepts/research/LLM_NATIVE_TOOLCHAIN.md
  - docs/how/concepts/research/UNPACKING.md
  - docs/how/concepts/research/ASYNC_AWAIT.md
  - docs/how/concepts/research/SORTING.md
  - docs/how/concepts/research/LITERATE_PROGRAMMING.md
  - docs/how/concepts/research/PATTERN_MATCHING.md
  - docs/how/concepts/research/CONCURRENCY.md
  - docs/how/concepts/research/OBJECT_INITIALIZATION.md
  - docs/what/GLOSSARY.md
  - docs/when/ROADMAP.md
  - docs/.planning/ROADMAP.md
  - docs/.planning/REQUIREMENTS.md
  - docs/.planning/codebase/CONCERNS.md
  - docs/AGENTS.md
  - docs/TODO.md
autonomous: false
requirements:
  - DEBT-01
  - DEBT-02
  - DEBT-03
  - DEBT-04
  - DEBT-05
  - DEBT-06
  - DEBT-07
  - DEBT-08
  - DEBT-09
  - DEBT-10
  - DEBT-11
  - DEBT-12
  - DEBT-13
  - DEBT-14
  - DEBT-15
  - DEBT-16
  - DEBT-17

must_haves:
  truths:
    - IR.md, PARSER.md, TYPE_SYSTEM.md, NAME_RESOLUTION.md each contain real design specifications (interface contracts, architectural invariants, key decisions) — none remain at 0 lines
    - Deprecated _adr.md template is archived; no live document references it as the current template
    - Zero references to superseded EXECUTION_IMAGE.md remain in active documents; all cross-references point to EXECUTION_PROGRAM.md
    - Every concept and architecture document carries a date-stamp or last_updated field
    - EDR-008/009 gap is documented in INDEX.md and EDR-001; TDR→EDR migration rationale is recorded
    - FITNESS_FUNCTIONS.md contains a full catalogue mapping principles to measurable checks (not just 2 examples)
    - DOCUMENTATION_PRINCIPLES.md is expanded beyond its 25-line stub and corrected to list 8 sections (not 7)
    - All 22 concept documents have been audited against the 8-section _concept.md template; under-developed ones (EQUALITY.md, FUNCTIONS.md, MUTABILITY.md, ALLOCATION.md, DATA_MODEL.md) brought to structurally complete drafts or carry explicit remediation plans
    - Every tracked TODO/requirement item has assigned owner and target-completion metadata
    - Repository-wide cross-reference link audit performed; broken/superseded links fixed
    - Glossary maintenance workflow, versioning/change-log policy, and documentation growth/IA plan are documented
    - All "7-section" errors in ROADMAP.md, CONCERNS.md, DOCUMENTATION_PRINCIPLES.md, when/ROADMAP.md, AGENTS.md, and REQUIREMENTS.md are corrected to "8-section"
  artifacts:
    - docs/how/architecture/IR.md (filled)
    - docs/how/architecture/PARSER.md (filled)
    - docs/how/architecture/TYPE_SYSTEM.md (filled)
    - docs/how/architecture/NAME_RESOLUTION.md (filled)
    - docs/how/architecture/FITNESS_FUNCTIONS.md (full catalogue)
    - docs/how/DOCUMENTATION_PRINCIPLES.md (expanded, 8-sections)
    - .archived/templates/_adr.md (moved)
    - .archived/concepts/EXECUTION_IMAGE.md (moved)
    - docs/how/concepts/research/imperative-crutch-lazy-sequences.md (updated refs)
    - docs/how/concepts/research/imperative-crutch-resource-management.md (updated refs)
    - docs/how/decision_records/INDEX.md (gap documented)
    - docs/how/decision_records/process/EDR-001-edr-system.md (gap documented)
    - docs/how/concepts/research/EQUALITY.md (structurally complete draft)
    - docs/how/concepts/research/FUNCTIONS.md (structurally complete draft)
    - docs/how/concepts/research/MUTABILITY.md (structurally complete draft)
    - docs/how/concepts/research/ALLOCATION.md (structurally complete draft)
    - Concept audit gap list (audit output file)
    - Ownership assignment in TODO.md
    - Link audit log
    - Glossary workflow reference in GLOSSARY.md tail
    - VERSIONING_POLICY.md
    - IA_PLAN.md
  key_links:
    - Arch specs (IR, Parser, Type System, Name Resolution) must align with ARCHITECTURE.md's layered model and not contradict each other or EDR-010
    - Template corrections (7→8 sections) applied consistently across ROADMAP.md, CONCERNS.md, DOCUMENTATION_PRINCIPLES.md, when/ROADMAP.md, AGENTS.md, REQUIREMENTS.md
    - EXECUTION_IMAGE references found and fixed in ALL notes files (not just the 2 mentioned)
    - Fitness functions catalogue maps each of 27 DESIGN_PRINCIPLES.md principles to at least one measurable function
---

<objective>
Resolve all 17 DEBT items (DEBT-01 through DEBT-17) from CONCERNS.md — fill empty architecture specs, clean up deprecated/superseded content, add freshness metadata, expand fitness functions and documentation principles, audit all concept documents against the 8-section template, assign ownership to all tracked items, perform a repository-wide link audit, and document glossary/versioning/IA processes — so later architecture-dependent work in Phases 3 and 5 rests on real content instead of placeholders, and no unresolved concern is silently carried past this phase.

Purpose: Clear tech-debt blockers and governance gaps before investing in process/vision work (Phase 2) and concept design (Phase 3).
Output: All 17 DEBT requirements satisfied; filled arch specs, cleaned-up templates, date-stamped docs, expanded catalogues, concept audit, link audit, and 3 process docs.
</objective>

<execution_context>
@$HOME/.claude/gsd-core/workflows/execute-plan.md
@$HOME/.claude/gsd-core/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
@.planning/REQUIREMENTS.md
@.planning/phases/01-concerns-remediation/01-CONTEXT.md
@.planning/codebase/CONCERNS.md
@.planning/codebase/STRUCTURE.md
@.planning/codebase/CONVENTIONS.md
@docs/how/templates/_concept.md
@docs/how/architecture/ARCHITECTURE.md
@docs/how/architecture/FITNESS_FUNCTIONS.md
@docs/how/DOCUMENTATION_PRINCIPLES.md
@docs/how/gates/_language-design.md
@docs/how/decision_records/INDEX.md
@docs/how/decision_records/process/EDR-001-edr-system.md
@docs/how/decision_records/quality/EDR-005-fitness-functions.md
@docs/what/DESIGN_PRINCIPLES.md
@docs/how/concepts/research/EQUALITY.md
@docs/how/concepts/research/ALLOCATION.md
@docs/how/concepts/research/CORE_CONCEPTS.md
@docs/what/GLOSSARY.md
@docs/when/ROADMAP.md
@docs/AGENTS.md
@docs/TODO.md
@docs/how/concepts/research/imperative-crutch-lazy-sequences.md
@docs/how/concepts/research/imperative-crutch-resource-management.md
</context>

<tasks>

<!-- ========================================================================== -->
<!-- WAVE A: Content Creation (independent parallel tasks) — per D-04            -->
<!-- ========================================================================== -->

<task type="auto">
  <name>Task A.1: Fill IR.md and PARSER.md with design specifications</name>
  <files>
    docs/how/architecture/IR.md
    docs/how/architecture/PARSER.md
  </files>
  <action>
    Per D-01: Fill each file with a design specification (interface contracts, architectural invariants, key design decisions) — NOT implementation-level detail. Follow ARCHITECTURE.md's layered model and respect Core Language → Syntax → Standard Library → Implementation Strategy boundaries.

    For IR.md:
    - Define IR's role in the architecture: bridge between parsed source and code generation, must be strategy-independent (per D-01)
    - Specify IR invariants: strategy-independence, composability, canonical representation for each Core Language construct
    - Define IR interface contracts: what the IR guarantees to consumers (type-checker, code generator), what it abstracts over
    - Document key design decisions: IR as an explicit tree/graph (not opaque bytecode), representation symmetry with source AST, support for incremental compilation
    - Document relationship to TYPE_SYSTEM.md (IR carries typed nodes) and NAME_RESOLUTION.md (IR nodes reference resolved symbols)
    - Add date-stamp metadata (per DEBT-07): `> **Last updated:** 2026-07-20`
    - Structure: ## Overview, ## Role in Architecture, ## IR Invariants, ## IR Interface Contracts, ## Key Design Decisions, ## Relationships, ## Open Questions

    For PARSER.md:
    - Define parser's role: produce an AST conforming to Core Language semantics — no semantic analysis in parse phase
    - Specify parsing invariants: error recovery, whitespace-significant vs. delimiter-based decision (defer to Phase 3), context-free grammar requirement
    - Define parser interface contracts: what the parser guarantees to consumers (name resolver, type checker)
    - Document key design decisions: recursive descent as default strategy (flexible for experimentation), lexer as a separate pass, support for incremental parsing
    - Document relationship to LEXER.md concept (if one exists) and NAME_RESOLUTION.md (parser feeds unresolved AST to name resolver)
    - Add date-stamp metadata
    - Structure: ## Overview, ## Role in Architecture, ## Parsing Invariants, ## Interface Contracts, ## Key Design Decisions, ## Relationships, ## Open Questions
  </action>
  <verify>
    <automated>grep -c '## Overview' docs/how/architecture/IR.md && grep -c '## Overview' docs/how/architecture/PARSER.md</automated>
  </verify>
  <done>
    IR.md contains ≥200 words of design specification with defined invariants, interface contracts, and key decisions. PARSER.md similarly filled. Both have date-stamp metadata.
  </done>
</task>

<task type="auto">
  <name>Task A.2: Fill TYPE_SYSTEM.md and NAME_RESOLUTION.md with design specifications</name>
  <files>
    docs/how/architecture/TYPE_SYSTEM.md
    docs/how/architecture/NAME_RESOLUTION.md
  </files>
  <action>
    Per D-01: Fill each file with design specifications — NOT implementation-level detail (no concrete unification algorithm, no symbol table data structure). Follow ARCHITECTURE.md's layered model.

    For TYPE_SYSTEM.md:
    - Define type system's role: validate semantic correctness via type checking and inference, bridge between parsed AST and IR generation
    - Specify type system invariants: soundness (no runtime type errors for well-typed programs), completeness (all valid programs are typeable), principal types (every expression has a unique most-general type)
    - Define interface contracts: what the type checker guarantees to downstream consumers (IR generation, strategy selection)
    - Document key design decisions: Hindley-Milner-based inference as the default (proven, predictable), structural typing over nominal (aligns with Data-first philosophy), explicit type annotations required at module boundaries
    - Document relationship to CORE_CONCEPTS.md (Data representations map to types), NAME_RESOLUTION.md (types reference resolved symbols), IR.md (typed IR nodes)
    - Add date-stamp metadata
    - Structure: ## Overview, ## Role in Architecture, ## Type System Invariants, ## Interface Contracts, ## Key Design Decisions, ## Relationships, ## Open Questions

    For NAME_RESOLUTION.md:
    - Define name resolution's role: resolve all identifiers to their declarations, establish scoping rules, detect shadowing/ambiguity
    - Specify name resolution invariants: hygienic scoping (no accidental capture), lexical scope by default, explicit shadowing with warning
    - Define interface contracts: what the resolver guarantees (e.g., every identifier resolves to exactly one declaration)
    - Document key design decisions: lexical (static) scoping as default, module-level explicit imports, no implicit globals, defer module system design to Phase 3
    - Document relationship to PARSER.md (resolver consumes AST), TYPE_SYSTEM.md (resolved names feed type checker), IMPLEMENTATION_POLICIES.md (scope policy)
    - Add date-stamp metadata
    - Structure: ## Overview, ## Role in Architecture, ## Name Resolution Invariants, ## Interface Contracts, ## Key Design Decisions, ## Relationships, ## Open Questions
  </action>
  <verify>
    <automated>grep -c '## Overview' docs/how/architecture/TYPE_SYSTEM.md && grep -c '## Overview' docs/how/architecture/NAME_RESOLUTION.md</automated>
  </verify>
  <done>
    TYPE_SYSTEM.md and NAME_RESOLUTION.md each contain ≥200 words of design specification with defined invariants, interface contracts, and key decisions. Both have date-stamp metadata.
  </done>
</task>

<task type="auto">
  <name>Task A.3: Expand FITNESS_FUNCTIONS.md to full catalogue</name>
  <files>
    docs/how/architecture/FITNESS_FUNCTIONS.md
  </files>
  <action>
    Per D-02: Source fitness functions from existing latent patterns in the codebase. Each of the 27 principles in DESIGN_PRINCIPLES.md should map to at least one measurable fitness function, following the pattern of the 3 existing entries (Concept Stability, Layered Isolation, Composition Surface).

    Source documents to mine for functions:
    - DESIGN_PRINCIPLES.md (27 principles across 3 groups: Core Philosophy, Language Consistency, Execution Model)
    - EDR-005-fitness-functions.md (concept and rationale for fitness functions)
    - _language-design.md gate checklist (each criterion implies a fitness function)
    - EDR-004-language-design-checklist.md (rationale behind the checklist)
    - EDR-010-layered-architecture.md (layer boundaries)
    - EDR-011-llm-generability-gate.md (LLM-specific fitness)

    New fitness functions to add (at minimum):
    - **Data First** — Every new concept must operate on Data as the primary abstraction; verify no concept introduces behavior without a Data representation
    - **Orthogonality** — No two concepts overlap in responsibility; verify composability matrix has no forbidden pairs
    - **Explicitness** — Semantic changes must be syntactically visible; no hidden conversions or implicit side-effects
    - **Minimal Core** — Core Language changes must be justified against composition alternative
    - **Named Before Symbolic** — Every symbolic operator has a named function equivalent
    - **LLM Generability** — Every construct must be expressible in machine-readable schema with no ambiguities
    - **Policy Independence** — Core concepts must not depend on specific Policy values
    - **Cross-Reference Validity** — All "See also" links between documents resolve correctly
    - **Document Freshness** — Documents with last_updated > 90 days trigger review flag

    Each fitness function entry should follow the established pattern: ## Name, one-paragraph description of what it measures, and a reference to the source principle(s) or EDR(s). Preserve the 3 existing entries verbatim.

    Add date-stamp metadata.
  </action>
  <verify>
    <automated>grep -c '^## ' docs/how/architecture/FITNESS_FUNCTIONS.md</automated>
  </verify>
  <done>
    FITNESS_FUNCTIONS.md contains all 3 existing entries preserved plus ≥9 new fitness functions, each mapping to a DESIGN_PRINCIPLES.md principle or gate criterion. Has date-stamp metadata.
  </done>
</task>

<task type="auto">
  <name>Task A.4: Expand DOCUMENTATION_PRINCIPLES.md and fix 7→8 section error</name>
  <files>
    docs/how/DOCUMENTATION_PRINCIPLES.md
  </files>
  <action>
    Per D-03: The actual _concept.md template has 8 sections (Issue, Principles, Policy Footprint, Model, Default Strategy, Alternative Strategies, Open Questions, Decision History). The current DOCUMENTATION_PRINCIPLES.md lists only 7 (missing Policy Footprint). Fix this and expand the document beyond its ~25-line stub.

    Specific changes:
    1. Correct the section list from 7 to 8 — add "Policy Footprint" between Principles and Model
    2. Add a new § documenting the Policy Footprint section requirement: each concept must declare which Policy Types it involves and how
    3. Add principle: "Affected Documents Checklist" — every concept must include the affected-documents checklist from the template
    4. Add principle: "Date-Stamp Requirement" — every concept and architecture document must carry a `> **Last updated:** YYYY-MM-DD` metadata line
    5. Add principle: "All Canonical Forms" (preserve existing, expand with examples from CORE_CONCEPTS.md)
    6. Add principle: "Model Before Implementation" — code examples precede semantic explanation
    7. Restructure: ## Document Status, ## Section Structure (8-section list), ## Writing Principles (subsections per principle)
    8. Add date-stamp metadata

    Keep the existing "Show All Canonical Forms" and "Concept Document Template" sections; merge them into the new structure.
  </verify>
  <verify>
    <automated>grep -c 'Policy Footprint' docs/how/DOCUMENTATION_PRINCIPLES.md</automated>
  </verify>
  <done>
    DOCUMENTATION_PRINCIPLES.md lists 8 sections (including Policy Footprint), contains ≥5 documented writing principles, and carries date-stamp metadata.
  </done>
</task>

<!-- ========================================================================== -->
<!-- WAVE B: Cleanup & Metadata (depends on Wave A) — per D-04                   -->
<!-- ========================================================================== -->

<task type="auto">
  <name>Task B.1: Archive deprecated _adr.md template and update references</name>
  <files>
    docs/how/templates/_adr.md
    docs/AGENTS.md
  </files>
  <action>
    Per DEBT-05: The _adr.md template is marked DEPRECATED. It references the old ADR-NNN system which has been superseded by EDR-001.

    Actions:
    1. Create .archived/ directory at docs root: `.archived/templates/`
    2. Move `docs/how/templates/_adr.md` to `.archived/templates/_adr.md` — preserve the file with its DEPRECATED header for historical reference
    3. Check AGENTS.md Document Map (§3) — if it lists `_adr.md`, update the entry to note it's archived and point to `_edr-architecture.md` instead
    4. Check `docs/AGENTS.md` §10.9 or any other section that references `_adr.md` — remove or update
    5. Verify no `ADR-NNN` references exist in live documents (grep for ADR-NNN pattern excluding .archived/)

    Note: The grep for ADR-NNN in live docs should find zero results since the migration was already complete per CONCERNS.md.
  </action>
  <verify>
    <automated>test -f .archived/templates/_adr.md</automated>
  </verify>
  <done>
    _adr.md moved to .archived/templates/_adr.md. AGENTS.md updated to reference _edr-architecture.md instead. No live ADR-NNN references remain.
  </done>
</task>

<task type="auto">
  <name>Task B.2: Archive EXECUTION_IMAGE.md and update all cross-references</name>
  <files>
    docs/how/concepts/research/EXECUTION_IMAGE.md
    docs/how/concepts/research/imperative-crutch-lazy-sequences.md
    docs/how/concepts/research/imperative-crutch-resource-management.md
  </files>
  <action>
    Per DEBT-06: EXECUTION_IMAGE.md is SUPERSEDED by EXECUTION_PROGRAM.md but still referenced in two notes files.

    Actions:
    1. Create `.archived/concepts/` directory
    2. Move `docs/how/concepts/research/EXECUTION_IMAGE.md` to `.archived/concepts/EXECUTION_IMAGE.md` — preserve the SUPERSEDED header and the note about it being replaced by EXECUTION_PROGRAM.md
    3. Fix `docs/how/concepts/research/imperative-crutch-lazy-sequences.md`: replace "See also: concepts `GENERATORS.md`, `EXECUTION_IMAGE.md`, `EXECUTION_PROGRAM.md`" with "See also: concepts `GENERATORS.md`, `EXECUTION_PROGRAM.md`"
    4. Fix `docs/how/concepts/research/imperative-crutch-resource-management.md`: replace "See also: concepts `OWNERSHIP.md`, `EXECUTION_IMAGE.md`" with "See also: concepts `OWNERSHIP.md`, `EXECUTION_PROGRAM.md`"
    5. Grep entire docs/ tree for any other EXECUTION_IMAGE references outside .planning/ and .archived/ — fix any found
  </action>
  <verify>
    <automated>test -f .archived/concepts/EXECUTION_IMAGE.md && ! grep -q 'EXECUTION_IMAGE' docs/how/concepts/research/imperative-crutch-lazy-sequences.md && ! grep -q 'EXECUTION_IMAGE' docs/how/concepts/research/imperative-crutch-resource-management.md</automated>
  </verify>
  <done>
    EXECUTION_IMAGE.md moved to .archived/concepts/. Both notes files updated to reference EXECUTION_PROGRAM.md. No stale EXECUTION_IMAGE refs remain in active documents.
  </done>
</task>

<task type="auto">
  <name>Task B.3: Add date-stamp/freshness metadata to all concept and architecture documents</name>
  <files>
    docs/how/architecture/ARCHITECTURE.md
    docs/how/architecture/FITNESS_FUNCTIONS.md
    docs/how/architecture/IR.md
    docs/how/architecture/PARSER.md
    docs/how/architecture/TYPE_SYSTEM.md
    docs/how/architecture/NAME_RESOLUTION.md
    docs/how/concepts/research/CORE_CONCEPTS.md
    docs/how/concepts/research/DATA_MODEL.md
    docs/how/concepts/research/ALLOCATION.md
    docs/how/concepts/research/OWNERSHIP.md
    docs/how/concepts/research/MUTABILITY.md
    docs/how/concepts/research/EQUALITY.md
    docs/how/concepts/research/FUNCTIONS.md
    docs/how/concepts/research/ASYNC_AWAIT.md
    docs/how/concepts/research/CONCURRENCY.md
    docs/how/concepts/research/GENERATORS.md
    docs/how/concepts/research/ERROR_HANDLING.md
    docs/how/concepts/research/PATTERN_MATCHING.md
    docs/how/concepts/research/GENERICS.md
    docs/how/concepts/research/OBJECT_INITIALIZATION.md
    docs/how/concepts/research/EXECUTION_PROGRAM.md
    docs/how/concepts/research/LLM_NATIVE_TOOLCHAIN.md
    docs/how/concepts/research/LITERATE_PROGRAMMING.md
    docs/how/concepts/research/METAOBJECTS.md
    docs/how/concepts/research/SPAN.md
    docs/how/concepts/research/UNPACKING.md
    docs/how/concepts/research/SORTING.md
  </files>
  <action>
    Per DEBT-07: Add a `> **Last updated:** YYYY-MM-DD` metadata line to every concept and architecture document. Use `2026-07-20` as the date for all documents (this phase's planning date).

    For documents that have a DRAFT or SUPERSEDED header block (e.g., `> **⚠️ DRAFT — ...`), insert the date-stamp line after the existing header block, separated by a blank line.

    For documents without any header block (e.g., ARCHITECTURE.md which starts with `> Orthon is a programming language...`), insert the date-stamp at the top of the file, after the title heading if one exists, or as the first non-heading line.

    Format:
    ```
    > **Last updated:** 2026-07-20
    ```

    For the 4 arch spec files just created (IR.md, PARSER.md, TYPE_SYSTEM.md, NAME_RESOLUTION.md): the date-stamp should already be added as part of Tasks A.1/A.2 — skip if already present. For FITNESS_FUNCTIONS.md: should already be added in Task A.3.

    For architecture files that are not concept docs (ARCHITECTURE.md, ANALOGIES.md):
    - Add as a standalone line after the opening blockquote paragraph.

    The 4 empty files should now have content from Wave A — add the date-stamp as part of their content structure.
  </action>
  <verify>
    <automated>for f in docs/how/architecture/ARCHITECTURE.md docs/how/architecture/FITNESS_FUNCTIONS.md docs/how/architecture/IR.md docs/how/architecture/PARSER.md docs/how/architecture/TYPE_SYSTEM.md docs/how/architecture/NAME_RESOLUTION.md docs/how/concepts/research/CORE_CONCEPTS.md docs/how/concepts/research/DATA_MODEL.md docs/how/concepts/research/ALLOCATION.md docs/how/concepts/research/OWNERSHIP.md docs/how/concepts/research/MUTABILITY.md docs/how/concepts/research/EQUALITY.md docs/how/concepts/research/FUNCTIONS.md docs/how/concepts/research/ASYNC_AWAIT.md docs/how/concepts/research/CONCURRENCY.md docs/how/concepts/research/GENERATORS.md docs/how/concepts/research/ERROR_HANDLING.md docs/how/concepts/research/PATTERN_MATCHING.md docs/how/concepts/research/GENERICS.md docs/how/concepts/research/OBJECT_INITIALIZATION.md docs/how/concepts/research/EXECUTION_PROGRAM.md docs/how/concepts/research/LLM_NATIVE_TOOLCHAIN.md docs/how/concepts/research/LITERATE_PROGRAMMING.md docs/how/concepts/research/METAOBJECTS.md docs/how/concepts/research/SPAN.md docs/how/concepts/research/UNPACKING.md docs/how/concepts/research/SORTING.md; do grep -q 'Last updated' "$f" || echo "MISSING: $f"; done</automated>
  </verify>
  <done>
    All concept and architecture documents carry a `> **Last updated:**` metadata line. No MISSING output from the verify command.
  </done>
</task>

<task type="auto">
  <name>Task B.4: Document EDR-008/009 gap and TDR→EDR migration rationale</name>
  <files>
    docs/how/decision_records/INDEX.md
    docs/how/decision_records/process/EDR-001-edr-system.md
  </files>
  <action>
    Per DEBT-08: The EDR numbering has a gap at 008-009. The current note in INDEX.md says "EDR-008 and EDR-009 are intentionally skipped. TDR-007 and TDR-008 are superseded by EDR-001, not migrated." This needs expansion.

    Update INDEX.md:
    1. Expand the gap note to explain WHY they were skipped: the old TDR system numbered decisions 001-010; TDR-007 and TDR-008 covered areas now handled by EDR-001 (the EDR system itself). The gap preserves original TDR numbering so anyone mapping old TDR records to EDRs can see the correspondence.
    2. Add a note about the EDR-009 slot: "EDR-009 is reserved for future use if a decision needs to occupy the TDR-009 position in the mapping."
    3. Add a `## Gap Registry` subsection listing: EDR-008 (skipped — see note), EDR-009 (reserved), with rationale for each.

    Update EDR-001-edr-system.md:
    1. Add a new subsection or note in the Context section explaining the TDR→EDR migration:
       - Why TDRs were abandoned (dual-system confusion, need for unified journal)
       - How the mapping works (TDR-NNN → EDR-NNN correspondence where possible)
       - Which TDRs were migrated (TDR-001..006 → EDR-002..007), which were superseded (TDR-007/008 → EDR-001), and which were skipped (TDR-009 → EDR-010, TDR-010 → EDR-011)
    2. Add a mapping table: TDR-NNN → EDR-NNN for all 10 TDR records

    Also add date-stamp updates to both files.
  </action>
  <verify>
    <automated>grep -c 'Gap Registry' docs/how/decision_records/INDEX.md && grep -c 'TDR-007' docs/how/decision_records/process/EDR-001-edr-system.md</automated>
  </verify>
  <done>
    INDEX.md has an expanded gap note with Gap Registry. EDR-001.md documents the full TDR→EDR migration with a mapping table. Both have updated date stamps.
  </done>
</task>

<!-- ========================================================================== -->
<!-- WAVE C: Audits & Process Docs (depends on Waves A+B) — per D-04             -->
<!-- ========================================================================== -->

<task type="auto">
  <name>Task C.1: Audit all 22 concept docs against 8-section template and bring under-developed concepts to structural completion</name>
  <files>
    docs/how/concepts/research/EQUALITY.md
    docs/how/concepts/research/FUNCTIONS.md
    docs/how/concepts/research/MUTABILITY.md
    docs/how/concepts/research/ALLOCATION.md
    docs/how/concepts/research/DATA_MODEL.md
    docs/how/concepts/research/CORE_CONCEPTS.md
    docs/how/concepts/research/OWNERSHIP.md
    docs/how/concepts/research/ERROR_HANDLING.md
    docs/how/concepts/research/GENERICS.md
    docs/how/concepts/research/SPAN.md
    docs/how/concepts/research/METAOBJECTS.md
    docs/how/concepts/research/EXECUTION_PROGRAM.md
    docs/how/concepts/research/GENERATORS.md
    docs/how/concepts/research/LLM_NATIVE_TOOLCHAIN.md
    docs/how/concepts/research/UNPACKING.md
    docs/how/concepts/research/ASYNC_AWAIT.md
    docs/how/concepts/research/SORTING.md
    docs/how/concepts/research/LITERATE_PROGRAMMING.md
    docs/how/concepts/research/PATTERN_MATCHING.md
    docs/how/concepts/research/CONCURRENCY.md
    docs/how/concepts/research/OBJECT_INITIALIZATION.md
    docs/.planning/ROADMAP.md
    docs/.planning/codebase/CONCERNS.md
    docs/when/ROADMAP.md
    docs/AGENTS.md
  </files>
  <action>
    Per DEBT-11, DEBT-14, and D-03/D-05: Audit all 22 concept files against the 8-section _concept.md template (Issue, Principles, Policy Footprint, Model, Default Strategy, Alternative Strategies, Open Questions, Decision History). Produce a gap list file. Bring the 5 under-developed concepts (EQUALITY.md, FUNCTIONS.md, MUTABILITY.md, ALLOCATION.md, DATA_MODEL.md) to structurally complete drafts (all 8 sections present with real draft content, per D-05). Fix "7-section" → "8-section" errors across the project.

    Part 1 — Create the audit gap list:
    Create a new file at `docs/.planning/concept-audit-gaps.md` with a table row per concept:
    | Concept | Issue | Principles | Policy Footprint | Model | Default Strategy | Alt Strategies | Open Questions | Decision History | Status | Notes |

    For each concept, mark each section as ✅ (present with real content), ⚠️ (present but thin), ❌ (missing/empty), or — (template header only). Add a Status column: Complete / Under-developed / Remediation-planned.

    Part 2 — Fix "7-section" errors (per D-03):
    - `docs/.planning/ROADMAP.md`: Change "7-section" → "8-section" in success criterion 6 and any other occurrence
    - `docs/.planning/codebase/CONCERNS.md`: Change "7-section" → "8-section" in the Concept Depth section
    - `docs/.planning/REQUIREMENTS.md`: Change "7-section" → "8-section" in DEBT-11 and PROC-04
    - `docs/when/ROADMAP.md`: Change "7-section" → "8-section" in Epic 1 deliverables and Epic 2 process deliverables
    - `docs/AGENTS.md`: If any "7-section" reference exists, fix it
    - `docs/.planning/PHILOSOPHY.md` or similar: check and fix

    Part 3 — Bring under-developed concepts to structural completion (per D-05):
    For EQUALITY.md: fill all 8 sections with draft content. Issue: what equality problem exists in programming languages (reference vs. value equality, NaN handling, structural vs. identity). Principles: reference DESIGN_PRINCIPLES.md (Consistency, Explicitness). Policy Footprint: Equality Policy, Comparison Policy. Model: define equality as structural by default, `is` for identity, `===` for value equality. Default Strategy: structural recursion. Alternative Strategies: pointer equality, custom equality. Open Questions: floating-point NaN semantics, object identity for mutable data. Decision History: deferred to Phase 3 for full design.

    For FUNCTIONS.md: fill all 8 sections. Issue: function as the primary unit of computation. Principles: refer to DESIGN_PRINCIPLES.md (Named Before Symbolic, Explicitness). Policy Footprint: Evaluation Policy, Lifetime Policy. Model: first-class functions, closures, named and anonymous forms. Default Strategy: closure capture by reference with lifetime tracking. Alternative Strategies: closure copy, stack allocation. Open Questions: closure syntax, `fn` vs `λ` keyword, variadic parameters. Decision History: deferred to Phase 3.

    For MUTABILITY.md: fill all 8 sections. Issue: mutability tracking for correctness and optimization. Principles: Explicitness, Declarative With Static Guarantees. Policy Footprint: Mutability Policy, Lifetime Policy. Model: immutable-by-default, `mut` keyword for mutable bindings, ownership transfer for mutation. Default Strategy: borrow-checking with mutation permissions. Alternative Strategies: full mutability (no borrow checker), transactional memory. Open Questions: interior mutability, mutable references in closures. Decision History: deferred to Phase 3.

    For ALLOCATION.md: it already has Default Strategy and Alternative Strategies filled; add Issue (why allocation model matters for performance/predictability), Principles (Minimal Core, Explicitness), Policy Footprint (Allocation Policy, Lifetime Policy), Model (data is allocated per strategy, programmer doesn't specify allocation site), and Decision History (deferred to Phase 3).

    For DATA_MODEL.md: fill all 8 sections. This was flagged as a 13-17 line stub. Issue: what is the formal data model (values, types, representations). Principles: Data First, Orthogonality, Simplicity. Policy Footprint: Representation Policy, Allocation Policy. Model: define the type hierarchy, primitive types, compound types, representation system. Default Strategy: value semantics by default. Alternative Strategies: reference semantics, packed representations. Open Questions: tagged unions, sum types, type aliases. Decision History: deferred to Phase 3.
  </action>
  <verify>
    <automated>test -f docs/.planning/concept-audit-gaps.md && for f in EQUALITY FUNCTIONS MUTABILITY ALLOCATION DATA_MODEL; do grep -q '## Issue' "docs/how/concepts/research/${f}.md" || echo "MISSING Issue in $f"; grep -q '## Policy Footprint' "docs/how/concepts/research/${f}.md" || echo "MISSING Policy Footprint in $f"; grep -q '## Decision History' "docs/how/concepts/research/${f}.md" || echo "MISSING Decision History in $f"; done</automated>
  </verify>
  <done>
    Concept audit gap list created at docs/.planning/concept-audit-gaps.md. All 5 under-developed concepts have all 8 template sections with draft content. All "7-section" errors corrected across the project.
  </done>
</task>

<task type="auto">
  <name>Task C.2: Assign ownership and target-completion metadata to all tracked items</name>
  <files>
    docs/TODO.md
    docs/.planning/REQUIREMENTS.md
  </files>
  <action>
    Per DEBT-12: Every tracked TODO/requirement item must have an assigned owner and target-completion metadata, replacing unowned freeform checkboxes.

    In docs/TODO.md:
    1. For each unchecked TODO item, add an ownership field and target completion:
       - Owner: "Solo author" (this is a solo project per the project constraints)
       - Target: Phase identifier (e.g., "Phase 2", "Phase 3")
    2. Format each TODO with ownership metadata as a sub-bullet:
       ```
       - [ ] Item description
         - **Owner:** Solo author
         - **Target:** Phase 3
       ```
    3. If the TODO.md uses sections by milestone, add a header note: "All items owned by Solo Author unless otherwise noted."

    In docs/.planning/REQUIREMENTS.md:
    1. Verify all requirements already have Phase assignments (they do per the traceability table)
    2. Add a metadata note at the top: "All requirements owned by Solo Author."
    3. For requirements not yet assigned to a phase (if any), add an explicit ownership assignment

    Keep changes scoped to the phase assignment — don't redesign TODO.md structure.
  </action>
  <verify>
    <automated>grep -c 'Owner:' docs/TODO.md && grep -c 'Target:' docs/TODO.md</automated>
  </verify>
  <done>
    Every entry in TODO.md has Owner and Target metadata. REQUIREMENTS.md has an ownership provenance note.
  </done>
</task>

<task type="auto">
  <name>Task C.3: Perform repository-wide cross-reference link audit</name>
  <files>
    Multiple files across docs/ tree (as needed)
  </files>
  <action>
    Per DEBT-13: Perform a repository-wide cross-reference / "See also" link audit. Find and fix all broken or superseded links. Document the link-validation approach for future changes.

    Part 1 — Link audit:
    1. Grep for "See also:" patterns across all .md files in docs/
    2. Grep for markdown links `[text](path)` across docs/
    3. For each link found, verify the target file exists at the resolved relative path
    4. Check especially: links to EXECUTION_IMAGE.md (should now all point to EXECUTION_PROGRAM.md after B.2), links to _adr.md (should now point to _edr-architecture.md after B.1)
    5. Document all broken/missing links found

    Part 2 — Fix broken links:
    1. For each broken link found, update to the correct target
    2. If a target was renamed, update the link
    3. If a target was removed, either point to the archived version or the superseding document

    Part 3 — Document validation approach:
    Create a section in the audit log file (or in a new process doc) describing:
    - A recommended `markdown-link-check` or similar tool configuration
    - A manual link-check procedure for pre-commit review
    - A convention: "when renaming a file, search for all inbound links and update them in the same commit"

    Create the audit log at `docs/.planning/link-audit-log.md` with:
    - Date of audit
    - Total links checked
    - Links fixed (table: file, old link, new link)
    - Links still broken (if any remain after fixes, with rationale)
    - Validation approach documentation
  </action>
  <verify>
    <automated>test -f docs/.planning/link-audit-log.md && grep -q 'Links fixed' docs/.planning/link-audit-log.md</automated>
  </verify>
  <done>
    Link audit log at docs/.planning/link-audit-log.md documents all links checked, all fix actions taken, and a link-validation approach for future changes. Zero broken links remain in active documents.
  </done>
</task>

<task type="auto">
  <name>Task C.4: Document glossary maintenance workflow, versioning policy, and IA plan</name>
  <files>
    docs/what/GLOSSARY.md
    docs/how/VERSIONING_POLICY.md
    docs/how/IA_PLAN.md
  </files>
  <action>
    Per DEBT-15, DEBT-16, DEBT-17: Document three process policies.

    Part 1 — Glossary maintenance workflow (DEBT-15):
    The GLOSSARY.md already has a "How to Maintain This Glossary" section at the bottom with 4 rules. Expand it to include:
    - Trigger conditions: when a new concept document is created OR an existing concept introduces a term not in GLOSSARY.md
    - Process: (1) Identify new term, (2) Add to GLOSSARY.md in alphabetical section, (3) Add source document cross-reference, (4) Add "See also" links to related terms, (5) Update the concept document's Affected Documents checklist to include GLOSSARY.md
    - Review cadence: every Phase boundary (before starting a new phase, verify all new terms are registered)
    - Consistency check: verify no term has conflicting definitions across documents

    Part 2 — Versioning/change-log policy (DEBT-16):
    Create `docs/how/VERSIONING_POLICY.md` covering large monolithic documents:
    EXECUTION_PROGRAM.md, GLOSSARY.md, ARCHITECTURE.md, DESIGN_PRINCIPLES.md, AGENTS.md
    Policy should specify:
    - Each large document carries a change-log section at the bottom tracking: Date, Change description, Affected sections
    - Major revisions (semantic change): add a change-log entry and update the `> **Last updated:**` date stamp
    - Minor revisions (typo, formatting): update date stamp only, no change-log entry required
    - When a document undergoes significant structural change, the change-log entry should reference the EDR or phase that drove the change
    - Date-stamp freshness: any large document with last_updated > 90 days should be flagged for review

    Part 3 — Documentation growth/IA plan (DEBT-17):
    Create `docs/how/IA_PLAN.md` covering:
    - Current state: 22 concept files under how/concepts/research/, projected to grow to 30-35 by Phase 3 completion
    - Scaling strategy: keep how/concepts/research/ as flat directory; consider subdirectories only if >40 files
    - New top-level directories: future stdlib specs go in `docs/what/stdlib/`; future tooling specs in `docs/what/tooling/`; future LLM toolchain specs in `docs/what/llm/`
    - File naming: maintain UPPER_CASE.md convention for content documents
    - Cross-reference growth: as directory count grows, maintain a directory index file (INDEX.md) in each top-level directory listing all files with brief descriptions, pattern from how/decision_records/INDEX.md
    - Search strategy: for files >50, recommend a simple INDEX.md per directory rather than nesting
  </action>
  <verify>
    <automated>test -f docs/how/VERSIONING_POLICY.md && test -f docs/how/IA_PLAN.md && grep -q 'change-log' docs/how/VERSIONING_POLICY.md</automated>
  </verify>
  <done>
    GLOSSARY.md has an expanded maintenance workflow section. VERSIONING_POLICY.md documents the change-log and freshness policy for monolithic docs. IA_PLAN.md documents the growth and directory strategy. All three carry date-stamp metadata.
  </done>
</task>

</tasks>

<!-- ========================================================================== -->
<!-- THREAT MODEL                                                               -->
<!-- ========================================================================== -->

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| None | Documentation-only repository — no runtime, no API, no user data. All changes are reviewed via git commit. |

## STRIDE Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation Plan |
|-----------|----------|-----------|----------|-------------|-----------------|
| T-01-01 | Tampering | Content files | low | accept | Git-tracked repository; all changes visible in commit history. Solo author controls commits. No external contributors. |
| T-01-02 | Repudiation | Decision records | low | accept | EDRs are timestamped and committed to git; authorship is traceable. |
| T-01-SC | Tampering | npm/pip/cargo installs | low | n/a | No package manager installs in this phase — documentation-only repository. |
</threat_model>

<verification>
## Phase Verification

- [ ] All 4 arch specs (IR.md, PARSER.md, TYPE_SYSTEM.md, NAME_RESOLUTION.md) contain ≥200 words of real design specification each
- [ ] _adr.md moved to .archived/templates/; AGENTS.md references updated
- [ ] EXECUTION_IMAGE.md moved to .archived/concepts/; zero EXECUTION_IMAGE refs in active docs outside .planning/
- [ ] All 27 concept + architecture docs carry `> **Last updated:** 2026-07-20` metadata
- [ ] INDEX.md has expanded gap note with Gap Registry; EDR-001.md has TDR→EDR mapping table
- [ ] FITNESS_FUNCTIONS.md has ≥12 total fitness functions
- [ ] DOCUMENTATION_PRINCIPLES.md lists 8 sections (not 7) with ≥5 writing principles
- [ ] Concept audit gap list exists at docs/.planning/concept-audit-gaps.md
- [ ] EQUALITY.md, FUNCTIONS.md, MUTABILITY.md, ALLOCATION.md, DATA_MODEL.md all have all 8 template sections with draft content
- [ ] All "7-section" → "8-section" errors corrected in ROADMAP.md, CONCERNS.md, REQUIREMENTS.md, when/ROADMAP.md, DOCUMENTATION_PRINCIPLES.md
- [ ] TODO.md entries have Owner: and Target: metadata
- [ ] Link audit log at docs/.planning/link-audit-log.md
- [ ] GLOSSARY.md maintenance workflow expanded; VERSIONING_POLICY.md exists; IA_PLAN.md exists
</verification>

<success_criteria>
## Success Criteria (from ROADMAP.md)

1. ✅ IR.md, PARSER.md, TYPE_SYSTEM.md, NAME_RESOLUTION.md filled with real design content
2. ✅ _adr.md archived/removed, no live references
3. ✅ EXECUTION_IMAGE.md refs updated to EXECUTION_PROGRAM.md; concept archived
4. ✅ Date-stamp/freshness metadata on concept and architecture docs
5. ✅ EDR-008/009 gap documented; FITNESS_FUNCTIONS.md full catalogue; DOCUMENTATION_PRINCIPLES.md expanded
6. ✅ All 22 concept documents audited against 8-section template; under-developed ones brought to depth or carry remediation plan
7. ✅ Every tracked TODO/requirement has assigned owner and target-completion metadata
8. ✅ Repository-wide cross-reference link audit performed; broken links fixed
9. ✅ Glossary maintenance workflow, versioning/change-log policy, and documentation growth/IA plan documented

## Execution Order (per D-04: three-wave sequencing)

```
Wave A (independent — parallel):
  ├── Task A.1: Fill IR.md + PARSER.md (DEBT-01, DEBT-02)
  ├── Task A.2: Fill TYPE_SYSTEM.md + NAME_RESOLUTION.md (DEBT-03, DEBT-04)
  ├── Task A.3: Expand FITNESS_FUNCTIONS.md (DEBT-09)
  └── Task A.4: Expand DOCUMENTATION_PRINCIPLES.md (DEBT-10)

Wave B (after Wave A — sequential within wave):
  ├── Task B.1: Archive _adr.md (DEBT-05)
  ├── Task B.2: Archive EXECUTION_IMAGE.md (DEBT-06)
  ├── Task B.3: Add date stamps (DEBT-07)
  └── Task B.4: Document EDR gap (DEBT-08)

Wave C (after Waves A+B — sequential within wave):
  ├── Task C.1: Concept audit + structural completion (DEBT-11, DEBT-14)
  ├── Task C.2: Ownership assignment (DEBT-12)
  ├── Task C.3: Link audit (DEBT-13)
  └── Task C.4: Glossary workflow + versioning + IA plan (DEBT-15, DEBT-16, DEBT-17)
```
</success_criteria>

<output>
Create `.planning/phases/01-concerns-remediation/01-01-SUMMARY.md` when done
</output>
