# ROADMAP — Orthon Language Design Roadmap

> Process-oriented milestones for the Orthon language design project.
> Each milestone represents a stage in the design process, from Vision
> to Language Specification Freeze.

---

## Overview

Language design follows a strict sequence of process milestones.
Each milestone produces concrete artifacts and must be substantially
complete before the next begins.

```
M1 — Orthon Language Spec (v0.1)
    │
    ├──► M2 — Standard Library & FFI
    │
    └──► M3 — Build System & Tooling
              │
              ▼
         M4 — Implementation
```

**Key principle: Design before implement.** M1 (Orthon Language Spec) covers
language design only — Epic 1 through Epic 5. M2–M4 (stdlib, tooling,
implementation) begin after the language specification is frozen.

---

## M1 — Orthon Language Spec (v0.1) ⬜

**Goal:** Deliver a frozen, self-consistent v0.1 language specification — from
establishing the design foundation and process infrastructure (Vision), through
inventorying and designing every language concept, to cross-cutting review,
consistency validation, and final freeze.

M1 is structured as five sequential Epics, mapped to the phased workflow in
`.planning/ROADMAP.md`:

```
Epic 1 — Concerns Remediation
    │
    ▼
Epic 2 — Foundations, Process & Vision
    │
    ▼
Epic 3 — Language Inventory & Concept Design Review
    │
    ▼
Epic 4 — Cross-Cutting Review
    │
    ▼
Epic 5 — Freeze & Naming
```

**Dependencies:** Nothing (initial milestone).

---

### Epic 1: Concerns Remediation ⬜

**Goal:** Resolve every finding in `.planning/codebase/CONCERNS.md` — architecture
gaps, tech debt, governance issues — so later Epic work rests on solid foundations
instead of placeholders.

**Deliverables:**
- Architecture specs filled: `IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`
- Template cleanup: `_adr.md` archived, `EXECUTION_IMAGE.md` archived
- Date-stamp/freshness metadata added to all concept and architecture documents
- Fitness functions catalogue completed in `FITNESS_FUNCTIONS.md`
- Documentation principles expanded beyond stub
- All 22 concept documents audited against `_concept.md` 8-section template
- Repository-wide cross-reference link audit
- Glossary maintenance workflow and versioning/change-log policy documented

---

### Epic 2: Foundations, Process & Vision ⬜

**Goal:** Define the language's purpose, philosophy, design principles, and the
process infrastructure for making design decisions. Establish the Concept
Design Review gate so subsequent Epic work has a defined acceptance procedure.

**Vision & Philosophy deliverables:**

| Area | Deliverables | Status |
|------|-------------|--------|
| **Vision & Philosophy** | `VISION.md`, `MANIFESTO.md`, `ZEN.md`, `GOALS.md` | 🔄 In Progress |
| **Design Method** | `WORKING_BACKWARDS.md` | 🔄 In Progress |
| **Design Principles** | `DESIGN_PRINCIPLES.md` | 🔄 In Progress |
| **Decision Infrastructure** | `DECISION_VALIDATION.md`, `_language-design.md`, `IMPLEMENTATION_POLICIES.md`, TDR-001–TDR-009 | 🔄 In Progress |
| **Validation Methods** | Socratic, Scientific, Logical Analysis, TRIZ, Einstein | 🔄 In Progress |
| **Templates** | EDR templates (Architecture, Process, base), Design review template, Concept template | 🔄 In Progress |
| **Meta** | `README.md`, `AGENTS.md`, `GLOSSARY.md`, Documentation Principles, Philosophy | 🔄 In Progress |

**Exit criteria:** Vision, Goals, Non-goals, Target Audience, Use Cases,
and Design Principles documented. Decision-making infrastructure (gates,
templates, methods) in place.

**Process deliverables:**
- `docs/how/concept-design-review.md` — complete DRAFT → Accepted transition process
- Template-compliance audit checklist for the 8-section `_concept.md` template
- All former `TODO.md` items migrated to tracked GSD requirements
- LLM-native concept shortlist: `docs/notes/llm-native-concept-shortlist.md`

---

### Epic 3: Language Inventory & Concept Design Review ⬜

**Goal:** Produce a complete inventory of all language concepts and Policy types,
then design each concept through a rigorous, uniform review process — informed
by anti-pattern research and the LLM-native shortlist from Epic 2.

**Language Inventory deliverables:**
- `LANGUAGE_INVENTORY.md` — categorised list of all concepts with descriptions
  and dependency relationships.
  • Includes **LLM Toolchain components**: Schema Provider, Code Completer,
    Code Generator, Static Analyser, Documentation Generator, Refactor/Migration Tool.
  • Includes **LLM-related Policy Types**: Toolchain Policy, Error Policy,
    Inference Policy.
- `IMPLEMENTATION_POLICIES.md` (updated) — initial Policy Type catalogue
  with preliminary concept associations.
- `PROCESS_INVENTORY.md` — process-lens catalogue of all tools, methods,
  and approaches used in the design process.

**Concept Design Review procedure:**
Each concept goes through the
[Concept Design Review](../how/concept-design-review.md) procedure —
an 11-step process (Problem → Use Cases → Necessity → Sufficiency →
Alternatives → Principles Check → Interactions → Rationale →
Examples → Open Questions → ADR) — passes **all 7 Decision Validation gates**
(including `LLM_GENERABILITY_GATE`) and satisfies the `_language-design.md`
checklist before being accepted.

> **LLM Toolchain components** (Schema Provider, Code Completer, etc.)
> go through Concept Design Review as a **parallel track** with a lighter
> set of gates: `ARCHITECTURAL_INTEGRITY_GATE`, `USER_VALUE_GATE`,
> `LLM_GENERABILITY_GATE`.

> **Existing partial concept docs** (`FOUNDATIONAL_ABSTRACTIONS.md`, `DATA_MODEL.md`,
> `EQUALITY.md`, etc.) are considered **drafts / examples**. They will be
> formally reviewed through this Epic's process. A concept is
> registered only after acceptance via EDR (Architecture category).

> **Note on interactions (Step 7):** At this stage, Step 7 only
> **identifies which existing concepts** the new concept interacts with.
> The detailed pair-wise interaction analysis is deferred to
> Epic 4 (Cross-Cutting Review), when the full set of accepted
> concepts is available for systematic analysis.

**Anti-pattern research:** Complete 10 `docs/how/concepts/research/imperative-crutch-*.md` topics,
each documenting findings and explicit implications for Orthon's concept designs.

**Deliverables:** 13 completed concept documents, 10 anti-pattern research notes,
one EDR per accepted concept.

**Concepts in scope:** `PATTERN_MATCHING.md`, `ERROR_HANDLING.md`,
`OWNERSHIP.md`, `GENERICS.md`, `ASYNC_AWAIT.md`, `OBJECT_INITIALIZATION.md`,
`CONCURRENCY.md`, `LITERATE_PROGRAMMING.md`, `SORTING.md`, `UNPACKING.md`,
`GENERATORS.md`, `METAOBJECTS.md`, `SPAN.md`.

**Dependencies:** Epic 2 (requires the acceptance gate and review process).

---

### Epic 4: Cross-Cutting Review ⬜

**Goal:** Verify that all accepted concepts work together without conflicts,
and finalise the Interaction Matrix format for documenting concept interactions.

After each concept has been reviewed individually (Epic 3), their interactions
are examined as a whole system. This is where contradictions at concept
boundaries are typically discovered.

| Pair Type | Example |
|-----------|---------|
| Generator × Context Manager | Resource lifetime across suspension points |
| Pattern Matching × Types | Exhaustiveness with nominal vs structural types |
| Modules × Visibility | Cross-module access control |
| Annotations × Reflection | Metadata discovery at runtime |
| Blocks × Closures | Variable capture and scope boundaries |
| Coroutine × Error Handling | Exception propagation across async boundaries |

**Deliverable:** Interaction matrix document + conflict registry.

**Primary gates applied:** `ARCHITECTURAL_INTEGRITY_GATE`,
`LOGICAL_CONSISTENCY_GATE`.

**Dependencies:** Epic 3 (concepts must be individually designed before
their interactions can be assessed).

> **Input from Epic 3:** Each concept's Interactions section identifies which
> other concepts it interacts with. This Epic receives that list of affected
> pairs and performs the full pair-wise interaction analysis, resolving
> conflicts at concept boundaries.

> **Note on format:** The exact format of the interaction matrix requires
> separate investigation (see `docs/notes/interaction-matrix-format.md`).
> The current approach uses per-concept "Interactions" sections
> (Epic 3) that feed into a consolidated matrix in this Epic.

---

### Epic 5: Freeze & Naming ⬜

**Goal:** From a consistent, cross-cutting-verified concept set, produce the
canonical specification, validate it in practice, and freeze Orthon v0.1.
This Epic consolidates four sequential phases: Consistency Review → Language
Specification → Validation → Freeze, plus the naming decision.

**Step 5.1 — Consistency Review:**
Review the language as a whole for uniformity, minimality, and absence of
special cases.

- Syntax consistency — similar constructs look similar
- No special cases — context-dependent behaviour is eliminated
- Naming uniformity — consistent naming conventions
- Operator uniformity — consistent operator semantics
- Minimality — no concept duplicates another's responsibility
- No redundancy — unnecessary capabilities removed

> **Distinction from Epic 3:** Epic 3 (Steps 4 and 6) checks *internal*
> minimality — is each individual concept as lean as it can be? This step
> checks *global* minimality — does the language as a whole have redundant,
> overlapping, or unnecessary concepts? The same feature that appears
> justified in isolation may be removed here because another concept already
> covers its responsibility.
>
> *This is where approximately half of "interesting" features are removed.*

**Primary gates applied:** `CONCEPTUAL_SIMPLICITY_GATE`,
`LONG_TERM_MAINTAINABILITY_GATE`.

**Dependencies:** Epic 4 (cross-cutting issues resolved before consistency
is locked).

**Step 5.2 — Language Specification:**
Produce the canonical language specification document. The specification no
longer discusses alternatives or design rationale — it fixes the language.

Each section contains:
- Description • Syntax • Semantics • Constraints • Examples • Links to ADRs

**Deliverables:**
- `SPEC.md` — canonical language specification.
- `SCHEMA.md` — machine-readable language schema (grammar, types, stdlib
  contracts) in a format consumable by the LLM Toolchain.

**Dependencies:** Epic 3–4 (all concepts designed, cross-cutting issues
resolved, consistency confirmed).

**Step 5.3 — Validation:**
Test the language design in practice before freezing.

- Write reference examples and real programs
- Implement known algorithms to find ergonomic issues
- Rewrite libraries to identify gaps
- **LLM generation tests** — validate that LLMs can generate correct Orthon
  code for each concept
- **Schema round-trip tests** — verify that the schema → generation →
  validation cycle works end-to-end
- Document ambiguities and friction points
- Produce new ADRs for issues discovered

**Dependencies:** Step 5.2 (specification needed to validate against).

**Step 5.4 — Freeze:**
Freeze the first version of the language specification.

- All validation issues resolved or deferred with documented rationale
- A documented decision (EDR or equivalent) confirms "Orthon" as the final
  project name, or names a replacement with rationale
- Specification is self-consistent and complete
- Version is tagged (e.g., Orthon 0.1 Specification)
- A link-check of all cross-references across the repository reports zero
  dangling references

**Deliverables:**
- Frozen specification — `SPEC.md` at release tag.
- `docs/notes/whitepaper-strategy.md` — whitepaper plan finalised; candidate
  topics, target audiences, and publication timeline defined.

**Dependencies:** Step 5.3 (all validation feedback incorporated).

---

## M2 — Standard Library & FFI (post-Freeze) ⬜

**Goal:** Define the standard library architecture and foreign-function
interface.

| # | Item | Description |
|---|------|-------------|
| 8.1 | Standard Library | Collections, I/O, formatting, etc. |
| 8.2 | FFI & Interoperability | C ABI, embedding API |
| 8.3 | LLM Toolchain | Schema Provider, Code Completer, Code Generator, Static Analyser implementation |

**Dependencies:** M1 (stdlib and FFI depend on the complete language
specification).

---

## M3 — Build System & Tooling (post-Freeze) ⬜

**Goal:** Design the developer toolchain.

| # | Item | Description |
|---|------|-------------|
| 9.1 | Build System | Build system & package manager design |
| 9.2 | Tooling | Formatter, linter, LSP (Language Server Protocol) |

**Dependencies:** M1 (tooling depends on the frozen specification).

---

## M4 — Implementation (post-Freeze) ⬜

**Goal:** Build the compiler / runtime (separate repository).

| # | Item | Description |
|---|------|-------------|
| 10.1 | Compiler | Compiler implementation |
| 10.2 | Runtime | Runtime library |
| 10.3 | Whitepapers | Publish launch whitepaper(s) alongside initial release |

**Dependencies:** M1–M3 (implementation should not begin until the language
is sufficiently specified and tooling is designed).

---

## Reference Documents

The following documents are **project reference materials**, available
throughout all milestones. They are not deliverables of any specific
milestone.

| Document | Purpose |
|----------|---------|
| `ARCHITECTURE.md` | Layered architecture definition |
| `FITNESS_FUNCTIONS.md` | Architectural fitness functions — measurable checks guarding against design decay |
| `whitepaper-strategy.md` (notes) | Whitepaper publication plan — timing, topics, and audience per milestone |
| `IMPLEMENTATION_STRATEGIES.md` | Strategy model: Default, Embedded, High-Performance |
| `DEFAULT_STRATEGY.md` | Default implementation strategy profile |
| `EMBEDDED_STRATEGY.md` | Embedded implementation strategy profile |
| `HIGH_PERFORMANCE_STRATEGY.md` | High-performance strategy profile |
| `LLM_STRATEGY.md` | LLM strategy profile — maximum LLM assistance during development |
| `PROCESS_INVENTORY.md` | Process-lens catalogue of all tools, methods, and approaches |

> Concept-specific architecture documents (Parser, Type System, Name
> Resolution, IR) are initial stubs and will be designed through
> the Concept Design Review process (Epic 3).

---

## Guiding Principles

- **One milestone at a time.** Each milestone should be substantially
  complete before the next begins (unless explicitly scoped otherwise).
- **Design before implement.** M1 (language design) precedes M2–M4 (stdlib,
  tooling, implementation).
- **ADRs as you go.** Architecture Decision Records are created alongside
  concept design decisions in Epic 3, not deferred.
- **Docs drive, code follows.** This repository contains *only* design
  documentation. The compiler and runtime live in a separate repository.
