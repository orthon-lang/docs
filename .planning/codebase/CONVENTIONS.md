# Coding Conventions

**Analysis Date:** 2026-07-20

## Naming Patterns

**Files:**
- Content documents: `UPPER_CASE.md` — one file, one coherent topic
  - Examples: `VISION.md`, `CORE_CONCEPTS.md`, `DESIGN_PRINCIPLES.md`, `GLOSSARY.md`
  - Location: Content files live inside their layer directory (`why/`, `what/`, `how/`, `when/`)
  
- Template files: `_kebab-case.md` with underscore prefix inside `how/templates/`
  - Examples: `_edr.md`, `_concept.md`, `_design-review.md`, `_edr-architecture.md`
  - Underscore prefix ensures templates sort first in directory listings
  
- Gate checklist files: `_kebab-case.md` with underscore prefix inside `how/gates/`
  - Examples: `_language-design.md`
  
- Decision records: `EDR-NNN-title-with-dashes.md` inside `decision_records/{category}/`
  - Example: `how/decision_records/architecture/EDR-010-layered-architecture.md`
  - Global sequential numbering (EDR-001, EDR-002, etc.) across all categories
  - Stored in per-category subdirectories: `architecture/`, `process/`, `quality/`, `technology/`, `tooling/`, `delivery/`, `operations/`, `security/`, `governance/`, `data/`, `ai/`, `documentation/`, `knowledge/`, `collaboration/`, `product/`

**Directories:**
- Lowercase, plural nouns: `why/`, `what/`, `how/`, `when/`, `decision_records/`, `templates/`, `gates/`, `concepts/`, `strategies/`, `architecture/`
- Do not nest directories deeper than `docs/{layer}/{category}/` unless explicitly justified

**Special Files:**
- Agent instructions: `AGENTS.md` at repository root (`docs/AGENTS.md`)
- Master EDR index: `how/decision_records/INDEX.md`
- Decision validation framework: `how/gates/DECISION_VALIDATION.md`
- Project root reference: `README.md` at repository root

## Code Style

**Markdown formatting:**
- Use standard Markdown syntax throughout
- Fenced code blocks with language identifier for syntax highlighting
  - Orthon code: use `orthon` as language tag
  - Structural diagrams: use `text` language tag
  - Non-Orthon code: use language tag (e.g., `python`, `bash`)

**Heading hierarchy:**
- Level 1 (`#`): Document title only, appears once at top
- Level 2 (`##`): Major sections within document
- Level 3 (`###`): Subsections
- Level 4 (`####`): Detailed breakdown within subsections
- Keep consistent structure within document types (concept docs, EDRs, etc.)

**Tables:**
- Use markdown tables for structured data comparison
- Include pipe separators and dashes for alignment
- Example from codebase: category/lens comparison tables in DECISION_VALIDATION.md

**Lists:**
- Use unordered lists (`-`) for general items
- Use ordered lists (`1.`, `2.`, etc.) for sequential steps or priorities
- Use nested lists with consistent indentation for hierarchical information

## Document Status Indicators

**Draft status marker:**
- Location: Top of document, before main heading
- Format: `> **⚠️ DRAFT — This document is a preliminary draft.`
- Content block includes:
  - Creation milestone (e.g., "Milestone 0 (Foundation)")
  - Purpose (e.g., "exploratory work")
  - Review process name (e.g., "Concept Design Review process")
  - Review milestone (e.g., "Milestone 2")
  - Acceptance criteria (e.g., "acceptance via EDR")
  
Example from `CORE_CONCEPTS.md`:
```
> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
```

**EDR status field:**
- Field: **Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]
- Appears in header metadata

## Cross-References and Linking

**Relative path format:**
- Links use markdown syntax: `[Link text](relative/path.md)`
- Paths relative to source document's location
- Paths to `concepts/` subdirectory must use prefix: `concepts/CORE_CONCEPTS.md` not `CORE_CONCEPTS.md`
- Cross-document links must resolve to existing files in repository

**Glossary entries:**
- Each entry includes:
  - **Definition:** Clear, single-paragraph explanation
  - **Source:** Link to originating document, often with section reference (e.g., `DESIGN_PRINCIPLES.md § Minimal Core`)
  - **See also:** Related terminology entries (internal links to other glossary entries with `#anchor` syntax)

**GLOSSARY.md link format example:**
```markdown
### Core Language

The minimal, stable set of language semantics...

- **Source:** `../how/architecture/ARCHITECTURE.md` § Core Language, `DESIGN_PRINCIPLES.md` § Minimal Core
- **See also:** [Architecture](#architecture), [Implementation Strategy](#implementation-strategy)
```

## Import/Cross-Reference Organization

**Layer-based organization:**
- Content organized by **Why → How → What** (Golden Circle) framework
- "Why" layer: purpose, vision, philosophy — `why/VISION.md`, `why/MANIFESTO.md`, `why/ZEN.md`, `why/GOALS.md`
- "How" layer: design and implementation — `how/IMPLEMENTATION_POLICIES.md`, `how/architecture/ARCHITECTURE.md`, `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- "What" layer: concrete language spec — `what/CORE_CONCEPTS.md`, `what/concepts/` subdirectory
- "When" layer: roadmap and milestones — `when/ROADMAP.md`

**Cross-layer validation:**
- An agent must always anchor new content to the correct layer
- "Why" arguments must not be smuggled into "What" documents
- If documentation spans layers, split it or add explicit cross-references
- Each document references its primary layer and related documents

**Affected Documents checklist:**
- Concept documents include checklist at end indicating which documents may need updates
- Example from `_concept.md` template:
  ```markdown
  ### Affected Documents
  
  - [ ] `CORE_CONCEPTS.md`
  - [ ] `DATA_MODEL.md`
  - [ ] `DESIGN_PRINCIPLES.md`
  - [ ] `GLOSSARY.md`
  - [ ] `IMPLEMENTATION_STRATEGIES.md`
  - [ ] `IMPLEMENTATION_POLICIES.md`
  - [ ] Other: ...
  ```

## Error Handling and Validation Notes

**Decision notation (inline):**
- Used for small decisions within sections
- Format:
  ```markdown
  > **Decision:** [What was decided]
  > **Rationale:** [Why this choice]
  > **Alternatives considered:** [What was rejected and why]
  ```

**Gate entry format:**
- Used for decisions requiring review in `how/gates/_language-design.md`
- Includes status and criteria checklist

**EDR format:**
- Used for consequential engineering decisions
- Includes: Status, Date, Category, Scope, Context, Decision, Consequences, Compliance, Alternatives

## Comments and Documentation

**When to comment:**
- Explain design intent, not implementation details
- Document non-obvious connections between sections
- Mark unresolved issues, open questions
- Note deprecation or rejection of alternatives

**JSDoc/TSDoc:**
- Not applicable — this is a documentation codebase, not a code codebase
- Code examples in documentation are illustrative only

**Template-based sections:**
- Concept documents follow strict template from `how/templates/_concept.md`
- All sections present in order: Issue (Why), Principles, Policy Footprint, Model (What), Default Strategy, Alternative Strategies, Open Questions, Decision History
- EDRs follow template from `how/templates/_edr.md` or category-specific variants
- Design reviews follow template from `how/templates/_design-review.md`

## Function/Concept Design

**Canonicality requirement:**
- When documenting a language feature, present **all canonical forms**, not just the most common
- All equivalent syntactic forms shown together in same section
- Example from CORE_CONCEPTS.md: `emit value`, `return sequence(value)`, `return value ->` are shown as equivalent

**Model/Semantics first:**
- Code examples precede semantic explanation (not the reverse)
- Syntax and semantics section describes ideal model before implementation
- Default Strategy describes how feature is implemented by default
- Alternative Strategies list permitted implementation variants with trade-offs

## Module/Document Design

**One file, one topic:**
- Do not create files titled "Miscellaneous" or "Various"
- Each document covers exactly one coherent topic
- Modularity achieved through cross-references, not consolidation

**Exports/Barrel structure:**
- Not applicable to documentation
- INDEX.md serves as unified journal/master index (e.g., `decision_records/INDEX.md`)

**Type declarations for content:**
- Concept documents declare their scope via template sections
- Policy Footprint section lists involved Policy Types
- EDRs declare Category and Scope fields

## Language Requirement

**Mandatory language enforcement:**
- All documentation, code snippets, comments, commit messages, and agent reasoning **MUST be in English**
- Non-negotiable project rule
- Enforcement: Before creating or modifying any file, agent MUST confirm every string is English
- If user request is in another language, output is translated to English
- Violation of this rule is a document validation failure

## Terminology Consistency

**Ubiquitous Language:**
- All project terminology defined in `what/GLOSSARY.md`
- Terms used consistently in defined meaning throughout codebase
- New terms added to GLOSSARY.md with cross-reference to source document

**Key terms reference:**
- **Data** — primary abstraction; values without imposed semantic meaning
- **Data Modifier** — transforms data between representations
- **Representation** — specific view of data (Value, Tuple, Reference, Sequence, Set, Option, Result)
- **Sequence** — values produced over time; describes *what*, not *how*
- **Orthogonality** — each construct solves one problem and combines freely with others
- **Core Language** — minimal, stable set of language semantics, independent of implementation
- **Implementation Strategy** — named set of Policies fulfilling Core Language contracts
- **Policy** — implementation-level decision (e.g., Allocation Policy, Lifetime Policy, Mutability Policy)

---

*Convention analysis: 2026-07-20*
