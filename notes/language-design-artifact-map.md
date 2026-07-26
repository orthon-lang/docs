# Language Design Artifact Map

> A complete taxonomy of documents a language design project should
> produce, organised by function. Each entry describes the artifact's
> purpose and, where applicable, Orthon's current status.
>
> **See also:** [`AGENTS.md`](../AGENTS.md) (document map),
> [`ROADMAP.md`](../when/ROADMAP.md) (phase planning),
> [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md) (layer map)

---

## Purpose

When designing a programming language, the specification is the core
deliverable — but a viable language requires a whole system of
interconnected artifacts. This map catalogs what a complete language
design produces, organised into seven functional categories:

1. [Why — rationale and philosophy](#1-why--rationale-and-philosophy)
2. [What — language specification](#2-what--language-specification)
3. [How — design process and architecture](#3-how--design-process-and-architecture)
4. [Who — audience and onboarding](#4-who--audience-and-onboarding)
5. [When — roadmap and planning](#5-when--roadmap-and-planning)
6. [Validation and testing](#6-validation-and-testing)
7. [Meta — project maintenance](#7-meta--project-maintenance)

Use this map as a completeness checklist: gaps indicate either
deferred work or deliberate scope decisions.

---

## 1. Why — Rationale and Philosophy

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Vision / Manifesto** | Philosophy, values, principles — why this language needs to exist | ✅ `why/VISION.md`, `why/MANIFESTO.md` |
| **Goals & Non-Goals** | Concrete aims with acceptance criteria + what the language explicitly does **not** try to do | ✅ `why/GOALS.md` |
| **Zen / Aphorisms** | Quick path to a guiding principle for design decisions | ✅ `why/ZEN.md` |
| **Working Backwards** | From programmer pain-point → to language solution | ✅ `why/WORKING_BACKWARDS.md` |

---

## 2. What — Language Specification

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Syntax Specification** | Formal grammar, lexical structure | 🔜 `what/SYNTAX.md` (Phase 5) |
| **Semantic Model** | Identity, ownership, mutation, evaluation, visibility, lifetime | ✅ `what/SEMANTIC_MODEL.md` |
| **Type System** | Types, subtyping, inference, generics | 🔜 `how/architecture/TYPE_SYSTEM.md` (Phase 4) |
| **Core Concepts** | Each feature formally defined: Data Model, Functions, Error Handling... | 🔜 `what/concepts/` (Phase 4) |
| **Primitive Blocks** | Minimal orthogonal core — everything decomposes to these | ✅ `what/PRIMITIVE_BLOCKS.md` |
| **Library Boundary** | What is language vs stdlib vs external | ✅ `what/LIBRARY_BOUNDARY.md` |
| **Execution Model** | Evaluation order guarantees, concurrency, memory model | ✅ `what/EXECUTION_MODEL.md` |
| **Optimisation Model** | Which optimisations are permitted by semantics (as-if rule) | ✅ `what/OPTIMIZATION_MODEL.md` |
| **Unified SPEC.md** | A single, canonical, frozen specification document | 🔜 Phase 8.3 |
| **Cross-Cutting Matrix** | Pair-wise interaction analysis of all concepts | ✅ `what/CROSS_CUTTING.md` |
| **Conflict Registry** | Open conflicts and resolutions between concepts | ✅ `what/CONFLICT_REGISTRY.md` |
| **Glossary** | Ubiquitous language — all terms with definitions | ✅ `what/GLOSSARY.md` |

---

## 3. How — Design Process and Architecture

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Design Principles** | 20-30 rules governing every design decision | ✅ `how/DESIGN_PRINCIPLES.md` |
| **Decision Process** | One-page authority map — who decides what, using which criteria | ✅ `how/process/DECISION_PROCESS.md` |
| **Decision Pipeline** | 10-question filter before designing any feature | ✅ `how/process/DECISION_PIPELINE.md` |
| **EDRs / ADRs** | Decision journal — every consequential choice with rationale | ✅ `how/decision_records/` (INDEX.md + 15+ categories) |
| **Validation Gates** | Criteria: correctness, orthogonality, learnability, LLM generability... | ✅ `how/gates/DECISION_VALIDATION.md` |
| **Design Review Template** | Formal peer-review checklist | ✅ `how/templates/_design-review.md`, `how/gates/_language-design.md` |
| **Fitness Functions** | Automatable checks guarding against design decay | ✅ `how/architecture/FITNESS_FUNCTIONS.md` |
| **Architecture Specs** | Parser, Type System, Name Resolution, IR | ✅ `how/architecture/` (stubs exist; detail added per Phase) |
| **Implementation Strategies** | How semantics map to different execution environments | ✅ `how/strategies/` (Default, Embedded, High-Performance, LLM) |
| **Evolution Model** | Versioning, deprecation, experimental features, feature gates | 🔜 `how/EVOLUTION_MODEL.md` (Phase 8.1) |
| **Documentation Principles** | Standards for writing and maintaining docs | ✅ `how/DOCUMENTATION_PRINCIPLES.md` |
| **Concept Design Review Procedure** | 5-step pipeline for designing each concept | ✅ `how/concept-design-review.md` |

---

## 4. Who — Audience and Onboarding

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Tutorial / Getting Started** | "Write your first Orthon program" | ❌ — planned in Phase 9 |
| **Language Tour** | Walk through concepts in learnability order | ❌ — planned in Phase 9 |
| **Cookbook / Idioms** | How to solve common tasks idiomatically | ❌ — planned in Phase 9 |
| **Agent Guide (AGENTS.md)** | How LLM agents should write code in this language | ✅ `AGENTS.md` |
| **Cross-Reference Index** | Map of related documents | ✅ `AGENTS.md` §3 |

---

## 5. When — Roadmap and Planning

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Roadmap** | Milestones with phases and dependency graph | ✅ `when/ROADMAP.md` |
| **Phase Plans** | Concrete implementation plans per phase | 🔜 per phase execution |
| **TODO / Backlog** | Ideas, deferred features, known gaps | ✅ `TODO.md` |
| **Progress Tracking** | What's done, what's in progress, what's blocked | 🔜 GSD tracking |

---

## 6. Validation and Testing

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Conformance Test Suite** | Orthon programs with expected behaviour — covering every concept, canonical form, edge case, and interaction pair | ❌ — planned in Phase 8.4 |
| **Decision Validation Results** | Results of running gates for each accepted concept | 🔜 per concept EDR |
| **LLM Generability Test Results** | Verification that LLMs can produce correct code using each feature | 🔜 Phase 8.4 |
| **Schema Round-Trip Tests** | Schema → generation → validation cycle | 🔜 Phase 8.4 |

---

## 7. Meta — Project Maintenance

| Artifact | Purpose | Orthon status |
|----------|---------|---------------|
| **Templates** | `_edr.md`, `_concept.md`, `_design-review.md` — consistency enforcers | ✅ `how/templates/` |
| **Style Guide** | Tone, structure, naming conventions | ✅ `how/DOCUMENTATION_PRINCIPLES.md`, `AGENTS.md` §4 |
| **Agent Instructions** | How AI agents contribute | ✅ `AGENTS.md` |
| **Artifact Map** (this document) | Completeness checklist for the whole project | ✅ |

---

## Summary: Gap Analysis

| Category | Status |
|----------|--------|
| Why — rationale | ✅ Complete |
| What — specification | ✅ Mostly complete (SPEC.md unification in Phase 8.3) |
| How — design process | ✅ Complete |
| Who — onboarding | ❌ **Gap** — tutorial, language tour, cookbook (Phase 9) |
| When — roadmap | ✅ Complete |
| Validation & testing | ❌ **Gap** — conformance tests, LLM generability results (Phase 8.4) |
| Meta — maintenance | ✅ Complete |

**Two remaining gaps:**
1. **Conformance test suite** — planned for Phase 8.4
2. **Tutorial / Cookbook / Language Tour** — planned for Phase 9

---

## How to Maintain This Document

- When a new artifact type is added to the project, add a row to the
  appropriate table.
- When a phase is completed, update the Orthon status column from
  `🔜` to `✅`.
- If a new functional category emerges (e.g., security, formal
  verification), add a new section.
- Keep the summary gap analysis in sync with `ROADMAP.md`.
