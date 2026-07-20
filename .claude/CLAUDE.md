<!-- GSD:project-start source:PROJECT.md -->

## Project

**Orthon Language Specification**

Orthon is a programming language designed using the same engineering principles it encourages in its users — orthogonal core, explicit semantics, SOLID architecture, and every consequential design choice recorded as an Engineering Decision Record (EDR). This repository is documentation-only: there is no compiler or runtime here. The project's output is a complete, self-consistent specification of Orthon v0.1, ready to hand off to a separate implementation repository.

Orthon targets two audiences simultaneously: humans (via orthogonality, deliberate design informed by prior languages, and logical evolution) and LLMs (via simplicity, minimal unnecessary invariants, and purpose-built LLM tooling). It also introduces a new DevOps model — the **Execution Program** — where a program's semantic meaning is decoupled from its execution strategy, so the same artifact can be interpreted, compiled, containerized, or built for WASM without modification.

**Core Value:** A complete, self-consistent Orthon v0.1 specification — every concept accepted (no DRAFT), core architecture specs (IR, Parser, Type System, Name Resolution) filled in, all cross-references valid, and a final review confirming the result coheres as a real language specification.

### Constraints

- **Scope**: Documentation/specification only — no compiler, runtime, or implementation code in this repository (that's a future, separate repo — Milestone 10).
- **Process**: Every consequential design decision must be recorded as an EDR with context, rationale, and alternatives considered.
- **Design philosophy**: Every language construct must pass the LLM Generability Gate (validates whether an LLM can reliably produce correct code using it) in addition to human-facing design review.
- **Ownership**: Solo-authored — no multi-contributor review/approval workflow required, but self-imposed rigor (Concept Design Review, acceptance gates) still applies.

<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->

## Technology Stack

## Languages

- Markdown - All documentation content (91 `.md` files, 2.8M total)
- YAML (implicit in `.gitignore` patterns) - Version control configuration

## Runtime

- Git-based documentation repository — no runtime execution
- Git — local and remote (`origin` → GitHub)

## Frameworks

- None — pure Markdown files without build framework (no mkdocs, Sphinx, Hugo, or similar)
- None — documentation is static Markdown intended for direct GitHub browsing or manual processing

## Key Dependencies

## Configuration

- `.git/` — Standard Git repository
- `.gitignore` — Python-focused patterns (legacy; no Python tooling present in repo)
- GitHub remote: `git@github.com:orthon-lang/docs.git`

## Documentation Structure

- Why → How → What (Golden Circle framework)
- Directories: `why/`, `what/`, `how/`, `when/`, `notes/`, `.planning/`
- See `AGENTS.md` for complete directory mapping
- `README.md` — Project overview and getting started
- `AGENTS.md` — AI agent contribution protocol and document map
- `why/VISION.md` — Core philosophy and pillars
- `how/decision_records/INDEX.md` — Engineering Decision Records journal

## Platform Requirements

- Text editor with Markdown support
- Git client (any version)
- No build tools, compilers, or runtimes required
- GitHub web interface (for browsing)
- Standard Markdown renderer
- Optional: local git clone for offline access

## Notes

- **No build artifacts** — Repository contains only source Markdown files
- **No external tooling dependencies** — Documentation is self-contained
- **Purpose:** Design documentation for the Orthon programming language (language design only; no compiler/runtime code present)

<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

## Naming Patterns

- Content documents: `UPPER_CASE.md` — one file, one coherent topic
- Template files: `_kebab-case.md` with underscore prefix inside `how/templates/`
- Gate checklist files: `_kebab-case.md` with underscore prefix inside `how/gates/`
- Decision records: `EDR-NNN-title-with-dashes.md` inside `decision_records/{category}/`
- Lowercase, plural nouns: `why/`, `what/`, `how/`, `when/`, `decision_records/`, `templates/`, `gates/`, `concepts/`, `strategies/`, `architecture/`
- Do not nest directories deeper than `docs/{layer}/{category}/` unless explicitly justified
- Agent instructions: `AGENTS.md` at repository root (`docs/AGENTS.md`)
- Master EDR index: `how/decision_records/INDEX.md`
- Decision validation framework: `how/gates/DECISION_VALIDATION.md`
- Project root reference: `README.md` at repository root

## Code Style

- Use standard Markdown syntax throughout
- Fenced code blocks with language identifier for syntax highlighting
- Level 1 (`#`): Document title only, appears once at top
- Level 2 (`##`): Major sections within document
- Level 3 (`###`): Subsections
- Level 4 (`####`): Detailed breakdown within subsections
- Keep consistent structure within document types (concept docs, EDRs, etc.)
- Use markdown tables for structured data comparison
- Include pipe separators and dashes for alignment
- Example from codebase: category/lens comparison tables in DECISION_VALIDATION.md
- Use unordered lists (`-`) for general items
- Use ordered lists (`1.`, `2.`, etc.) for sequential steps or priorities
- Use nested lists with consistent indentation for hierarchical information

## Document Status Indicators

- Location: Top of document, before main heading
- Format: `> **⚠️ DRAFT — This document is a preliminary draft.`
- Content block includes:
- Field: **Status:** [Proposed | Accepted | Deprecated | Superseded by EDR-NNN]
- Appears in header metadata

## Cross-References and Linking

- Links use markdown syntax: `[Link text](relative/path.md)`
- Paths relative to source document's location
- Paths to `concepts/` subdirectory must use prefix: `concepts/CORE_CONCEPTS.md` not `CORE_CONCEPTS.md`
- Cross-document links must resolve to existing files in repository
- Each entry includes:

### Core Language

- **Source:** `../how/architecture/ARCHITECTURE.md` § Core Language, `DESIGN_PRINCIPLES.md` § Minimal Core
- **See also:** [Architecture](#architecture), [Implementation Strategy](#implementation-strategy)

## Import/Cross-Reference Organization

- Content organized by **Why → How → What** (Golden Circle) framework
- "Why" layer: purpose, vision, philosophy — `why/VISION.md`, `why/MANIFESTO.md`, `why/ZEN.md`, `why/GOALS.md`
- "How" layer: design and implementation — `how/IMPLEMENTATION_POLICIES.md`, `how/architecture/ARCHITECTURE.md`, `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- "What" layer: concrete language spec — `what/CORE_CONCEPTS.md`, `what/concepts/` subdirectory
- "When" layer: roadmap and milestones — `when/ROADMAP.md`
- An agent must always anchor new content to the correct layer
- "Why" arguments must not be smuggled into "What" documents
- If documentation spans layers, split it or add explicit cross-references
- Each document references its primary layer and related documents
- Concept documents include checklist at end indicating which documents may need updates
- Example from `_concept.md` template:

## Error Handling and Validation Notes

- Used for small decisions within sections
- Format:
- Used for decisions requiring review in `how/gates/_language-design.md`
- Includes status and criteria checklist
- Used for consequential engineering decisions
- Includes: Status, Date, Category, Scope, Context, Decision, Consequences, Compliance, Alternatives

## Comments and Documentation

- Explain design intent, not implementation details
- Document non-obvious connections between sections
- Mark unresolved issues, open questions
- Note deprecation or rejection of alternatives
- Not applicable — this is a documentation codebase, not a code codebase
- Code examples in documentation are illustrative only
- Concept documents follow strict template from `how/templates/_concept.md`
- All sections present in order: Issue (Why), Principles, Policy Footprint, Model (What), Default Strategy, Alternative Strategies, Open Questions, Decision History
- EDRs follow template from `how/templates/_edr.md` or category-specific variants
- Design reviews follow template from `how/templates/_design-review.md`

## Function/Concept Design

- When documenting a language feature, present **all canonical forms**, not just the most common
- All equivalent syntactic forms shown together in same section
- Example from CORE_CONCEPTS.md: `emit value`, `return sequence(value)`, `return value ->` are shown as equivalent
- Code examples precede semantic explanation (not the reverse)
- Syntax and semantics section describes ideal model before implementation
- Default Strategy describes how feature is implemented by default
- Alternative Strategies list permitted implementation variants with trade-offs

## Module/Document Design

- Do not create files titled "Miscellaneous" or "Various"
- Each document covers exactly one coherent topic
- Modularity achieved through cross-references, not consolidation
- Not applicable to documentation
- INDEX.md serves as unified journal/master index (e.g., `decision_records/INDEX.md`)
- Concept documents declare their scope via template sections
- Policy Footprint section lists involved Policy Types
- EDRs declare Category and Scope fields

## Language Requirement

- All documentation, code snippets, comments, commit messages, and agent reasoning **MUST be in English**
- Non-negotiable project rule
- Enforcement: Before creating or modifying any file, agent MUST confirm every string is English
- If user request is in another language, output is translated to English
- Violation of this rule is a document validation failure

## Terminology Consistency

- All project terminology defined in `what/GLOSSARY.md`
- Terms used consistently in defined meaning throughout codebase
- New terms added to GLOSSARY.md with cross-reference to source document
- **Data** — primary abstraction; values without imposed semantic meaning
- **Data Modifier** — transforms data between representations
- **Representation** — specific view of data (Value, Tuple, Reference, Sequence, Set, Option, Result)
- **Sequence** — values produced over time; describes *what*, not *how*
- **Orthogonality** — each construct solves one problem and combines freely with others
- **Core Language** — minimal, stable set of language semantics, independent of implementation
- **Implementation Strategy** — named set of Policies fulfilling Core Language contracts
- **Policy** — implementation-level decision (e.g., Allocation Policy, Lifetime Policy, Mutability Policy)

<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

## System Overview

```text

```

## Component Responsibilities

| Component | Location | Responsibility | Key Files |
|-----------|----------|-----------------|-----------|
| **Vision & Philosophy** | `why/` | Define *why* the language exists, what we believe, what we're trying to achieve | `VISION.md`, `GOALS.md`, `MANIFESTO.md`, `WORKING_BACKWARDS.md`, `ZEN.md` |
| **Design Framework & Process** | `how/` | Define *how* design decisions are made, validated, and recorded; provide implementation guidance | `PHILOSOPHY.md`, `IMPLEMENTATION_POLICIES.md`, `decision_records/`, `gates/`, `strategies/` |
| **Language Specification** | `what/` | Define *what* the language concretely is — data model, core concepts, features | `DESIGN_PRINCIPLES.md`, `concepts/`, `GLOSSARY.md` |
| **Implementation Strategies** | `how/strategies/` | Specify *how* language semantics are implemented across different execution environments (default, embedded, high-performance) | `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md` |
| **Roadmap & Governance** | `when/` | Define *when* features are implemented, milestones, and delivery timeline | `ROADMAP.md` |

## Pattern Overview

- **Five-layer design flow:** Vision → Core Principles → Decision Validation → Language Design Gate → Concept/Strategy
- **Explicit decision recording:** Every consequential design choice is documented as an Engineering Decision Record (EDR)
- **Cross-layer linking:** Documents in different layers reference each other explicitly via markdown links, creating a traceable decision graph
- **Ubiquitous language:** `what/GLOSSARY.md` provides single source of truth for terminology across all layers
- **Template-driven:** Standardized formats for EDRs, concepts, design reviews, and strategies ensure consistency

## Layers

### Layer 1: Vision (`why/`)

- `VISION.md` — Five pillars: Architectural Integrity, Inherited Wisdom, Comfortable by Construction, LLM Readiness, Execution Program Model
- `GOALS.md` — Six concrete aims derived from vision, each with acceptance criteria
- `MANIFESTO.md` — Ten fundamental principles: consistency over legacy, one concept per syntax, minimal core, etc.
- `WORKING_BACKWARDS.md` — Programmer-perspective rationale for why Orthon exists
- `ZEN.md` — Aphorisms capturing the language spirit

### Layer 2: Core Principles (`what/DESIGN_PRINCIPLES.md` + `why/MANIFESTO.md`)

- **DESIGN_PRINCIPLES.md:** Three groups of rules (Core Philosophy, Language Consistency, Execution Model)
- **MANIFESTO.md:** Ten foundational agreements (consistency over legacy, minimal core, composition over exceptions, etc.)

### Layer 3: Decision Validation (`how/gates/`)

- `DECISION_VALIDATION.md` — Six independent gates (Correctness, Implementation Independence, Orthogonality, Learnability, LLM Generability, Principle Alignment)
- `_language-design.md` — Quality gate checklist for language design decisions
- `methods/` — Validation methods (Scientific Method, Logical Analysis, Socratic Method, TRIZ, Einstein Method, Working Backwards Method)

### Layer 4: Language Design Gate (`how/gates/_language-design.md`)

- Checklist of questions every design proposal must answer
- Policy footprint determination (which Implementation Policies the concept affects)
- Semantic specification requirements
- LLM generability verification

### Layer 5: Concepts & Strategies

- Purpose: Formal semantic definition of language features
- Location: `what/concepts/{FEATURE_NAME}.md`
- Pattern: Each concept starts as DRAFT, passes Concept Design Review, then becomes accepted via EDR (Architecture category)
- Examples: `CORE_CONCEPTS.md`, `DATA_MODEL.md`, `FUNCTIONS.md`, `ERROR_HANDLING.md`
- Purpose: Specify *how* language semantics are implemented for different execution environments
- Location: `how/strategies/{STRATEGY_NAME}.md`
- Pattern: Strategy = named set of Policies; Policies are declarative preferences (Allocation, Algorithm, Evaluation, Lifetime, Concurrency, etc.)
- Examples: `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md`
- Purpose: Specify implementation architecture (compiler, runtime structure)
- Location: `how/architecture/{COMPONENT}.md`
- Contains: `ARCHITECTURE.md` (layered architecture), `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`, `IR.md`

## Data Flow

### Primary Design Flow

### Secondary Flows

- New terms introduced in any concept document must be added to `what/GLOSSARY.md`
- GLOSSARY.md provides ubiquitous language across all layers
- Cross-references from GLOSSARY back to defining documents
- Every EDR (`how/decision_records/{category}/EDR-*.md`) links to:
- Each Implementation Strategy in `how/strategies/` references:

## Key Abstractions

### Engineering Decision Records (EDRs)

- **ID:** EDR-NNN (globally unique)
- **Title:** One-line decision statement
- **Date:** When decision was made
- **Status:** Accepted / Proposed / Deprecated / Superseded
- **Context:** Problem/opportunity description
- **Decision:** The choice made and why
- **Consequences:** Positive and negative impacts
- **Alternatives:** What was considered and why rejected
- **Related Concepts:** Links to `what/concepts/` docs
- **Supersedes:** Links to prior EDRs this replaces

### Concepts

- **Overview:** One-paragraph summary
- **Motivation:** Why this concept is needed
- **Semantic Model:** Formal definition of what it means
- **Examples:** Canonical forms and use cases
- **Interactions:** How it composes with other concepts
- **Policy Footprint:** Which Implementation Policies it affects
- **Gates:** Validation against Language Design Gate checklist
- Vision/Goals it serves
- Design Principles it embodies
- Related concepts
- Implementation Strategy handling
- EDR of acceptance decision

### Implementation Strategies & Policies

```

```

- `DEFAULT_STRATEGY.md` — Balanced, general-purpose
- `EMBEDDED_STRATEGY.md` — Minimal memory, predictable timing
- `HIGH_PERFORMANCE_STRATEGY.md` — Throughput/latency optimized

### Design Review

## Entry Points

- Location: Project root
- Triggers: AI agent contribution workflow
- Responsibilities: 
- Location: Project root
- Triggers: Human visitor orientation
- Responsibilities:
- Location: `how/` layer
- Triggers: Understanding design decision flow
- Responsibilities:

## Architectural Constraints

- **Single-responsibility per file:** Each document addresses one coherent topic. Files titled "Miscellaneous" or "Various" are prohibited
- **Layer integrity:** Why → How → What layers must not be mixed. A Why argument cannot appear in a What document
- **Explicit linking:** Cross-document references must be explicit markdown links, not vague mentions
- **No circular dependencies:** Downstream layers (How, What) may reference upstream layers (Why, How), but not vice versa
- **Naming consistency:** All document names are UPPER_CASE, all directories are lowercase
- **Template prefix convention:** Templates and checklists use `_` prefix (e.g., `_edr.md`, `_language-design.md`) to sort first in listings
- **EDR precedence:** Consequential decisions must be recorded as EDRs before implementation begins; proposals without EDRs are unrecorded experiments
- **Glossary as source of truth:** All terminology is centralized; new terms introduced in any document must be added to `what/GLOSSARY.md`

## Anti-Patterns

### Anti-Pattern: Why → What Smuggling

- Philosophical rationale → `why/MANIFESTO.md` or `why/VISION.md`
- Concrete specification → `what/concepts/` file
- Link between them: Concept document says "See why/MANIFESTO.md §X for rationale"

### Anti-Pattern: Undocumented Design Decisions

- If decision is consequential → create EDR in `how/decision_records/{category}/`
- If decision is small → record as inline gate entry in relevant document
- If decision is exploratory → document in `notes/` with clear "exploratory" marker

### Anti-Pattern: Isolated Concept Design

- Every concept document includes:

### Anti-Pattern: Strategy-Specific Language Semantics

- Specify semantics in strategy-independent terms: "values have managed lifetimes"
- Let IMPLEMENTATION_POLICIES decide *how*: Allocation Policy = GC vs. RC vs. Arena
- Concept document includes Policy Footprint section showing which policies affect it

## Cross-Cutting Concerns

- `what/DESIGN_PRINCIPLES.md` — checks consistency principle adherence
- `how/gates/_language-design.md` — checklist item for representation symmetry
- EDRs reference DESIGN_PRINCIPLES they satisfy, creating traceability
- `what/GLOSSARY.md` — single source of truth for all defined terms
- AGENTS.md §6 — defines terminology rules
- Every new term introduced in any document must be added to GLOSSARY
- Cross-references from GLOSSARY back to defining documents
- `how/gates/DECISION_VALIDATION.md` — six independent validation gates all proposals must pass
- `how/gates/_language-design.md` — language-design-specific quality checklist
- Design Review process (`how/concept-design-review.md`) — formal review before EDR acceptance
- EDR linking: Each EDR links to vision/goals it serves, principles it satisfies, alternatives considered
- Concept linking: Each concept references its EDR, validation gates, related concepts
- Strategy linking: Each strategy references which concepts it implements, policy values, architecture files
- Glossary linking: Each term links back to defining document

<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:

- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
