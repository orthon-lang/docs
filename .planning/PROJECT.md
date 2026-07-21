# Orthon Language Specification

## What This Is

Orthon is a programming language designed using the same engineering principles it encourages in its users — orthogonal core, explicit semantics, SOLID architecture, and every consequential design choice recorded as an Engineering Decision Record (EDR). This repository is documentation-only: there is no compiler or runtime here. The project's output is a complete, self-consistent specification of Orthon v0.1, ready to hand off to a separate implementation repository.

Orthon targets two audiences simultaneously: humans (via orthogonality, deliberate design informed by prior languages, and logical evolution) and LLMs (via simplicity, minimal unnecessary invariants, and purpose-built LLM tooling). It also introduces a new DevOps model — the **Execution Program** — where a program's semantic meaning is decoupled from its execution strategy, so the same artifact can be interpreted, compiled, containerized, or built for WASM without modification.

## Core Value

A complete, self-consistent Orthon v0.1 specification — every concept accepted (no DRAFT), core architecture specs (IR, Parser, Type System, Name Resolution) filled in, all cross-references valid, and a final review confirming the result coheres as a real language specification.

## Requirements

### Validated

- [x] Phase 1 — Concerns Remediation: all 17 DEBT requirements resolved and verified (2026-07-20) — empty architecture specs filled (`IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`), deprecated `_adr.md`/`EXECUTION_IMAGE.md` archived, date stamps added, fitness functions catalogued, template audit completed, ownership/link audit done, glossary/versioning/IA plans documented.

### Active

Roadmap restructured 2026-07-21 to mirror `when/ROADMAP.md`'s 8-phase architectural design pipeline (an 11-level engineering design hierarchy), replacing the prior 5-milestone breakdown:

- [ ] Phase 1.1 — Foundation Completion: consolidate Vision into one canonical block, lock Design Principles as constitutional, finalize process infrastructure (Decision Process, collapsed Concept Design Review, throughput fitness functions), validate criteria-based algorithm selection idea
- [ ] Phase 2 — Semantic Model: define identity, ownership, mutation, evaluation, visibility, and lifetime as one unified model
- [ ] Phase 3 — Primitive Blocks: identify the minimal orthogonal set of primitive building blocks every derived feature decomposes into
- [ ] Phase 4 — Derived Features & Decision Pipeline: design the 13 outstanding concepts (Pattern Matching, Error Handling, Ownership, Generics/Traits, Async/Await, Object Initialization, Concurrency/Actors, Literate Programming, Sorting Stability, Unpacking/Destructuring, Generators/Yield, Metaobjects, Span/Memory View) via the 10-question Decision Pipeline; complete the 10 `imperative-crutch-*` anti-pattern research topics; classify each concept Language/StdLib/External; accept via EDR
- [ ] Phase 5 — Syntax Design: derive syntax from the semantic model for every accepted concept; update `PARSER.md` with concrete grammar
- [ ] Phase 6 — Cross-Cutting Review: finalize the Interaction Matrix format and build the full pairwise interaction analysis; resolve all conflicts
- [ ] Phase 7 — Execution & Optimization Model: define the semantics/implementation boundary; reconcile Implementation Strategies
- [ ] Phase 8 — Evolution Model & Freeze: define versioning/deprecation policy; verify/decide "Orthon" naming; resolve/defer all validation issues; produce canonical `SPEC.md`/`SCHEMA.md`; final consistency review; tag version; execute whitepaper/launch strategy

### Out of Scope

- Milestone 8 (Standard Library & FFI design) — post-Freeze, deferred until v0.1 spec is locked
- Milestone 9 (Build System & Tooling design) — post-Freeze, deferred until v0.1 spec is locked
- Milestone 10 (actual compiler/runtime implementation) — happens in a separate repository, not this docs repo
- "Orthon for LLM" research items beyond what's needed to keep Milestone 1-2 concepts LLM-generable — deferred to post-Freeze LLM Toolchain work (Milestone 8 area), except the already-designed LLM Generability Gate (EDR-011)

## Context

- This is a solo project — one author making all design decisions; no team/contributor governance process needed.
- Existing `.planning/codebase/` map (created via `/gsd-map-codebase`) documents the current state in detail: `ARCHITECTURE.md`, `STRUCTURE.md`, `CONVENTIONS.md`, `TESTING.md` (validation practices), `STACK.md`, `INTEGRATIONS.md`, and especially `CONCERNS.md` (4 critical issues, 14 tech-debt areas, 12 process gaps).
- `TODO.md` at the repo root already tracks a milestone roadmap (Vision → Language Inventory → Anti-pattern Analysis → Cross-cutting Review → Naming → Freeze → Stdlib/FFI → Build System → Implementation → Orthon for LLM). This GSD roadmap takes that TODO as its base through Milestone 7 (Freeze), reordered/regrouped in light of CONCERNS.md findings, with each TODO line converted into a GSD requirement/phase seed rather than tracked as freeform TODO checkboxes going forward.
- 18 of 22 concept documents are currently marked `⚠️ DRAFT` with no formal acceptance process yet — Milestone 1-2 work depends on resolving this (see Foundations/Process requirement above).
- Four architecture documents referenced in `AGENTS.md` are currently empty (`IR.md`, `NAME_RESOLUTION.md`, `PARSER.md`, `TYPE_SYSTEM.md`) — flagged as the highest-severity blocker in `CONCERNS.md`.
- Project uses an EDR (Engineering Decision Record) system for documenting consequential design choices; an older TDR system was superseded but not fully archived (EDR numbering gap at 008-009).

## Constraints

- **Scope**: Documentation/specification only — no compiler, runtime, or implementation code in this repository (that's a future, separate repo — Milestone 10).
- **Process**: Every consequential design decision must be recorded as an EDR with context, rationale, and alternatives considered.
- **Design philosophy**: Every language construct must pass the LLM Generability Gate (validates whether an LLM can reliably produce correct code using it) in addition to human-facing design review.
- **Ownership**: Solo-authored — no multi-contributor review/approval workflow required, but self-imposed rigor (Concept Design Review, acceptance gates) still applies.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| GSD roadmap scoped to Milestones 0-7 only (through Freeze) | Stdlib/tooling (M8-9) and implementation (M10) depend on a locked spec and belong to later/separate efforts | — Pending |
| Existing TODO.md milestones re-planned as a dedicated early "Foundations/Process" phase plus CONCERNS.md remediation, before resuming Milestone 1-2 content work | CONCERNS.md identifies undefined acceptance/review process as a blocker that would otherwise stall Milestone 2 | — Pending |
| Definition of Done for v0.1 spec includes: all concepts Accepted (no DRAFT), architecture specs filled (IR/Parser/TypeSystem/NameRes), all cross-references valid, plus a final review confirming Orthon v0.1 coheres as a language specification | User-specified acceptance criteria for the Freeze milestone | — Pending |
| GSD roadmap restructured (2026-07-21) from 5 phases to an 8-phase pipeline (+ inserted Phase 1.1) mirroring `when/ROADMAP.md`'s 11-level engineering design hierarchy: Foundation → Semantic Model → Primitive Blocks → Derived Features & Decision Pipeline → Syntax Design → Cross-Cutting → Execution & Optimization Model → Evolution & Freeze | `when/ROADMAP.md` was independently rewritten to this structure on 2026-07-21 (commit `3ec6c8a`), informed by a chorus review (`docs/reviews/2026-07-21-chorus-review.md`) and `notes/pipeline-throughput-gaps.md`; GSD tracking had fallen out of sync with it | Applied |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-07-20 after initialization*
