# Testing Patterns

**Analysis Date:** 2026-07-20

## Test Framework

**Testing approach:**
- No automated unit tests or integration tests in repository
- No test runner configured (Jest, Vitest, Mocha, pytest, etc.)
- No test directory structure
- Codebase is **design documentation only** — no implementation code to unit test

**Quality assurance method:**
- Validation through **Decision Validation Gates** framework (primary testing mechanism)
- Manual peer review using structured checklists
- Template-based enforcement of document structure
- Cross-document consistency validation through references

## Validation and Quality Gates

**Decision Validation Gates:**
- Located: `how/gates/DECISION_VALIDATION.md`
- **Seven independent gates** for language design decisions:
  1. `USER_VALUE_GATE` — Does this solve a real programmer problem?
  2. `LOGICAL_CONSISTENCY_GATE` — Is the proposal internally consistent?
  3. `CONCEPTUAL_SIMPLICITY_GATE` — Is it as simple as possible?
  4. `ARCHITECTURAL_INTEGRITY_GATE` — Does it fit the layered architecture?
  5. `IMPLEMENTATION_INDEPENDENCE_GATE` — Can it be defined without tying to strategy?
  6. `LONG_TERM_MAINTAINABILITY_GATE` — Will this decision age well?
  7. `LLM_GENERABILITY_GATE` — Can LLMs reliably produce code using this?

**Gate evaluation methods:**
- `USER_VALUE_GATE` uses Working Backwards method
- `LOGICAL_CONSISTENCY_GATE` uses Socratic Method
- `CONCEPTUAL_SIMPLICITY_GATE` uses Scientific Method
- `ARCHITECTURAL_INTEGRITY_GATE` uses Logical Analysis
- `IMPLEMENTATION_INDEPENDENCE_GATE` uses TRIZ
- `LONG_TERM_MAINTAINABILITY_GATE` uses Einstein's Method
- `LLM_GENERABILITY_GATE` uses Empirical Analysis (structural analysis + schema round-trip)

**Gate selection by decision type:**
| Decision Type | Required Gates | Optional Gates |
|---------------|---|---|
| New language construct | All seven | — |
| Syntax change | LOGICAL_CONSISTENCY, CONCEPTUAL_SIMPLICITY, ARCHITECTURAL_INTEGRITY, LLM_GENERABILITY | USER_VALUE (if syntactic sugar), IMPLEMENTATION_INDEPENDENCE, LONG_TERM_MAINTAINABILITY |
| LLM Toolchain component | ARCHITECTURAL_INTEGRITY, LLM_GENERABILITY, USER_VALUE | CONCEPTUAL_SIMPLICITY, LOGICAL_CONSISTENCY, IMPLEMENTATION_INDEPENDENCE, LONG_TERM_MAINTAINABILITY |
| Semantic refinement | LOGICAL_CONSISTENCY, ARCHITECTURAL_INTEGRITY | CONCEPTUAL_SIMPLICITY (if narrowing), LLM_GENERABILITY |
| Standard Library addition | USER_VALUE, CONCEPTUAL_SIMPLICITY, LONG_TERM_MAINTAINABILITY | ARCHITECTURAL_INTEGRITY (if within Library), IMPLEMENTATION_INDEPENDENCE, LLM_GENERABILITY |
| Compiler optimisation | IMPLEMENTATION_INDEPENDENCE | All others (optimisations do not change semantics) |

**Gate entry format:**
- Location: `how/gates/_language-design.md` (and other gate files)
- Structure:
  ```markdown
  ### Gate NNN: [Feature Name]
  
  **Status:** [Pending | Passed | Failed]
  
  **Criteria:**
  - [ ] Criterion 1
  - [ ] Criterion 2
  - [ ] Criterion 3
  ```

**Gate compliance verification:**
- Gates produce binary verdict: pass / fail
- Unresolved flags also block finalization
- Any proposal with fail or unresolved flag must be revised before specification

## Test/Validation File Organization

**Location:**
- Gate checklists: `how/gates/` directory
- Validation methods: `how/gates/methods/` subdirectory
- EDR index and records: `how/decision_records/` and subdirectories
- Templates for validation: `how/templates/` directory

**Naming:**
- Gate checklists: `_kebab-case.md` (underscore prefix)
- Filled-in gates/decisions: no prefix (content, not template)
- Method documents: `*_METHOD.md` in `methods/` subdirectory

## Validation Checklist

**Pre-finalization review checklist (from AGENTS.md §9):**
- [ ] **Consistency:** Does this align with `why/MANIFESTO.md` and `what/DESIGN_PRINCIPLES.md`?
- [ ] **Orthogonality:** Does this combine freely with existing constructs?
- [ ] **Minimality:** Is this adding a new concept when composition would suffice?
- [ ] **Explicitness:** Are semantic changes syntactically visible?
- [ ] **EDR:** Does this decision warrant an EDR?
- [ ] **All canonical forms:** Are all equivalent forms documented?
- [ ] **English (MANDATORY):** Every string in English? (non-negotiable)
- [ ] **Terminology:** Project terms used consistently with `what/GLOSSARY.md`?
- [ ] **Gate:** Does the decision have a gate entry if needed?
- [ ] **Cross-references:** Are related documents linked?

**Layer-based validation (from AGENTS.md §5.1):**
1. Read relevant layer first (Why/How/What)
2. Check cross-references for design decisions affecting other layers
3. Check `how/gates/_language-design.md` if decision involves design
4. Verify against Design Philosophy in `how/PHILOSOPHY.md`
5. Verify against VISION and MANIFESTO if contradicting

## Design Review Process

**Design Review template:**
- Location: `how/templates/_design-review.md`
- Used for proposals warranting formal peer review
- Structured review form for substantive design changes

**Concept Design Review process:**
- All concept documents in `docs/how/concepts/research/` follow uniform template from `how/templates/_concept.md`
- Concepts created as DRAFT during Milestone 0/1
- Formal review occurs during Milestone 2 through Concept Design Review process
- Concept registration requires acceptance via EDR (Architecture category)
- Sections required in order:
  1. Issue (Why) — what problem does this solve?
  2. Principles — which principles cannot be violated?
  3. Policy Footprint — which Policy Types does this involve?
  4. Model (What) — what does the language concept look like?
  5. Default Strategy — how is it implemented by default?
  6. Alternative Strategies — what implementation variants are permitted?
  7. Open Questions — what remains unresolved?
  8. Decision History — why were alternatives rejected?

## Engineering Decision Records (EDRs)

**EDR system:**
- Location: `how/decision_records/` with category subdirectories
- Master index: `how/decision_records/INDEX.md`
- Unified system recording all consequential engineering decisions

**EDR categories (15 lenses):**
- Architecture, Technology, Tooling, Process, Delivery, Operations, Quality, Security, Governance, Data, AI, Documentation, Knowledge, Collaboration, Product

**EDR base template:**
- Location: `how/templates/_edr.md`
- Required fields:
  - **EDR-NNN:** Global sequential ID
  - **Status:** Proposed | Accepted | Deprecated | Superseded by EDR-NNN
  - **Date:** YYYY-MM-DD
  - **Category:** One of 15 categories
  - **Scope:** Platform | Subsystem | Module | Team | Project | Policy
  - **Context:** What problem prompted this decision?
  - **Decision:** What did we decide?
  - **Consequences:** Positive and negative impacts
  - **Compliance:** How will we verify adherence?
  - **Alternatives Considered:** Table format

**EDR category-specific templates:**
- `_edr-architecture.md` — adds Architecture-specific sections
- `_edr-process.md` — adds "Without It" (risk analysis) and "Evolution" (deprecation criteria) sections

**EDR compliance verification:**
- Compliance field specifies how adherence is verified:
  - Code review checklist item
  - Gate entry (e.g., `how/gates/_language-design.md`)
  - Automated lint rule (if applicable)
  - Documentation convention
  - Fitness function (if architectural)

**EDR numbering:**
- Global sequential numbering: EDR-001, EDR-002, etc.
- Single sequence across all categories
- ID does not encode category (Category field + directory placement serve that purpose)

## Inline Decision Notation

**Small decision format (for section-level decisions):**
- Location: End of relevant section
- Format:
  ```markdown
  > **Decision:** [What was decided]
  > **Rationale:** [Why this choice]
  > **Alternatives considered:** [What was rejected and why]
  ```
- Examples from codebase (FOUNDATIONAL_ABSTRACTIONS.md, etc.)

## Template Enforcement

**Concept document enforcement:**
- All concept files follow template from `how/templates/_concept.md`
- Template includes section ordering and required headings
- Affected Documents checklist at end (with checkboxes)
- Documents start with DRAFT status indicator if not formally accepted

**Other template enforcement:**
- EDR documents follow EDR template structure
- Design reviews follow design review template structure
- Gate entries follow gate checklist format
- Document Map (AGENTS.md §3) lists all documents with their purposes

**Template validation:**
- Manual review against template structure
- Cross-reference validation (links must exist)
- Terminology consistency check against GLOSSARY.md
- Layer validation (Why/How/What alignment)

## Cross-Document Consistency

**Cross-reference verification:**
- All markdown links must resolve to existing files
- Paths relative to source document location
- Paths to `concepts/` subdirectory must use prefix: `concepts/CORE_CONCEPTS.md`
- Bidirectional reference validation:
  - If Document A links to Document B, Document B should reference back or cross-reference A

**Glossary consistency:**
- All project-specific terminology must be defined in `what/GLOSSARY.md`
- Terms must be used consistently throughout codebase in their defined meaning
- New terms require glossary entry with cross-reference to source document

**Layer coherence:**
- "Why" documents (vision, manifesto, goals) establish principles and motivation
- "How" documents (design principles, architecture, strategies) flow from Why
- "What" documents (concepts, glossary) flow from both Why and How
- "When" documents (roadmap) aligned with all three

**EDR validation:**
- All EDRs reference and align with VISION, MANIFESTO, and DESIGN_PRINCIPLES
- EDRs cross-reference related EDRs and related documents
- Superseded EDRs linked to superseding EDRs

## Coverage and Completeness

**Coverage requirements:**
- No explicit code coverage targets (not applicable to documentation)
- Design coverage: all language concepts documented via concept docs
- All consequential decisions recorded in EDR system
- All terminology unified in GLOSSARY.md

**Completeness validation:**
- Template checklist enforcement (all required sections present)
- Affected Documents checklist (identifies which docs need updating)
- Master INDEX.md in decision_records directory (all EDRs catalogued)
- Document Map in AGENTS.md (all content files listed)

**Gap identification:**
- "Open Questions" section in concept documents marks unresolved issues
- TODO.md file at repository root tracks work items
- EDR system provides visibility into delayed decisions

## Common Validation Patterns

**Orthogonality validation:**
- Verify new construct combines freely with existing constructs
- No special cases or context-dependent syntax
- Check against DESIGN_PRINCIPLES.md § Orthogonality

**Explicitness validation:**
- Verify semantic changes are syntactically visible
- No silent type conversions or implicit behavior
- Check against DESIGN_PRINCIPLES.md § Explicit Semantics

**Minimality validation:**
- Verify concept cannot be expressed through composition of existing concepts
- Burden of proof on new concept proposer
- CONCEPTUAL_SIMPLICITY_GATE validates this

**LLM generability validation:**
- Empirical analysis: can LLM reliably produce correct code?
- Structural analysis: does feature reduce prediction space or expand it?
- Schema round-trip: can feature be serialized and deserialized consistently?
- Separate gate (`LLM_GENERABILITY_GATE`) validates this

## Quality Assurance Processes

**Agent workflow validation (from AGENTS.md §5-9):**
1. **Orient** — Read relevant layer, check cross-references, check gates
2. **Design** — Start from first principles, favor minimal additions, document alternatives, preserve orthogonality
3. **Gate** — Verify language compliance (English), verify against gate checklist
4. **Write** — Place in correct location, follow structure, use Markdown, write in English
5. **Review** — Full pre-finalization checklist before document is considered complete

**Read-first requirement:**
- Agent must read existing documents affected by proposed changes before proceeding
- No change proposals without understanding existing context

**One intention per commit (git):**
- Commits grouped by topic, not by file
- Enables traceability of design decisions to specific changes

**Gate before merge:**
- Any design decision must pass appropriate gates before finalization
- Unresolved gates block completion

## No Automated Testing

**Important note:**
- This is a design-documentation-only repository
- No automated tests, linters, or formatters configured
- No CI/CD pipeline for testing code changes
- No build system or runtime to test
- Quality assurance entirely through manual review, structured templates, and validation gates

---

*Testing analysis: 2026-07-20*
