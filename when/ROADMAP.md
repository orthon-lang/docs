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
Milestone 0 (Vision)
    │
    ▼
Milestone 1 (Language Inventory)
    │
    ▼
Milestone 2 (Concept Design Review)
    │
    ▼
Milestone 3 (Cross-cutting Review)
    │
    ▼
Milestone 4 (Consistency Review)
    │
    ▼
Milestone 5 (Language Specification)
    │
    ▼
Milestone 6 (Validation)
    │
    ▼
Milestone 7 (Freeze)
    │
    ├──► Milestone 8 (Standard Library & FFI)
    │
    └──► Milestone 9 (Build System & Tooling)
              │
              ▼
         Milestone 10 (Implementation)
```

**Key principle: Design before implement.** Milestones 0–7 cover language
design only. Milestones 8–10 (implementation, tooling, stdlib) begin
after the language specification is frozen.

---

## Milestone 0 — Vision 🔄

**Goal:** Define the language's purpose, philosophy, design principles,
and the process infrastructure for making design decisions.

| Area | Deliverables | Status |
|------|-------------|--------|
| **Vision & Philosophy** | `VISION.md`, `MANIFESTO.md`, `ZEN.md` | 🔄 In Progress |
| **Design Method** | `WORKING_BACKWARDS.md` | 🔄 In Progress |
| **Design Principles** | `DESIGN_PRINCIPLES.md` | 🔄 In Progress |
-| **Decision Infrastructure** | `DECISION_VALIDATION.md`, `_language-design.md`, `IMPLEMENTATION_POLICIES.md`, TDR-001–TDR-009 | 🔄 In Progress |
| **Validation Methods** | Socratic, Scientific, Logical Analysis, TRIZ, Einstein | 🔄 In Progress |
| **Templates** | EDR templates (Architecture, Process, base), Design review template, Concept template | 🔄 In Progress |
| **Meta** | `README.md`, `AGENTS.md`, `GLOSSARY.md`, Documentation Principles, Philosophy | 🔄 In Progress |

**Exit criteria:** Vision, Goals, Non-goals, Target Audience, Use Cases,
and Design Principles are documented and agreed. Decision-making
infrastructure (gates, templates, methods) is in place.

---

## Milestone 1 — Language Inventory & Policy Research ⬜

**Goal:** Produce a complete inventory of all language concepts and
Policy types to be designed. This is a research phase — concepts are
identified and catalogued, not yet designed.

Every concept that will appear in the language specification must be
identified and listed. This includes core concepts (data, types,
functions) as well as higher-level constructs (modules, concurrency,
macros).

Additionally, the **Policy types** required by the language are
researched and catalogued. Each Policy type's preliminary association
with concepts is documented in `IMPLEMENTATION_POLICIES.md`, providing
the Policy footprint context needed before individual concept design
begins. (The `IMPLEMENTATION_POLICIES.md` framework is established in
Milestone 0 as a draft; concrete Policy types and their values are
filled in based on this research and finalised through Milestone 2.)

**Deliverables:**
- `LANGUAGE_INVENTORY.md` — categorised list of all concepts with brief
  descriptions and dependency relationships.
  • Includes **LLM Toolchain components**: Schema Provider, Code
    Completer, Code Generator, Static Analyser, Documentation
    Generator, Refactor/Migration Tool.
  • Includes **LLM-related Policy Types**: Toolchain Policy, Error
    Policy, Inference Policy.
- `IMPLEMENTATION_POLICIES.md` (updated) — initial Policy Type catalogue
  with preliminary concept associations.
- `PROCESS_INVENTORY.md` — process-lens catalogue of all tools, methods,
  and approaches used in the design process.

**Dependencies:** Milestone 0 (Vision provides the criteria for what
belongs in the language).

---

## Milestone 2 — Concept Design Review ⬜

**Goal:** Design each concept from Milestone 1 through a rigorous,
uniform review process.

Each concept goes through the
[Concept Design Review](../how/concept-design-review.md) procedure —
an 11-step process (Problem → Use Cases → Necessity → Sufficiency →
Alternatives → Principles Check → Interactions → Rationale →
Examples → Open Questions → ADR) — passes **all 7 Decision Validation
gates** (including `LLM_GENERABILITY_GATE`, see
[`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md)),
and satisfies the [`_language-design.md`](../how/gates/_language-design.md)
checklist before being accepted.

> **LLM Toolchain components** (Schema Provider, Code Completer, etc.)
> go through Concept Design Review as a **parallel track** with a lighter
> set of gates: `ARCHITECTURAL_INTEGRITY_GATE`, `USER_VALUE_GATE`,
> `LLM_GENERABILITY_GATE`.

> **Existing partial concept docs** (`concepts/CORE_CONCEPTS.md`, `concepts/DATA_MODEL.md`,
> `concepts/EQUALITY.md`, etc.) are considered **drafts / examples**. They will be
> formally reviewed through this milestone's process. A concept is
> registered only after acceptance via EDR (Architecture category).

> **Note on interactions (Step 7):** At this stage, Step 7 only
> **identifies which existing concepts** the new concept interacts with.
> The detailed pair-wise interaction analysis is deferred to
> Milestone 3 (Cross-cutting Review), when the full set of accepted
> concepts is available for systematic analysis.

**Deliverables:** One concept document per concept + one ADR per
accepted concept.

**Dependencies:** Milestone 1 (inventory tells us which concepts to design).

---

## Milestone 3 — Cross-cutting Review ⬜

**Goal:** Verify that all accepted concepts work together without
conflicts.

After each concept has been reviewed individually, their interactions
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

**Dependencies:** Milestone 2 (concepts must be individually designed
before their interactions can be assessed).

> **Input from Milestone 2:** Each concept's Step 7 identifies which
> other concepts it interacts with. This milestone receives that list
> of affected pairs and performs the full pair-wise interaction
> analysis, resolving conflicts at concept boundaries.

> **Note on format:** The exact format of the interaction matrix
> requires separate investigation (see `docs/notes/interaction-matrix-format.md`).
> The current approach uses per-concept "Interactions" sections
> (Milestone 2) that feed into a consolidated matrix in this milestone.

---

## Milestone 4 — Consistency Review ⬜

**Goal:** Review the language as a whole for uniformity, minimality,
and absence of special cases.

The language is examined as one coherent system:

- Syntax consistency — similar constructs look similar
- No special cases — context-dependent behaviour is eliminated
- Naming uniformity — consistent naming conventions
- Operator uniformity — consistent operator semantics
- Minimality — no concept duplicates another's responsibility
- No redundancy — "interesting" but unnecessary capabilities removed

> **Distinction from Milestone 2:** Milestone 2 (Steps 4 and 6) checks
> *internal* minimality — is each individual concept as lean as it can
> be? This milestone checks *global* minimality — does the language as
> a whole have redundant, overlapping, or unnecessary concepts? The
> same feature that appears justified in isolation may be removed here
> because another concept already covers its responsibility.
>
> *This is where approximately half of "interesting" features are
> removed.*

**Primary gates applied:** `CONCEPTUAL_SIMPLICITY_GATE`,
`LONG_TERM_MAINTAINABILITY_GATE`.

**Dependencies:** Milestone 3 (cross-cutting issues resolved before
consistency is locked).

---

## Milestone 5 — Language Specification ⬜

**Goal:** Produce the canonical language specification document.

The specification no longer discusses alternatives or design rationale.
It fixes the language.

Each section contains:

- Description
- Syntax
- Semantics
- Constraints
- Examples
- Links to ADRs

**Deliverables:**
- `SPEC.md` — canonical language specification.
- `SCHEMA.md` — machine-readable language schema (grammar, types,
  stdlib contracts) in a format consumable by the LLM Toolchain.

**Dependencies:** Milestones 2–4 (all concepts designed, cross-cutting
issues resolved, consistency confirmed).

---

## Milestone 6 — Validation ⬜

**Goal:** Test the language design in practice before freezing.

- Write reference examples and real programs
- Implement known algorithms to find ergonomic issues
- Rewrite libraries to identify gaps
- **LLM generation tests** — validate that LLMs can generate
  correct Orthon code for each concept
- **Schema round-trip tests** — verify that the schema →
  generation → validation cycle works end-to-end
- Document ambiguities and friction points
- Produce new ADRs for issues discovered

**Dependencies:** Milestone 5 (specification needed to validate against).

---

## Milestone 7 — Freeze ⬜

**Goal:** Freeze the first version of the language specification.

- All validation issues resolved or deferred with documented rationale
- Specification is self-consistent and complete
- Version is tagged (e.g., Orthon 0.1 Specification)

**Deliverable:** Frozen specification — `SPEC.md` at release tag.

**Dependencies:** Milestone 6 (all validation feedback incorporated).

---

## Milestone 8 — Standard Library & FFI (post-Freeze) ⬜

**Goal:** Define the standard library architecture and foreign-function
interface.

| # | Item | Description |
|---|------|-------------|
| 8.1 | Standard Library | Collections, I/O, formatting, etc. |
| 8.2 | FFI & Interoperability | C ABI, embedding API |
| 8.3 | LLM Toolchain | Schema Provider, Code Completer, Code Generator, Static Analyser implementation |

**Dependencies:** Milestones 0–7 (stdlib and FFI depend on the complete
language specification).

---

## Milestone 9 — Build System & Tooling (post-Freeze) ⬜

**Goal:** Design the developer toolchain.

| # | Item | Description |
|---|------|-------------|
| 9.1 | Build System | Build system & package manager design |
| 9.2 | Tooling | Formatter, linter, LSP (Language Server Protocol) |

**Dependencies:** Milestones 0–7 (tooling depends on the frozen specification).

---

## Milestone 10 — Implementation (post-Freeze) ⬜

**Goal:** Build the compiler / runtime (separate repository).

| # | Item | Description |
|---|------|-------------|
| 10.1 | Compiler | Compiler implementation |
| 10.2 | Runtime | Runtime library |

**Dependencies:** Milestones 0–9 (implementation should not begin until
the language is sufficiently specified and tooling is designed).

---

## Reference Documents

The following documents are **project reference materials**, available
throughout all milestones. They are not deliverables of any specific
milestone.

| Document | Purpose |
|----------|---------|
| `ARCHITECTURE.md` | Layered architecture definition |
| `FITNESS_FUNCTIONS.md` | Architectural fitness functions — measurable checks guarding against design decay |
| `IMPLEMENTATION_STRATEGIES.md` | Strategy model: Default, Embedded, High-Performance |
| `DEFAULT_STRATEGY.md` | Default implementation strategy profile |
| `EMBEDDED_STRATEGY.md` | Embedded implementation strategy profile |
| `HIGH_PERFORMANCE_STRATEGY.md` | High-performance strategy profile |
| `LLM_STRATEGY.md` | LLM strategy profile — maximum LLM assistance during development |
| `PROCESS_INVENTORY.md` | Process-lens catalogue of all tools, methods, and approaches |

> Concept-specific architecture documents (Parser, Type System, Name
> Resolution, IR) are initial stubs and will be designed through
> the Concept Design Review process (Milestone 2).

---

## Guiding Principles

- **One milestone at a time.** Each milestone should be substantially
  complete before the next begins (unless explicitly scoped otherwise).
- **Design before implement.** Milestones 0–7 (design) precede milestones
  8–10 (implementation).
- **ADRs as you go.** Architecture Decision Records are created alongside
  concept design decisions in Milestone 2, not deferred.
- **Docs drive, code follows.** This repository contains *only* design
  documentation. The compiler and runtime live in a separate repository.
