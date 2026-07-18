# EDR-001: Engineering Decision Records (EDR) System

**Status:** Accepted

**Date:** 2026-07-18

**Category:** Process

**Scope:** Project

---

### Context

The project initially adopted two parallel decision record systems:
- **ADR** (Architecture Decision Records, TDR-007) — for language design decisions
- **TDR** (Tools Decision Records, TDR-008) — for process infrastructure decisions

This dual system creates an artificial boundary. In practice, engineering
decisions span many dimensions beyond Architecture and Process: Technology
choices, Security postures, Quality standards, Delivery pipelines, Operational
practices, Governance models, Data strategies, AI integration decisions,
Documentation conventions, Knowledge management, Collaboration patterns, and
Product decisions.

Each of these dimensions carries its own context, trade-offs, and compliance
requirements. Treating Architecture and Process as the only "decision-worthy"
categories leaves the other dimensions undocumented — leading to the same
design amnesia that ADR was created to prevent.

The project needs a **unified engineering decision journal** — a single system
where any consequential engineering decision can be recorded, categorized by
lens, and surfaced in a master index.

### Decision

Adopt the **Engineering Decision Record (EDR)** system — a unified model
for recording all consequential engineering decisions, classified by one of
15 categorical lenses.

#### The 15 Lenses

Every EDR is tagged with exactly one primary Category:

| Lens | Category | Core Question |
|------|----------|---------------|
| Architecture | `Architecture` | How is the system structured? |
| Technology | `Technology` | What do we build with? |
| Tooling | `Tooling` | What tools do we use? |
| Process | `Process` | How do we work? |
| Delivery | `Delivery` | How do we ship? |
| Operations | `Operations` | How do we run it? |
| Quality | `Quality` | How do we verify quality? |
| Security | `Security` | How do we ensure security? |
| Governance | `Governance` | Who decides and how? |
| Data | `Data` | How do we work with data? |
| AI | `AI` | How do we use LLMs/AI? |
| Documentation | `Documentation` | How do we document? |
| Knowledge | `Knowledge` | How do we share knowledge? |
| Collaboration | `Collaboration` | How do teams interact? |
| Product | `Product` | How are product decisions made? |

#### Format

All EDRs share a common base format (`_edr.md`) with these fields:
- **ID** — global EDR-NNN (sequential across all categories)
- **Status** — Proposed, Accepted, Deprecated, or Superseded
- **Date** — YYYY-MM-DD
- **Category** — one of the 15 lenses
- **Scope** — Platform, Subsystem, Module, Team, Project, Policy
- **Context** — what problem prompted this decision
- **Decision** — the concrete decision
- **Consequences** — positive and negative
- **Compliance** — how adherence is verified
- **Alternatives Considered** — and why rejected

Category-specific templates add sections as needed:
- **Process** adds: "Without It" (risk analysis), "Evolution" (deprecation criteria)
- Other lenses use the base template; additional sections are added on demand
  when the first record of that category is created.

#### Numbering

Global sequential EDR-NNN numbering across all categories. The ID does not
encode the category — the `Category` field and directory placement serve that
purpose. The master INDEX.md provides category-based filtering.

#### Storage

Files are stored in per-category directories under `docs/how/decision_records/`:

```
decision_records/
├── INDEX.md
├── architecture/    EDR-NNN-*.md
├── technology/      EDR-NNN-*.md
├── tooling/         EDR-NNN-*.md
├── process/         EDR-NNN-*.md
├── delivery/        EDR-NNN-*.md
├── operations/      EDR-NNN-*.md
├── quality/         EDR-NNN-*.md
├── security/        EDR-NNN-*.md
├── governance/      EDR-NNN-*.md
├── data/            EDR-NNN-*.md
├── ai/              EDR-NNN-*.md
├── documentation/   EDR-NNN-*.md
├── knowledge/       EDR-NNN-*.md
├── collaboration/   EDR-NNN-*.md
└── product/         EDR-NNN-*.md
```

Templates are in `docs/how/templates/`: `_edr.md` (base), `_edr-architecture.md`,
`_edr-process.md`.

#### Unified Journal

The master index at `decision_records/INDEX.md` lists all EDRs with columns:
ID, Category, Title, Status, Date, Supersedes. This is the single entry point
for discovering any engineering decision.

#### Relationship to ADR and TDR

ADR and TDR are now **categories of EDR**, not separate systems:
- **ADR** → EDR with `Category: Architecture`
- **TDR** → EDR with `Category: Process` (or another appropriate lens — e.g.,
  fitness functions are `Quality`, layered architecture is `Architecture`)

TDR-007 (ADR System) and TDR-008 (TDR System) are superseded by this record.

### Without It

| Risk | Severity | Manifestation |
|------|----------|---------------|
| Decision blind spots | High | Non-Architecture, non-Process decisions remain undocumented |
| Fragmented knowledge | Medium | Decisions scattered across ADR, TDR, and ad-hoc notes — no single source of truth |
| Category confusion | Medium | "Is this an ADR or a TDR?" — artificial boundary creates friction |
| Onboarding friction | Low | New contributors must learn two separate record systems |

### Consequences

- **Positive:**
  - All engineering decisions, regardless of lens, have a home
  - Single numbering scheme — one source of truth
  - Category system prompts the team to consider dimensions they might overlook
  - INDEX.md is a single entry point for decision discovery
  - Extensible — new lenses can be added without changing the format
- **Negative:**
  - 15 categories may feel like overhead for a small project (but directories
    remain empty until needed)
  - Migration cost — 10 existing TDRs must be reformatted as EDRs
  - ADR and TDR as terms have industry recognition; replacing them with EDR
    loses that familiarity

### Evolution

- **Add a lens:** propose a new Category via an EDR (this record is amended).
  The new lens must cover a dimension not already addressed by the existing 15.
- **Remove a lens:** if a category has zero records after a significant period
  and no foreseeable need, deprecate it via an EDR.
- **Change the format:** propose an amendment EDR. The base template (`_edr.md`)
  is the canonical source; all category templates inherit from it.

### Compliance

- Every new consequential engineering decision must be recorded as an EDR.
- EDRs are referenced in code review checklists, gate entries, and concept
  design reviews.
- INDEX.md is updated on every EDR creation or status change.
- Fitness function: `edr-coverage` — ratio of documented to undocumented
  significant decisions (target: 100% for decisions that survive more than
  one milestone).

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep ADR + TDR separate, add new record types per lens | Proliferates record types; 15 different formats to maintain |
| Single flat file (one CHANGELOG.md) | Not discoverable by category; hard to maintain cross-references |
| Database-backed decision registry | Over-engineered for a documentation-first project; plain markdown is git-friendly |
| Per-category numbering (ADR-NNN, SEC-NNN, etc.) | Creates 15 separate sequences; harder to see chronological order across categories |

### Relationship to Other Records

| Record | Relationship |
|--------|-------------|
| TDR-007 (ADR System) | Superseded by this record |
| TDR-008 (TDR System) | Superseded by this record |
| EDR-002 (Gate System) | Migrated from TDR-001 |
| EDR-003 (Validation Methods) | Migrated from TDR-002 |
| EDR-004 (Checklist) | Migrated from TDR-003 |
| EDR-005 (Fitness Functions) | Migrated from TDR-004 |
| EDR-006 (Policies & Strategies) | Migrated from TDR-005 |
| EDR-007 (Concept Design Review) | Migrated from TDR-006 |
| EDR-010 (Layered Architecture) | Migrated from TDR-009 |
| EDR-011 (Process Inventory) | Migrated from TDR-010 |
| INDEX.md | Aggregated view of all EDRs |
