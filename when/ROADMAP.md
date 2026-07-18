# ROADMAP — Orthon Language Design Roadmap

> Process-oriented milestones for the Orthon language design project.
> Each milestone represents a stage in the design process, from Vision
> to Language Specification Freeze.
> See [TODO.md](../TODO.md) for detailed task tracking.

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
| **Decision Infrastructure** | `DECISION_VALIDATION.md`, `_language-design.md`, `IMPLEMENTATION_POLICIES.md` | 🔄 In Progress |
| **Validation Methods** | Socratic, Scientific, Logical Analysis, TRIZ, Einstein | 🔄 In Progress |
| **Templates** | ADR template, Design review template, Concept template | 🔄 In Progress |
| **Meta** | `README.md`, `AGENTS.md`, `GLOSSARY.md`, Documentation Principles, Philosophy | 🔄 In Progress |

**Exit criteria:** Vision, Goals, Non-goals, Target Audience, Use Cases,
and Design Principles are documented and agreed. Decision-making
infrastructure (gates, templates, methods) is in place.

---

## Milestone 1 — Language Inventory ⬜

**Goal:** Produce a complete inventory of all language concepts to be
designed.

Every concept that will appear in the language specification must be
identified and listed. This includes core concepts (data, types,
functions) as well as higher-level constructs (modules, concurrency,
macros).

**Deliverable:** `LANGUAGE_INVENTORY.md` — categorised list of all
concepts with brief descriptions and dependency relationships.

**Dependencies:** Milestone 0 (Vision provides the criteria for what
belongs in the language).

---

## Milestone 2 — Concept Design Review ⬜

**Goal:** Design each concept from Milestone 1 through a rigorous,
uniform review process.

Each concept goes through the same 11-step procedure:

| Step | Section | Description |
|------|---------|-------------|
| 1 | **Problem** | What problem does this concept solve? Without which problem would its existence be meaningless? |
| 2 | **Use Cases** | Where is it used? What real problems does it solve? |
| 3 | **Necessity** | Can we live without it? If removed, what becomes impossible? What becomes inconvenient? |
| 4 | **Sufficiency** | Is the concept too large? Can it be split? Does it contain unnecessary capabilities? |
| 5 | **Alternatives** | What alternative implementations exist? (e.g., generator → yield → iterator → stream → pipeline → callback) |
| 6 | **Principles Check** | Verified against all Design Principles (Simplicity, Orthogonality, Explicitness, etc.) |
| 7 | **Interactions** | How does this concept interact with every other concept? Any non-orthogonal behaviours? |
| 8 | **Rationale** | Why was this specific option chosen? Why were others rejected? |
| 9 | **Examples** | Minimal, typical, edge case, and incorrect usage examples |
| 10 | **Open Questions** | What remains unresolved? |
| 11 | **ADR** | If a decision is made, it is formalised as an Architecture Decision Record |

Each concept passes through all 6 Decision Validation gates
(see [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md))
and satisfies the [`_language-design.md`](../how/gates/_language-design.md)
checklist before being accepted.

> **Existing partial concept docs** (`CORE_CONCEPTS.md`, `DATA_MODEL.md`,
> `EQUALITY.md`, etc.) are considered **drafts / examples**. They will be
> formally reviewed through this milestone's process. A concept is
> registered only after acceptance via ADR.

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

> **Note on format:** The exact format of the interaction matrix
> requires separate investigation (see `docs/notes/interaction-matrix-format.md`
> and `TODO.md`). The current approach uses per-concept "Interactions"
> sections (Milestone 2) that feed into a consolidated matrix in this
> milestone.

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

**Deliverable:** `SPEC.md`

**Dependencies:** Milestones 2–4 (all concepts designed, cross-cutting
issues resolved, consistency confirmed).

---

## Milestone 6 — Validation ⬜

**Goal:** Test the language design in practice before freezing.

- Write reference examples and real programs
- Implement known algorithms to find ergonomic issues
- Rewrite libraries to identify gaps
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

**Dependencies:** Milestones 0–7 (stdlib and FFI depend on the complete
language specification).

---

## Milestone 9 — Build System & Tooling (post-Freeze) ⬜

**Goal:** Design the developer toolchain.

| # | Item | Description |
|---|------|-------------|
| 9.1 | Build System | Build system & package manager design |
| 9.2 | Tooling | Formatter, linter, LSP protocol |

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
| `PARSER.md` | Parsing strategy and grammar structure |
| `TYPE_SYSTEM.md` | Type system architecture |
| `NAME_RESOLUTION.md` | Name resolution and scoping rules |
| `IR.md` | Intermediate representation design |
| `IMPLEMENTATION_STRATEGIES.md` | Strategy model: Default, Embedded, High-Performance |
| `DEFAULT_STRATEGY.md` | Default implementation strategy profile |
| `EMBEDDED_STRATEGY.md` | Embedded implementation strategy profile |
| `HIGH_PERFORMANCE_STRATEGY.md` | High-performance strategy profile |

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
