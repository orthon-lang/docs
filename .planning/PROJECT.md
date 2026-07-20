# Orthon Language Specification

## What This Is

Orthon is a programming language designed using the same engineering principles it encourages in its users — orthogonal core, explicit semantics, SOLID architecture, and every consequential design choice recorded as an Engineering Decision Record (EDR). This repository is documentation-only: there is no compiler or runtime here. The project's output is a complete, self-consistent specification of Orthon v0.1, ready to hand off to a separate implementation repository.

Orthon targets two audiences simultaneously: humans (via orthogonality, deliberate design informed by prior languages, and logical evolution) and LLMs (via simplicity, minimal unnecessary invariants, and purpose-built LLM tooling). It also introduces a new DevOps model — the **Execution Program** — where a program's semantic meaning is decoupled from its execution strategy, so the same artifact can be interpreted, compiled, containerized, or built for WASM without modification.

## Core Value

A complete, self-consistent Orthon v0.1 specification — every concept accepted (no DRAFT), core architecture specs (IR, Parser, Type System, Name Resolution) filled in, all cross-references valid, and a final review confirming the result coheres as a real language specification.

## Requirements

### Validated

(None yet — nothing has passed formal acceptance/review yet; see Context for existing draft material)

### Active

- [ ] Foundations/Process phase: define concept acceptance gate (DRAFT → Accepted transition), Concept Design Review process, and milestone ownership model (solo-author, so this is about self-discipline/checklists, not team governance)
- [ ] Resolve critical blockers from CONCERNS.md: empty architecture specs (`IR.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`)
- [ ] Clean up tech debt from CONCERNS.md: deprecated `_adr.md` template, superseded `EXECUTION_IMAGE.md` references, missing date stamps
- [ ] Milestone 0 — Vision: review Manifesto, Vision, Zen against Working Backwards rationale; validate criteria-based algorithm selection idea
- [ ] Milestone 1 — Language Inventory: design the 13 outstanding concepts (Pattern Matching, Error Handling, Ownership, Generics/Traits, Async/Await, Object Initialization, Concurrency/Actors, Literate Programming, Sorting Stability, Unpacking/Destructuring, Generators/Yield, Metaobjects, Span/Memory View)
- [ ] Milestone 2 — Anti-pattern & Declarative Design Analysis: complete the 10 `imperative-crutch-*` research topics and run all concepts through Concept Design Review (DRAFT → Accepted)
- [ ] Milestone 3 — Cross-cutting Review: finalize Interaction Matrix format
- [ ] Milestone 6 — Naming: verify/decide whether "Orthon" is the final name
- [ ] Milestone 7 — Freeze: resolve/defer all validation issues with documented rationale; verify the spec is self-consistent and complete; final review confirming Orthon v0.1 coheres as a language specification; tag version; draft whitepaper strategy execution; define launch strategy
- [ ] Fix template-compliance and cross-reference integrity issues identified in CONCERNS.md as part of the above milestones (not a separate phase)

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
