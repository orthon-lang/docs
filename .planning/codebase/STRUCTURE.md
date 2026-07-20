# Codebase Structure

**Analysis Date:** 2026-07-20

## Directory Layout

```
/Users/mniedre/git/orthon-lang/docs/
├── README.md                              # Project overview and navigation
├── AGENTS.md                              # Agent protocol and document map
├── TODO.md                                # Open tasks and work items
├── LICENSE                                # MIT license
│
├── why/                                   # WHY layer — vision, philosophy, goals
│   ├── VISION.md                          # Five pillars and core inspiration
│   ├── GOALS.md                           # Six concrete aims with criteria
│   ├── MANIFESTO.md                       # Ten foundational principles
│   ├── WORKING_BACKWARDS.md               # Programmer perspective rationale
│   └── ZEN.md                             # Aphorisms capturing language spirit
│
├── what/                                  # WHAT layer — language specification
│   ├── DESIGN_PRINCIPLES.md               # 27 design rules organized in 3 groups
│   ├── GLOSSARY.md                        # Ubiquitous language — term definitions
│   ├── ROADMAP.md                         # Delivery timeline and feature phases
│   └── concepts/                          # Core language concepts (25+ files)
│       ├── CORE_CONCEPTS.md               # Data and Data Modifiers (foundational)
│       ├── DATA_MODEL.md                  # Formal data model specification
│       ├── ALLOCATION.md                  # Memory allocation and lifetime
│       ├── OWNERSHIP.md                   # Ownership and borrowing semantics
│       ├── MUTABILITY.md                  # Immutable-by-default design
│       ├── EQUALITY.md                    # Equality semantics and comparison
│       ├── FUNCTIONS.md                   # Function declarations and closures
│       ├── ASYNC_AWAIT.md                 # Asynchronous operations model
│       ├── CONCURRENCY.md                 # Concurrency model and threading
│       ├── GENERATORS.md                  # Generator and sequence production
│       ├── ERROR_HANDLING.md              # Error model and exception handling
│       ├── PATTERN_MATCHING.md            # Pattern matching expressions
│       ├── GENERICS.md                    # Generic type parameters
│       ├── OBJECT_INITIALIZATION.md       # Object construction and initialization
│       ├── EXECUTION_PROGRAM.md           # Execution Program / Engine separation
│       ├── EXECUTION_IMAGE.md             # Runtime image and deployment artifact
│       ├── LLM_NATIVE_TOOLCHAIN.md        # LLM-assisted development features
│       ├── LITERATE_PROGRAMMING.md        # Documentation-as-code support
│       ├── METAOBJECTS.md                 # Reflective/meta-object API
│       ├── SPAN.md                        # Memory span/pointer semantics
│       ├── UNPACKING.md                   # Tuple unpacking and destructuring
│       └── SORTING.md                     # Sorting and comparison semantics
│
├── how/                                   # HOW layer — design process and architecture
│   ├── PHILOSOPHY.md                      # Five-layer design flow diagram and explanation
│   ├── IMPLEMENTATION_POLICIES.md         # Policy types and their values
│   ├── DOCUMENTATION_PRINCIPLES.md        # Documentation writing standards
│   ├── PROCESS_INVENTORY.md               # Inventory of design processes
│   ├── concept-design-review.md           # Concept Design Review process
│   │
│   ├── architecture/                      # Compiler and runtime architecture
│   │   ├── ARCHITECTURE.md                # Layered architecture overview
│   │   ├── FITNESS_FUNCTIONS.md           # Measurable architectural checks
│   │   ├── ANALOGIES.md                   # Design analogies and inspirations
│   │   ├── PARSER.md                      # Source parsing and lexing
│   │   ├── TYPE_SYSTEM.md                 # Type checking and inference
│   │   ├── NAME_RESOLUTION.md             # Symbol resolution and scope
│   │   └── IR.md                          # Intermediate representation
│   │
│   ├── strategies/                        # Implementation strategy profiles
│   │   ├── IMPLEMENTATION_STRATEGIES.md   # How strategies work (meta)
│   │   ├── DEFAULT_STRATEGY.md            # Balanced, general-purpose strategy
│   │   ├── EMBEDDED_STRATEGY.md           # Minimal memory, predictable strategy
│   │   └── HIGH_PERFORMANCE_STRATEGY.md   # Throughput/latency optimized strategy
│   │
│   ├── decision_records/                  # Engineering Decision Records (EDRs)
│   │   ├── INDEX.md                       # Unified EDR journal and master index
│   │   ├── architecture/                  # Architecture category EDRs
│   │   │   ├── EDR-010-layered-architecture.md
│   │   │   └── EDR-011-llm-generability-gate.md
│   │   ├── process/                       # Process category EDRs (7 EDRs)
│   │   │   ├── EDR-001-edr-system.md
│   │   │   ├── EDR-002-gate-system.md
│   │   │   ├── EDR-003-validation-methods.md
│   │   │   ├── EDR-004-language-design-checklist.md
│   │   │   ├── EDR-006-policies-and-strategies.md
│   │   │   ├── EDR-007-concept-design-review.md
│   │   │   └── EDR-011-process-inventory.md
│   │   ├── quality/                       # Quality category EDRs
│   │   │   └── EDR-005-fitness-functions.md
│   │   ├── technology/                    # Technology category (reserved)
│   │   ├── tooling/                       # Tooling category (reserved)
│   │   ├── delivery/                      # Delivery category (reserved)
│   │   ├── operations/                    # Operations category (reserved)
│   │   ├── security/                      # Security category (reserved)
│   │   ├── governance/                    # Governance category (reserved)
│   │   ├── data/                          # Data category (reserved)
│   │   ├── ai/                            # AI category (reserved)
│   │   ├── documentation/                 # Documentation category (reserved)
│   │   ├── knowledge/                     # Knowledge category (reserved)
│   │   ├── collaboration/                 # Collaboration category (reserved)
│   │   └── product/                       # Product category (reserved)
│   │
│   ├── gates/                             # Design validation gates
│   │   ├── DECISION_VALIDATION.md         # Six independent validation gates
│   │   ├── _language-design.md            # Quality checklist for language decisions
│   │   └── methods/                       # Validation method documentation
│   │       ├── SCIENTIFIC_METHOD.md
│   │       ├── LOGICAL_ANALYSIS_METHOD.md
│   │       ├── SOCRATIC_METHOD.md
│   │       ├── TRIZ_METHOD.md
│   │       ├── EINSTEIN_METHOD.md
│   │       └── WORKING_BACKWARDS_METHOD.md
│   │
│   └── templates/                         # Fill-in templates (sorted first via _ prefix)
│       ├── _edr.md                        # Base EDR template
│       ├── _edr-architecture.md           # EDR Architecture category template
│       ├── _edr-process.md                # EDR Process category template
│       ├── _design-review.md              # Design review template
│       ├── _concept.md                    # Concept document template
│       └── _adr.md                        # ADR (Architecture Decision Record) template
│
├── when/                                  # WHEN layer — roadmap and timeline
│   └── ROADMAP.md                         # Delivery phases and milestone planning
│
├── notes/                                 # Research notes and explorations (22+ files)
│   ├── imperative-crutches-index.md       # Index of imperative programming patterns
│   ├── imperative-crutch-*.md             # Analysis of specific patterns (12 files)
│   │   ├── imperative-crutch-null-handling.md
│   │   ├── imperative-crutch-metaprogramming.md
│   │   ├── imperative-crutch-collection-init.md
│   │   ├── imperative-crutch-datetime.md
│   │   ├── imperative-crutch-collections-loops.md
│   │   ├── imperative-crutch-lazy-sequences.md
│   │   ├── imperative-crutch-serialization.md
│   │   ├── imperative-crutch-sorting.md
│   │   ├── imperative-crutch-type-casting.md
│   │   ├── imperative-crutch-resource-management.md
│   │   └── [more patterns as discovered]
│   ├── language-llm-comparison.md         # Comparative analysis of languages for LLM era
│   ├── llm-generability-gate.md           # Research on LLM code generation validation
│   ├── interaction-matrix-format.md       # Specification of interaction matrix format
│   ├── criteria-based-algorithm-selection.md  # Research on algorithm selection patterns
│   ├── whitepaper-strategy.md             # Whitepaper writing and publication strategy
│   └── [other research as conducted]
│
└── .planning/                             # Automation-generated analysis (git-ignored)
    └── codebase/                          # Codebase mapper output directory
        ├── ARCHITECTURE.md                # This file — system architecture
        └── STRUCTURE.md                   # Directory layout and file organization
```

## Directory Purposes

### Root Level

**README.md** — Project entry point for humans. Contains:
- Project overview and key concepts
- "Getting Started" table linking to essential documents
- "For AI Agents" section pointing to AGENTS.md

**AGENTS.md** — Project entry point for AI agents. Contains:
- Project identity and constraints
- Complete documentation framework (Why → How → What)
- Full document map with layer assignments
- Agent workflow protocol (Orient → Design → Gate → Write)
- Terminology and ubiquitous language rules
- Design decision protocol
- Proposal structure and contribution patterns

**TODO.md** — Work tracking. Contains:
- Open tasks and improvements
- Known gaps in documentation
- Future work items (not yet scheduled to a milestone)

### `why/` — Vision and Philosophy Layer

**Purpose:** Answer the foundational question — *why does Orthon exist?*

**Contains:** Five documents totaling ~12,000 words

- `VISION.md` — Five pillars: (1) Language Designed Like Software (SOLID principles), (2) Learn from What Came Before (Python + Java inspiration), (3) Comfortable by Design (Principle of Least Astonishment), (4) Designed for the LLM Era, (5) Execution Program / Engine separation model
- `GOALS.md` — Six concrete aims with measurable criteria: Architectural Integrity, Inherited Wisdom, Comfortable by Construction, LLM Readiness, DevOps Transform, Implementation Independence
- `MANIFESTO.md` — Ten foundational principles: Consistency over legacy, One concept = one syntax, Semantics over syntax, Declarative over implicit, No privileged features, Extensibility over magic, Composition over exceptions, Unified declaration, Minimal core, Transparency
- `WORKING_BACKWARDS.md` — Amazon-style working backwards: What problem does Orthon solve? What does a programmer want? Why should they care?
- `ZEN.md` — Aphorisms capturing language spirit (e.g., "Explicit is better than implicit", "One responsibility per construct")

**Key property:** These documents are *prescriptive* — they define what we believe, not what we're considering

### `what/` — Language Specification Layer

**Purpose:** Define *what* the language concretely is

**Contains:** Three root documents + 25+ concept documents

**Root documents:**
- `DESIGN_PRINCIPLES.md` (27 principles in 3 groups: Core Philosophy, Language Consistency, Execution Model) — translates Vision into rules all proposals must follow
- `GLOSSARY.md` (18,000+ words) — ubiquitous language and terminology reference with cross-document links
- `ROADMAP.md` — feature delivery phases and milestone planning

**Concepts directory** (`concepts/`):
- Each concept document defines one language feature
- Documents start as DRAFT, pass Concept Design Review, then become ACCEPTED (recorded via EDR in Architecture category)
- ~25 concept files including: CORE_CONCEPTS, DATA_MODEL, ALLOCATION, OWNERSHIP, FUNCTIONS, ERROR_HANDLING, etc.
- Each concept file contains:
  - Semantic model (formal definition)
  - Motivation and rationale
  - Examples (canonical forms)
  - Interactions with other concepts
  - Policy Footprint (which Implementation Policies it affects)
  - Validation against gates
  - Links to EDR of acceptance

**Key property:** These are *specifications*, not proposals. Everything here is either accepted or explicitly marked DRAFT

### `how/` — Design Process and Implementation Architecture

**Purpose:** Define *how* design decisions are made and *how* they're implemented

**Contains:** 50+ files organized in 6 subdirectories

**Root files:**
- `PHILOSOPHY.md` — Visualizes the five-layer design flow (Vision → Principles → Validation → Gate → Concept/Strategy) and explains each layer's output
- `IMPLEMENTATION_POLICIES.md` — Defines Policy types (Allocation, Algorithm, Evaluation, Lifetime, Concurrency, etc.) and their possible values; explains the declarative principle
- `DOCUMENTATION_PRINCIPLES.md` — Standards for documentation writing (precision, consistency, authority)
- `PROCESS_INVENTORY.md` — Catalog of design processes used in the project
- `concept-design-review.md` — Formal review process for new concepts before EDR acceptance

**architecture/** subdirectory (7 files):
- `ARCHITECTURE.md` — Layered architecture (Language layer, Implementation Strategy layer, Execution Environment layer) with dependency flow
- `FITNESS_FUNCTIONS.md` — Measurable checks guarding against architectural decay
- `ANALOGIES.md` — Design analogies and inspirations
- `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`, `IR.md` — Compiler component specifications

**strategies/** subdirectory (4 files):
- `IMPLEMENTATION_STRATEGIES.md` — Meta: how strategies work, relationship to policies
- `DEFAULT_STRATEGY.md` — Balanced, general-purpose strategy (balanced memory/performance)
- `EMBEDDED_STRATEGY.md` — Minimal memory, predictable timing strategy
- `HIGH_PERFORMANCE_STRATEGY.md` — Throughput/latency optimized strategy

Each strategy file specifies Policy values for: Allocation, Algorithm, Evaluation, Lifetime, Concurrency, etc.

**decision_records/** subdirectory (15+ files):
- `INDEX.md` — Unified journal of all EDRs with status, date, supersedes relationships
- `architecture/` — Architecture-category EDRs (2 files: EDR-010, EDR-011)
- `process/` — Process-category EDRs (7 files: EDR-001, EDR-002, EDR-003, EDR-004, EDR-006, EDR-007, EDR-011)
- `quality/` — Quality-category EDRs (1 file: EDR-005)
- `technology/`, `tooling/`, `delivery/`, `operations/`, `security/`, `governance/`, `data/`, `ai/`, `documentation/`, `knowledge/`, `collaboration/`, `product/` — Reserved for future EDRs

**gates/** subdirectory (9 files):
- `DECISION_VALIDATION.md` — Six independent validation gates: Correctness, Implementation Independence, Orthogonality, Learnability, LLM Generability, Principle Alignment
- `_language-design.md` — Quality checklist (gate) for language design decisions; specifies what every design proposal must answer
- `methods/` — Documentation of validation methods (6 methods: Scientific, Logical Analysis, Socratic, TRIZ, Einstein, Working Backwards)

**templates/** subdirectory (6 files):
- `_edr.md` — Base template for creating new EDRs
- `_edr-architecture.md` — EDR template for Architecture category
- `_edr-process.md` — EDR template for Process category
- `_design-review.md` — Template for Concept Design Review
- `_concept.md` — Template for new concept documents
- `_adr.md` — ADR (Architecture Decision Record) template

### `when/` — Timeline and Roadmap

**Purpose:** Define *when* features are delivered

**Contains:** One file

- `ROADMAP.md` (13,500 words) — Delivery phases organized by milestone:
  - Milestone 0 (Foundation) — Core concepts, architecture, process
  - Milestone 1 — Language semantics specification
  - Milestone 2 — Concept design review and LLM gate implementation
  - Milestone 3+ — Parser, type system, strategies, implementation

### `notes/` — Research and Exploration

**Purpose:** Capture research, explorations, and analysis that informs design but isn't yet policy

**Contains:** 22+ files organized by topic

**Imperative Crutches** (12 files analyzing alternatives to imperative programming):
- `imperative-crutches-index.md` — Index of all patterns
- `imperative-crutch-null-handling.md` — Null handling alternatives
- `imperative-crutch-metaprogramming.md` — Metaprogramming patterns
- `imperative-crutch-collection-init.md`, `collections-loops.md`, `lazy-sequences.md`, `datetime.md`, `serialization.md`, `sorting.md`, `type-casting.md`, `resource-management.md` — Domain-specific patterns

**Language and LLM Research** (3 files):
- `language-llm-comparison.md` (18,000+ words) — Comparative analysis of language features for LLM-era suitability
- `llm-generability-gate.md` — Research on what makes constructs LLM-generizable
- `whitepaper-strategy.md` — Publication strategy for language whitepaper

**Design Patterns and Methodology** (3 files):
- `interaction-matrix-format.md` — Specification of matrix format for showing feature interactions
- `criteria-based-algorithm-selection.md` — Research on algorithm selection patterns

**Key property:** Notes are exploratory and may not result in policy. They're marked as such in content

## Key File Locations

### Entry Points

| Document | Path | Purpose |
|----------|------|---------|
| **README.md** | `/Users/mniedre/git/orthon-lang/docs/README.md` | Human navigation; getting started guide |
| **AGENTS.md** | `/Users/mniedre/git/orthon-lang/docs/AGENTS.md` | AI agent navigation; protocol and framework |
| **PHILOSOPHY.md** | `/Users/mniedre/git/orthon-lang/docs/how/PHILOSOPHY.md` | Understanding design flow; entry to How layer |

### Foundation (Why Layer)

| Document | Path | Purpose |
|----------|------|---------|
| **VISION.md** | `why/VISION.md` | Five pillars and core beliefs |
| **GOALS.md** | `why/GOALS.md` | Six concrete aims with criteria |
| **MANIFESTO.md** | `why/MANIFESTO.md` | Ten foundational principles |

### Framework (How Layer)

| Document | Path | Purpose |
|----------|------|---------|
| **DESIGN_PRINCIPLES.md** | `what/DESIGN_PRINCIPLES.md` | 27 rules translating vision to criteria |
| **PHILOSOPHY.md** | `how/PHILOSOPHY.md` | Five-layer design flow diagram |
| **IMPLEMENTATION_POLICIES.md** | `how/IMPLEMENTATION_POLICIES.md` | Policy types and their relationship to strategies |
| **DECISION_VALIDATION.md** | `how/gates/DECISION_VALIDATION.md` | Six independent validation gates |
| **_language-design.md** | `how/gates/_language-design.md` | Quality checklist for language decisions |

### Core Concepts (What Layer)

| Document | Path | Purpose |
|----------|------|---------|
| **CORE_CONCEPTS.md** | `what/concepts/CORE_CONCEPTS.md` | Data and Data Modifiers (foundational) |
| **DATA_MODEL.md** | `what/concepts/DATA_MODEL.md` | Formal data model specification |
| **GLOSSARY.md** | `what/GLOSSARY.md` | Ubiquitous language and terminology |

### Decision Records

| Document | Path | Purpose |
|----------|------|---------|
| **INDEX.md** | `how/decision_records/INDEX.md` | Master index of all EDRs; status summary |
| **EDR-010** | `how/decision_records/architecture/EDR-010-layered-architecture.md` | Layered architecture decision |
| **EDR-001** | `how/decision_records/process/EDR-001-edr-system.md` | EDR system decision |

### Implementation Strategies

| Document | Path | Purpose |
|----------|------|---------|
| **IMPLEMENTATION_STRATEGIES.md** | `how/strategies/IMPLEMENTATION_STRATEGIES.md` | How strategies work; meta information |
| **DEFAULT_STRATEGY.md** | `how/strategies/DEFAULT_STRATEGY.md` | Balanced, general-purpose strategy policies |
| **EMBEDDED_STRATEGY.md** | `how/strategies/EMBEDDED_STRATEGY.md` | Minimal-memory strategy policies |
| **HIGH_PERFORMANCE_STRATEGY.md** | `how/strategies/HIGH_PERFORMANCE_STRATEGY.md` | Optimized strategy policies |

### Roadmap & Timeline

| Document | Path | Purpose |
|----------|------|---------|
| **ROADMAP.md** | `when/ROADMAP.md` | Delivery phases and milestone planning |

## Naming Conventions

### Content Documents

**Files:** `UPPER_CASE.md` with single coherent topic

Examples:
- `VISION.md` — content file (not `_vision.md` or `Vision.md`)
- `CORE_CONCEPTS.md` — one topic per file (not "CONCEPTS_AND_FEATURES.md")
- `IMPLEMENTATION_POLICIES.md` — specific topic (not "POLICIES_AND_STUFF.md")

**Rule:** If you're tempted to name a file "MISCELLANEOUS" or "VARIOUS", split it into multiple focused files

### Directories

**Directories:** `lowercase`, plural nouns

Examples:
- `why/`, `what/`, `how/`, `when/` — main layers
- `concepts/`, `strategies/`, `decision_records/`, `gates/`, `templates/`, `architecture/` — category directories
- `process/`, `quality/`, `architecture/` — subcategories within `decision_records/`

**Rule:** Directory names are lowercase; document names are UPPERCASE

### Templates and Checklists

**Files:** `_kebab-case.md` with leading underscore

Examples:
- `_edr.md` — template (underscore sorts first)
- `_language-design.md` — gate checklist (underscore sorts first)
- `_design-review.md` — review template (underscore sorts first)

**Rule:** Templates and forms get `_` prefix so they appear first in directory listings. Once filled in, the underscore is dropped (e.g., `EDR-010-layered-architecture.md`)

### Engineering Decision Records

**Files:** `EDR-{NNN}-{title-with-dashes}.md` inside `decision_records/{category}/`

Examples:
- `decision_records/architecture/EDR-010-layered-architecture.md`
- `decision_records/process/EDR-001-edr-system.md`
- `decision_records/quality/EDR-005-fitness-functions.md`

**Rule:** EDR IDs are globally unique; format is `EDR-NNN` where NNN is zero-padded integer. Category subdirectory organizes by decision type

### Concept Documents

**Files:** `{CONCEPT_NAME}.md` inside `what/concepts/`

Examples:
- `what/concepts/CORE_CONCEPTS.md`
- `what/concepts/DATA_MODEL.md`
- `what/concepts/ERROR_HANDLING.md`

**Rule:** One concept per file; name is the concept's primary identifier

## Where to Add New Code

### New Design Principle (rare)

**Location:** `what/DESIGN_PRINCIPLES.md` — append to appropriate section (Core Philosophy, Language Consistency, or Execution Model)

**Process:**
1. Add principle to DESIGN_PRINCIPLES.md
2. Update AGENTS.md §3 Document Map if this creates new category
3. If consequential change to philosophy → create EDR in `how/decision_records/process/`

### New Language Concept

**Location:** `what/concepts/{CONCEPT_NAME}.md` (new file)

**Process:**
1. Create concept file from template `how/templates/_concept.md`
2. Mark as DRAFT at top of document
3. Specify:
   - Semantic model (formal definition)
   - Motivation (why it's needed)
   - Examples (canonical forms)
   - Policy Footprint (which Implementation Policies it affects)
   - Validation against `how/gates/_language-design.md` checklist
4. Add concept to GLOSSARY.md when terms are introduced
5. Submit for Concept Design Review (see `how/concept-design-review.md`)
6. After approval: Create EDR in `how/decision_records/architecture/` to formalize acceptance
7. Update `how/strategies/{STRATEGY}.md` files to specify how each strategy implements this concept

### New Implementation Policy

**Location:** `how/IMPLEMENTATION_POLICIES.md` — add to Policy Catalogue section

**Process:**
1. Add Policy type and possible values to `how/IMPLEMENTATION_POLICIES.md`
2. Link to related concepts in `what/concepts/`
3. Update each strategy file (`DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, etc.) to specify values for this new Policy
4. If consequential → create EDR in `how/decision_records/process/`

### New Implementation Strategy

**Location:** `how/strategies/{STRATEGY_NAME}.md` (new file)

**Process:**
1. Create strategy file (or copy `DEFAULT_STRATEGY.md` as template)
2. Document:
   - Purpose (what this strategy optimizes for)
   - Target platforms/scenarios
   - Policy values (Allocation, Algorithm, Evaluation, Lifetime, Concurrency, etc.)
   - Tradeoffs (what it gains, what it sacrifices)
   - Which concepts it fully/partially supports
   - Links to architecture files (`how/architecture/`) detailing concrete implementation
3. Update `how/strategies/IMPLEMENTATION_STRATEGIES.md` to reference new strategy
4. Consider creating EDR in `how/decision_records/process/` if strategy introduces new design policy

### New Architecture Component

**Location:** `how/architecture/{COMPONENT}.md` (new file) if describing compiler/runtime architecture

**Process:**
1. Analyze which layer/component needs documentation
2. Determine if this belongs in `how/architecture/` (compiler/runtime) or `what/concepts/` (language feature)
3. If `how/architecture/`: Document component's responsibility, interfaces, dependency relationships
4. Link to related concepts in `what/concepts/`
5. Link to strategy files showing how each strategy implements this component
6. If consequential architectural decision → create EDR in `how/decision_records/architecture/`

### New Engineering Decision Record (EDR)

**Location:** `how/decision_records/{CATEGORY}/{EDR-NNN-title}.md` (new file)

**Process:**
1. Determine category: Architecture, Process, Quality, Technology, Tooling, Delivery, Operations, Security, Governance, Data, AI, Documentation, Knowledge, Collaboration, Product
2. Find next available ID number in category (see `how/decision_records/INDEX.md`)
3. Create file from appropriate template:
   - `how/templates/_edr.md` (base template)
   - `how/templates/_edr-architecture.md` (Architecture category)
   - `how/templates/_edr-process.md` (Process category)
4. Fill in: Context, Decision, Rationale, Consequences, Alternatives, Related Concepts, Supersedes
5. Update `how/decision_records/INDEX.md` to add entry to master table
6. Add link to decision record in relevant content documents (e.g., if EDR-042 describes concept X, link from `what/concepts/X.md`)

### New Research Note

**Location:** `notes/{TOPIC}.md` (new file)

**Process:**
1. Only create if exploratory or informational (not policy)
2. Mark document status clearly (e.g., "⚠️ EXPLORATORY" or "📋 RESEARCH")
3. If research becomes policy → promote to appropriate layer (why/, what/, how/) with EDR
4. If multiple related notes → create index file (e.g., `notes/imperative-crutches-index.md`)

### New Validation Method

**Location:** `how/gates/methods/{METHOD_NAME}.md` (new file)

**Process:**
1. Reference `how/gates/DECISION_VALIDATION.md` to see existing gates
2. Determine if new method is needed (unlikely; six methods cover most validation needs)
3. If creating new method:
   - Document approach with examples
   - Show how it validates language design decisions
   - Link from `how/gates/DECISION_VALIDATION.md`

### New Documentation Rule or Style Standard

**Location:** 
- Style/tone rules → `how/DOCUMENTATION_PRINCIPLES.md`
- Process rules → `how/PROCESS_INVENTORY.md`
- Agent protocol changes → `AGENTS.md`

**Process:**
1. Identify which document owns the rule
2. Add rule with clear example
3. If affecting multiple documents, update AGENTS.md as well
4. If substantial change → consider EDR in `how/decision_records/process/`

### Update GLOSSARY

**Location:** `what/GLOSSARY.md` (append to alphabetical list)

**Process:**
1. When introducing new term in ANY document:
   - Add entry to GLOSSARY.md in alphabetical order
   - Include: term, definition, context, link to where it's defined
2. Link from term back to defining document
3. If term affects multiple layers, note all relevant files

## Special Directories

### `.planning/` — Automation Output (git-ignored)

**Purpose:** Generated by automation tools; not authored by hand

**Committed:** No (see .gitignore)

**Generated by:** Codebase analysis tools, CI/CD scripts, etc.

**Contents:**
- `codebase/` — Output from `/gsd-map-codebase` analysis
  - `ARCHITECTURE.md` — System architecture (this file)
  - `STRUCTURE.md` — Directory layout and naming (this file)
  - Future: STACK.md, INTEGRATIONS.md, etc. for other focus areas

**Behavior:** Regenerated on demand; never edited directly. Git ignores all files in this directory.

### `.git/` — Version Control (standard git)

**Purpose:** Git repository metadata

**Committed:** Yes (as `.git/` directory)

**Generated by:** `git init` and git operations

### `how/templates/` — Fill-in Forms (reference, not authored each time)

**Purpose:** Reusable templates for creating new documents

**Committed:** Yes

**Key files:**
- `_edr.md` — Base EDR template (copy and fill)
- `_edr-architecture.md` — Architecture EDR template
- `_edr-process.md` — Process EDR template
- `_design-review.md` — Design review template
- `_concept.md` — Concept document template
- `_adr.md` — ADR template

**Naming rule:** Leading underscore ensures templates sort first in directory listings

**Usage:** Copy template, rename (drop underscore), fill in content

### `how/decision_records/` — EDR Archive (write-once, never modify)

**Purpose:** Permanent record of design decisions

**Committed:** Yes

**Key rule:** EDRs are write-once. Once accepted, they're never deleted or edited (except for typos/clarifications, marked via superseding EDR if substantial). This maintains decision history integrity.

**Structure:** Categories as subdirectories (architecture/, process/, quality/, etc.)

---

*Structure analysis: 2026-07-20*
