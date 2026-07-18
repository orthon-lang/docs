# Process Inventory

> Catalogue of process tools and approaches — the process-lens
> counterpart to `LANGUAGE_INVENTORY.md`. Answers the question:
> *what tools govern our design work, and what happens without them?*

---

## Overview

Every language design decision in Orthon is shaped by a set of
process tools: gates, methods, fitness functions, policies,
checklists, and procedures. This document catalogues them all in
one place, with their purpose, their TDR reference, and the
specific failure mode each one prevents.

| Tool | Type | EDR | What it prevents | Without it… |
|------|------|-----|------------------|-------------|
| Decision Validation Gates | Gate System | EDR-002 | Single-perspective validation; groupthink | Proposals pass on one lens, fail silently on others |
| Working Backwards | Method | EDR-003 | Building features nobody asked for | Language drifts from user needs |
| Socratic Method | Method | EDR-003 | Contradictions hidden by imprecise language | Ambiguity becomes specification |
| Scientific Method | Method | EDR-003 | Over-engineering disguised as completeness | Language bloats with unnecessary features |
| Logical Analysis | Method | EDR-003 | Layering violations that seem harmless | Architecture erodes through accumulated small breaches |
| TRIZ | Method | EDR-003 | Strategy compromises accepted too early | Implementation strategy becomes locked into semantics |
| Einstein's Method | Method | EDR-003 | Complexity that becomes permanent debt | Irreversible design mistakes |
| Language Design Gate Checklist | Checklist | EDR-004 | Gates without operational teeth | Surface-level validation; perfunctory gate-passing |
| Fitness Functions | Guard | EDR-005 | No measurable architectural health checks | Architectural decay accumulates undetected |
| Implementation Policies | Policy | EDR-006 | What and how entangled | Cannot change implementation strategy without changing semantics |
| Implementation Strategies | Strategy | EDR-006 | Ad-hoc strategy selection | No coherent implementation profiles for different use cases |
| Concept Design Review (11-step) | Procedure | EDR-007 | Unstructured, inconsistent concept design | Variable quality across language concepts |
| Architecture Decision Records (ADR) | Record | EDR-001 | Decisions without documented rationale | Design amnesia — "why did we choose this?" |
| Tools Decision Records (TDR) | Record | EDR-001 | Process tools without documented rationale | Process becomes unexamined ritual |
| Layered Architecture | Architecture | EDR-010 | Cross-layer coupling | Changes in one layer ripple unpredictably to others |
| Process Inventory | Inventory | EDR-011 | No map of process tools | Onboarding friction; tool blind spots |

---

## Dependency Graph

How process tools depend on and complement each other:

```mermaid
flowchart TD
    EDR-010["EDR-010: Layered Architecture"] --> EDR-006["EDR-006: Policies & Strategies"]
    EDR-003["EDR-003: Validation Methods"] --> EDR-002["EDR-002: Gate System"]
    EDR-002 --> EDR-004["EDR-004: Checklist"]
    EDR-002 --> EDR-007["EDR-007: Concept Design Review"]
    EDR-004 --> EDR-007
    EDR-005["EDR-005: Fitness Functions"] --> EDR-007
    EDR-006 --> EDR-007
    EDR-007 --> EDR-001["EDR-001: EDR System"]
    EDR-001 --> EDR-011["EDR-011: Process Inventory"]
    EDR-001 -.-> EDR-011
    EDR-002 -.-> EDR-011
    EDR-003 -.-> EDR-011
    EDR-004 -.-> EDR-011
    EDR-005 -.-> EDR-011
    EDR-006 -.-> EDR-011
    EDR-007 -.-> EDR-011
    EDR-010 -.-> EDR-011
```

Solid arrows: direct dependency (tool B requires tool A).
Dotted arrows: inventory reference (Process Inventory catalogues the tool).

---

## Coverage Map

Which tools are active at each milestone:

| Milestone | Tools Active |
|-----------|-------------|
| **M0 — Vision** | EDR-002 (Gates), EDR-003 (Methods), EDR-004 (Checklist), EDR-005 (Fitness Functions), EDR-006 (Policies), EDR-001 (EDR System), EDR-010 (Architecture) |
| **M1 — Inventory** | All M0 tools + EDR-011 (Process Inventory) |
| **M2 — Concept Design** | EDR-007 (Concept Design Review), all Gates (EDR-002), all Methods (EDR-003), Checklist (EDR-004), Fitness Functions (EDR-005), Policies (EDR-006), EDR (EDR-001) |
| **M3 — Cross-cutting** | `ARCHITECTURAL_INTEGRITY_GATE`, `LOGICAL_CONSISTENCY_GATE` (from EDR-002) |
| **M4 — Consistency** | `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE` (from EDR-002) |
| **M5 — Specification** | Fitness Functions (EDR-005) — guarding against late design changes |
| **M6 — Validation** | Fitness Functions (EDR-005) |
| **M7 — Freeze** | Fitness Functions (EDR-005) |
| **M8–M10** | Architecture (EDR-010), Policies (EDR-006) |

---

## See Also

- [`LANGUAGE_INVENTORY.md`](../when/LANGUAGE_INVENTORY.md) — language-lens counterpart
- [`ROADMAP.md`](../when/ROADMAP.md) — milestone definitions
- [`DECISION_VALIDATION.md`](gates/DECISION_VALIDATION.md) — gate catalogue
- [`tdr/`](tdr/) — superseded TDR records (migrated to EDR)
- [`decision_records/`](decision_records/) — active Engineering Decision Records
- [`decision_records/INDEX.md`](decision_records/INDEX.md) — unified EDR journal
