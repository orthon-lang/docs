<!-- refreshed: 2026-07-20 -->
# Architecture

**Analysis Date:** 2026-07-20

## System Overview

The Orthon documentation codebase is a **design knowledge base** organized as a layered, multi-perspective system that guides language design decisions. It implements the **Why → How → What** framework — a golden circle model where each layer answers a distinct design question and feeds into the next.

The documentation serves as the single source of truth for language design, capturing not just *what* decisions were made, but *why* they were made, *how* they fit into the larger architecture, and *what* they mean concretely.

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        User / Agent                                  │
│                    (Reading, Contributing)                           │
└─────────────────────┬───────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────┐
│                     AGENTS.md (Navigation Hub)                       │
│              Entry point: protocol, framework, document map          │
└─────────────────────┬───────────────────────────────────────────────┘
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
│ WHY Layer    │ │ HOW Layer    │ │ WHAT Layer       │
│ (Vision)     │ │ (Design)     │ │ (Definition)     │
│ `why/`       │ │ `how/`       │ │ `what/`          │
│              │ │              │ │                  │
│ ·VISION.md   │ │ ·PHILOSOPHY  │ │ ·DESIGN_         │
│ ·GOALS.md    │ │  .md         │ │  PRINCIPLES.md   │
│ ·MANIFESTO   │ │ ·IMPL_       │ │ ·GLOSSARY.md     │
│  .md         │ │  POLICIES.md │ │ ·concepts/       │
│ ·WORKING_    │ │ ·strategies/ │ │  (CORE, DATA,    │
│  BACKWARDS   │ │ ·gates/      │ │   EQUALITY,      │
│  .md         │ │ ·decision_   │ │   etc.)          │
│ ·ZEN.md      │ │  records/    │ │ ·ROADMAP.md      │
│              │ │ ·templates/  │ │ (in `when/`)     │
│              │ │ ·architecture│ │                  │
└──────────────┘ └──────────────┘ └──────────────────┘
       │              │              │
       └──────────────┼──────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────┐
│                      GLOSSARY.md (Ubiquitous Language)               │
│                   Unified terminology reference                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

The documentation is organized into **five major components**, each with a distinct responsibility in the design process:

| Component | Location | Responsibility | Key Files |
|-----------|----------|-----------------|-----------|
| **Vision & Philosophy** | `why/` | Define *why* the language exists, what we believe, what we're trying to achieve | `VISION.md`, `GOALS.md`, `MANIFESTO.md`, `WORKING_BACKWARDS.md`, `ZEN.md` |
| **Design Framework & Process** | `how/` | Define *how* design decisions are made, validated, and recorded; provide implementation guidance | `PHILOSOPHY.md`, `IMPLEMENTATION_POLICIES.md`, `decision_records/`, `gates/`, `strategies/` |
| **Language Specification** | `what/` | Define *what* the language concretely is — data model, core concepts, features | `DESIGN_PRINCIPLES.md`, `concepts/`, `GLOSSARY.md` |
| **Implementation Strategies** | `how/strategies/` | Specify *how* language semantics are implemented across different execution environments (default, embedded, high-performance) | `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md` |
| **Roadmap & Governance** | `when/` | Define *when* features are implemented, milestones, and delivery timeline | `ROADMAP.md` |

## Pattern Overview

**Overall Pattern:** Layered, multi-perspective design system with explicit traceability

**Key Characteristics:**
- **Five-layer design flow:** Vision → Core Principles → Decision Validation → Language Design Gate → Concept/Strategy
- **Explicit decision recording:** Every consequential design choice is documented as an Engineering Decision Record (EDR)
- **Cross-layer linking:** Documents in different layers reference each other explicitly via markdown links, creating a traceable decision graph
- **Ubiquitous language:** `what/GLOSSARY.md` provides single source of truth for terminology across all layers
- **Template-driven:** Standardized formats for EDRs, concepts, design reviews, and strategies ensure consistency

## Layers

### Layer 1: Vision (`why/`)

**Purpose:** Answer the foundational question — why does Orthon exist? What do we believe? What are we trying to achieve?

**Location:** `why/`

**Contains:**
- `VISION.md` — Five pillars: Architectural Integrity, Inherited Wisdom, Comfortable by Construction, LLM Readiness, Execution Program Model
- `GOALS.md` — Six concrete aims derived from vision, each with acceptance criteria
- `MANIFESTO.md` — Ten fundamental principles: consistency over legacy, one concept per syntax, minimal core, etc.
- `WORKING_BACKWARDS.md` — Programmer-perspective rationale for why Orthon exists
- `ZEN.md` — Aphorisms capturing the language spirit

**Depends on:** Nothing (foundational layer)

**Used by:** `what/DESIGN_PRINCIPLES.md`, `how/PHILOSOPHY.md`, all downstream layers

**Cross-references:** Every design document in `how/` and `what/` traces back to specific sections in `why/VISION.md` and `why/GOALS.md`

### Layer 2: Core Principles (`what/DESIGN_PRINCIPLES.md` + `why/MANIFESTO.md`)

**Purpose:** Translate vision into concrete, verifiable rules that can guide design decisions

**Location:** `what/DESIGN_PRINCIPLES.md`, `why/MANIFESTO.md`

**Contains:**
- **DESIGN_PRINCIPLES.md:** Three groups of rules (Core Philosophy, Language Consistency, Execution Model)
- **MANIFESTO.md:** Ten foundational agreements (consistency over legacy, minimal core, composition over exceptions, etc.)

**Depends on:** `why/VISION.md`, `why/GOALS.md`

**Used by:** `how/gates/_language-design.md`, `how/decision_records/`, concept design

**Key principle:** Rules are prescriptive — "orthogonality means exactly one responsibility per construct" — not descriptive

### Layer 3: Decision Validation (`how/gates/`)

**Purpose:** Define independent validation gates that proposals must pass before acceptance

**Location:** `how/gates/`

**Contains:**
- `DECISION_VALIDATION.md` — Six independent gates (Correctness, Implementation Independence, Orthogonality, Learnability, LLM Generability, Principle Alignment)
- `_language-design.md` — Quality gate checklist for language design decisions
- `methods/` — Validation methods (Scientific Method, Logical Analysis, Socratic Method, TRIZ, Einstein Method, Working Backwards Method)

**Depends on:** `what/DESIGN_PRINCIPLES.md`, `why/MANIFESTO.md`

**Used by:** Proposals before they become accepted concepts or EDRs

**Pattern:** Each gate is independent and answers a different validation question; a proposal must pass all gates to be accepted

### Layer 4: Language Design Gate (`how/gates/_language-design.md`)

**Purpose:** Formalize acceptance criteria for new language concepts

**Location:** `how/gates/_language-design.md`

**Contains:**
- Checklist of questions every design proposal must answer
- Policy footprint determination (which Implementation Policies the concept affects)
- Semantic specification requirements
- LLM generability verification

**Depends on:** `how/gates/DECISION_VALIDATION.md`, `what/DESIGN_PRINCIPLES.md`

**Used by:** Designers proposing new concepts (before creating concept document)

### Layer 5: Concepts & Strategies

**Concepts** (`how/concepts/research/`):
- Purpose: Formal semantic definition of language features
- Location: `how/concepts/research/{FEATURE_NAME}.md`
- Pattern: Each concept starts as DRAFT, passes Concept Design Review, then becomes accepted via EDR (Architecture category)
- Examples: `CORE_CONCEPTS.md`, `DATA_MODEL.md`, `FUNCTIONS.md`, `ERROR_HANDLING.md`

**Implementation Strategies** (`how/strategies/`):
- Purpose: Specify *how* language semantics are implemented for different execution environments
- Location: `how/strategies/{STRATEGY_NAME}.md`
- Pattern: Strategy = named set of Policies; Policies are declarative preferences (Allocation, Algorithm, Evaluation, Lifetime, Concurrency, etc.)
- Examples: `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md`

**Architecture Details** (`how/architecture/`):
- Purpose: Specify implementation architecture (compiler, runtime structure)
- Location: `how/architecture/{COMPONENT}.md`
- Contains: `ARCHITECTURE.md` (layered architecture), `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`, `IR.md`

## Data Flow

### Primary Design Flow

1. **Vision** (`why/VISION.md`)
   - Defines *why* — five pillars, core beliefs
   
2. **Core Principles** (`what/DESIGN_PRINCIPLES.md` + `why/MANIFESTO.md`)
   - Translates vision into concrete rules
   - Defines acceptance criteria
   
3. **Decision Validation** (`how/gates/DECISION_VALIDATION.md`)
   - Proposal passes six independent gates
   - Multi-perspective assessment
   
4. **Language Design Gate** (`how/gates/_language-design.md`)
   - Final checklist before acceptance
   - Determines Policy footprint, semantic spec requirements
   
5. **Concept Document** (`how/concepts/research/{FEATURE}.md`)
   - Formal semantic definition
   - Starts as DRAFT
   
6. **Concept Design Review** (`how/concept-design-review.md`)
   - Formal review process
   - Determines acceptance readiness
   
7. **Engineering Decision Record** (`how/decision_records/architecture/EDR-*.md`)
   - Consequential decisions recorded with context, rationale, alternatives
   - Creates permanent record linked to concept
   
8. **Implementation Strategy** (`how/strategies/{STRATEGY}.md`)
   - Translates semantic to concrete implementation through Policies
   - Specifies allocation, algorithm, evaluation, lifetime decisions
   
9. **Architecture Implementation** (`how/architecture/*.md`)
   - Detailed compiler/runtime architecture for strategies

### Secondary Flows

**Terminology Alignment:**
- New terms introduced in any concept document must be added to `what/GLOSSARY.md`
- GLOSSARY.md provides ubiquitous language across all layers
- Cross-references from GLOSSARY back to defining documents

**Decision Traceability:**
- Every EDR (`how/decision_records/{category}/EDR-*.md`) links to:
  - Justification in `why/VISION.md` or `why/GOALS.md`
  - Principles it satisfies (`what/DESIGN_PRINCIPLES.md`)
  - Alternatives considered
  - Related concepts (forward references to `how/concepts/research/`)
  
**Strategy Mapping:**
- Each Implementation Strategy in `how/strategies/` references:
  - Which concepts it implements
  - Policy values for each Policy type
  - Concrete implementation details in `how/architecture/`

## Key Abstractions

### Engineering Decision Records (EDRs)

**Purpose:** Permanent record of consequential design decisions with full context

**Structure:**
- **ID:** EDR-NNN (globally unique)
- **Title:** One-line decision statement
- **Date:** When decision was made
- **Status:** Accepted / Proposed / Deprecated / Superseded
- **Context:** Problem/opportunity description
- **Decision:** The choice made and why
- **Consequences:** Positive and negative impacts
- **Alternatives:** What was considered and why rejected
- **Related Concepts:** Links to `how/concepts/research/` docs
- **Supersedes:** Links to prior EDRs this replaces

**Categories:** Architecture, Process, Quality, Technology, Tooling, Delivery, Operations, Security, Governance, Data, AI, Documentation, Knowledge, Collaboration, Product

**Location:** `how/decision_records/{category}/EDR-{NNN}-title.md`

**Index:** `how/decision_records/INDEX.md` — unified journal of all decisions with status summary

**Pattern:** EDRs are first-class artifacts; design decisions are not complete until recorded

### Concepts

**Purpose:** Formal semantic definition of a language feature

**States:**
1. **DRAFT** — Exploratory phase, not yet validated
2. **IN DESIGN REVIEW** — Undergoing Concept Design Review process
3. **ACCEPTED** — Approved via EDR (Architecture category)

**Structure:** (from `how/templates/_concept.md`)
- **Overview:** One-paragraph summary
- **Motivation:** Why this concept is needed
- **Semantic Model:** Formal definition of what it means
- **Examples:** Canonical forms and use cases
- **Interactions:** How it composes with other concepts
- **Policy Footprint:** Which Implementation Policies it affects
- **Gates:** Validation against Language Design Gate checklist

**Location:** `how/concepts/research/{CONCEPT_NAME}.md`

**Linking:** Each concept references:
- Vision/Goals it serves
- Design Principles it embodies
- Related concepts
- Implementation Strategy handling
- EDR of acceptance decision

### Implementation Strategies & Policies

**Purpose:** Decouple language semantics (what) from implementation (how)

**Structure:**
```
Strategy (named set of Policies)
├── Allocation Policy (Heap, Arena, Linear, GC, Static)
├── Algorithm Policy (Adaptive, MemoryOptimized, Parallel)
├── Evaluation Policy (Eager, Lazy, Hybrid)
├── Lifetime Policy (Manual, Reference Counting, GC)
├── Concurrency Policy (Sequential, Threaded, Async/Await, Distributed)
└── [Other Policies as needed]
```

**Key Constraint:** Policies are *declarative* (state preference, not implementation), making strategies interchangeable without changing program semantics

**Predefined Strategies:**
- `DEFAULT_STRATEGY.md` — Balanced, general-purpose
- `EMBEDDED_STRATEGY.md` — Minimal memory, predictable timing
- `HIGH_PERFORMANCE_STRATEGY.md` — Throughput/latency optimized

### Design Review

**Purpose:** Formal validation of concept designs before acceptance

**Location:** `how/concept-design-review.md` (process definition), `how/templates/_design-review.md` (template)

**Pattern:** Concept → Design Review → EDR (acceptance) → Implementation Strategy

## Entry Points

**AGENTS.md** (`/Users/mniedre/git/orthon-lang/docs/AGENTS.md`):
- Location: Project root
- Triggers: AI agent contribution workflow
- Responsibilities: 
  - Defines agent protocol (Orient → Design → Gate → Write)
  - Provides complete document map with layer/purpose for each file
  - Specifies naming conventions, style guidelines
  - Lists terminology rules (ubiquitous language)

**README.md** (`/Users/mniedre/git/orthon-lang/docs/README.md`):
- Location: Project root
- Triggers: Human visitor orientation
- Responsibilities:
  - High-level project overview
  - Links to "Getting Started" documents (Manifesto, Vision, Goals)
  - Navigation table for key documents
  - Links to AGENTS.md for AI contributors

**PHILOSOPHY.md** (`how/PHILOSOPHY.md`):
- Location: `how/` layer
- Triggers: Understanding design decision flow
- Responsibilities:
  - Visualizes five-layer design flow as flowchart
  - Explains each layer's role and output
  - Links to corresponding files in each layer

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

**What happens:** A Why-level argument (philosophical belief) gets embedded in a What-level document (concrete specification)

Example: Writing "Orthon rejects null because we believe in explicit semantics" in `how/concepts/research/CORE_CONCEPTS.md`

**Why it's wrong:** Conflates layers. The philosophical rationale belongs in Why layer; the specification belongs in What layer. Mixing confuses future readers about whether the design is optional or mandatory.

**Do this instead:** 
- Philosophical rationale → `why/MANIFESTO.md` or `why/VISION.md`
- Concrete specification → `how/concepts/research/` file
- Link between them: Concept document says "See why/MANIFESTO.md §X for rationale"

### Anti-Pattern: Undocumented Design Decisions

**What happens:** A design choice is made, implemented in code or described in informal notes, but not recorded in an EDR

Example: A policy about whether to use reference counting is mentioned in a Slack conversation but not in `how/decision_records/`

**Why it's wrong:** Future designers cannot understand *why* decisions were made. Institutional knowledge is lost when people leave. Design rationale cannot be questioned or updated.

**Do this instead:** 
- If decision is consequential → create EDR in `how/decision_records/{category}/`
- If decision is small → record as inline gate entry in relevant document
- If decision is exploratory → document in `notes/` with clear "exploratory" marker

### Anti-Pattern: Isolated Concept Design

**What happens:** A new language concept is designed and documented in `how/concepts/research/` without connecting to Design Principles, validation gates, or implementation strategy

Example: A new `Option` type is defined in `how/concepts/research/` but nowhere does it explain how it satisfies DESIGN_PRINCIPLES, which gates validated it, or how implementation strategies handle it

**Why it's wrong:** Readers cannot trace concept back to vision or forward to implementation. The concept appears arbitrary rather than well-founded.

**Do this instead:**
- Every concept document includes:
  - Explicit link to which DESIGN_PRINCIPLEs it embodies
  - Reference to its validation (which gates passed, EDR of acceptance)
  - Policy Footprint section specifying affected Implementation Policies
  - Links to strategy-specific implementation

### Anti-Pattern: Strategy-Specific Language Semantics

**What happens:** Language semantics are specified in a way that only makes sense for one implementation strategy

Example: Defining a concept as "memory is allocated on a garbage-collected heap" instead of "memory lifetime is managed by the implementation strategy"

**Why it's wrong:** Breaks Implementation Independence Gate. Locks language to one strategy, preventing EMBEDDED_STRATEGY or other alternatives.

**Do this instead:** 
- Specify semantics in strategy-independent terms: "values have managed lifetimes"
- Let IMPLEMENTATION_POLICIES decide *how*: Allocation Policy = GC vs. RC vs. Arena
- Concept document includes Policy Footprint section showing which policies affect it

## Cross-Cutting Concerns

**Consistency:** Ensured by:
- `what/DESIGN_PRINCIPLES.md` — checks consistency principle adherence
- `how/gates/_language-design.md` — checklist item for representation symmetry
- EDRs reference DESIGN_PRINCIPLES they satisfy, creating traceability

**Terminology (Ubiquitous Language):** Ensured by:
- `what/GLOSSARY.md` — single source of truth for all defined terms
- AGENTS.md §6 — defines terminology rules
- Every new term introduced in any document must be added to GLOSSARY
- Cross-references from GLOSSARY back to defining documents

**Validation:** Ensured by:
- `how/gates/DECISION_VALIDATION.md` — six independent validation gates all proposals must pass
- `how/gates/_language-design.md` — language-design-specific quality checklist
- Design Review process (`how/concept-design-review.md`) — formal review before EDR acceptance

**Traceability:** Ensured by:
- EDR linking: Each EDR links to vision/goals it serves, principles it satisfies, alternatives considered
- Concept linking: Each concept references its EDR, validation gates, related concepts
- Strategy linking: Each strategy references which concepts it implements, policy values, architecture files
- Glossary linking: Each term links back to defining document

---

*Architecture analysis: 2026-07-20*
