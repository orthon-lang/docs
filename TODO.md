# TODO — Orthon Language

> All items owned by **Solo Author** unless otherwise noted.

> **⚠ Urgent:** These items address systemic pipeline throughput gaps identified in
> `docs/notes/pipeline-throughput-gaps.md`. They should be prioritised before
> milestone work to unblock the design pipeline.

## Pipeline & Process Gaps

- [ ] **GAP-01:** Run ONE concept through the full pipeline — take `EQUALITY.md` or `DATA_MODEL.md`, push through a 3-step process (Problem → Principle Check → Examples), accept into `CORE_CONCEPTS.md`
  - **Owner:** Solo author
  - **Target:** Now
  - **Ref:** `docs/notes/pipeline-throughput-gaps.md` § Top-5 #1
- [ ] **GAP-02:** Collapse the 11-stage concept design review to a 5-step pipeline suitable for solo authorship; reserve full gate mechanism for retroactive refinement
  - **Owner:** Solo author
  - **Target:** After GAP-01
  - **Ref:** `docs/notes/pipeline-throughput-gaps.md` § Top-5 #2, § Minimal Pipeline
- [ ] **GAP-03:** Create `DECISION_PROCESS.md` — one-page decision authority extracted from `AGENTS.md` and `concept-design-review.md`; update `README.md` and `AGENTS.md` to reference it
  - **Owner:** Solo author
  - **Target:** After GAP-02
  - **Ref:** `docs/notes/pipeline-throughput-gaps.md` § Top-5 #3
- [ ] **GAP-04:** Add throughput fitness functions to `FITNESS_FUNCTIONS.md` — minimum: concepts-in-flight, oldest open proposal age
  - **Owner:** Solo author
  - **Target:** After GAP-03
  - **Ref:** `docs/notes/pipeline-throughput-gaps.md` § Top-5 #4
- [ ] **GAP-05:** The next EDR must be a language design decision (not meta-process) — reorient the EDR system to its stated purpose
  - **Owner:** Solo author
  - **Target:** Next EDR
  - **Ref:** `docs/notes/pipeline-throughput-gaps.md` § Top-5 #5

## Milestone 0 — Vision

- [ ] review Manifesto (`docs/why/MANIFESTO.md`), Vision (`docs/why/VISION.md`), Zen (`docs/why/ZEN.md`) using Working Backwards rationale (`docs/why/WORKING_BACKWARDS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 2
- [ ] validate criteria-based algorithm selection idea (`docs/notes/criteria-based-algorithm-selection.md`)
  - **Owner:** Solo author
  - **Target:** Phase 2

## Milestone 1 — Language Inventory

- [ ] research Pattern Matching concept (`docs/how/concepts/research/PATTERN_MATCHING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Error Handling concept (`docs/how/concepts/research/ERROR_HANDLING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Ownership concept (`docs/how/concepts/research/OWNERSHIP.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Generics / Traits concept (`docs/how/concepts/research/GENERICS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Async/Await concept (`docs/how/concepts/research/ASYNC_AWAIT.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Object Initialization concept (`docs/how/concepts/research/OBJECT_INITIALIZATION.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Concurrency / Actors concept (`docs/how/concepts/research/CONCURRENCY.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Literate Programming concept (`docs/how/concepts/research/LITERATE_PROGRAMMING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Sorting Stability concept (`docs/how/concepts/research/SORTING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Unpacking / Destructuring concept (`docs/how/concepts/research/UNPACKING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Generators / Yield concept (`docs/how/concepts/research/GENERATORS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Metaobjects concept (`docs/how/concepts/research/METAOBJECTS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research Span / Memory View concept (`docs/how/concepts/research/SPAN.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3

## Milestone 2 — Anti-pattern & Declarative Design Analysis

Analysis of imperative crutches (anti-patterns from Python/Java) and
projection of conclusions onto Orthon's design. Each topic has a file in
`docs/how/concepts/research/imperative-crutch-*.md`.

- [ ] research: Collections & Loops — implications for Orthon collection API
      (`docs/how/concepts/research/imperative-crutch-collections-loops.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Null handling — nullable types, safe navigation, Optional
      (`docs/how/concepts/research/imperative-crutch-null-handling.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Resource management — RAII / defer / context managers
      (`docs/how/concepts/research/imperative-crutch-resource-management.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Collection initialization — literal syntax, immutability defaults
      (`docs/how/concepts/research/imperative-crutch-collection-init.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Type checking & casting — pattern matching, structural typing
      (`docs/how/concepts/research/imperative-crutch-type-casting.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Date/time API — immutable types, thread-safety, formatters
      (`docs/how/concepts/research/imperative-crutch-datetime.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Lazy sequences & generators — yield, lazy pipelines
      (`docs/how/concepts/research/imperative-crutch-lazy-sequences.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Multi-key sorting — tuple ordering, composable comparators
      (`docs/how/concepts/research/imperative-crutch-sorting.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Metaprogramming & reflection — compile-time vs runtime
      (`docs/how/concepts/research/imperative-crutch-metaprogramming.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Serialization — auto-derive, validation, format-agnostic
      (`docs/how/concepts/research/imperative-crutch-serialization.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3

## Milestone 3 — Cross-cutting Review

- [ ] investigate Interaction Matrix format and finalise (`docs/notes/interaction-matrix-format.md`)
  - **Owner:** Solo author
  - **Target:** Phase 4

## Milestone 6 — Naming

- [ ] verify whether the name «Orthon» fits the language being developed, or whether a more suitable name should be chosen
  - **Owner:** Solo author
  - **Target:** Phase 5

## Milestone 7 — Freeze

- [ ] resolve or defer all validation issues with documented rationale
  - **Owner:** Solo author
  - **Target:** Phase 5
- [ ] verify specification is self-consistent and complete
  - **Owner:** Solo author
  - **Target:** Phase 5
- [ ] tag version (e.g., Orthon 0.1 Specification)
  - **Owner:** Solo author
  - **Target:** Phase 5
- [ ] draft whitepaper(s) — revisit `docs/notes/whitepaper-strategy.md` and commission whitepapers
  - **Owner:** Solo author
  - **Target:** Phase 5
- [ ] define launch / announcement strategy (audience, channels, timing)
  - **Owner:** Solo author
  - **Target:** Phase 5

## Milestone 8 — Standard Library & FFI (post-Freeze)

- [ ] design standard library architecture — collections, I/O, formatting
  - **Owner:** Solo author
  - **Target:** Milestone 8
- [ ] design FFI & interoperability — C ABI, embedding API
  - **Owner:** Solo author
  - **Target:** Milestone 8
- [ ] implement LLM Toolchain — Schema Provider, Code Completer, Code Generator, Static Analyser
  - **Owner:** Solo author
  - **Target:** Milestone 8
- [ ] write standard library whitepaper if stdlib design introduces novel concepts
  - **Owner:** Solo author
  - **Target:** Milestone 8

## Milestone 9 — Build System & Tooling (post-Freeze)

- [ ] design build system & package manager
  - **Owner:** Solo author
  - **Target:** Milestone 9
- [ ] design developer tooling — formatter, linter, LSP (Language Server Protocol)
  - **Owner:** Solo author
  - **Target:** Milestone 9
- [ ] evaluate whether build system whitepaper would benefit early adopters
  - **Owner:** Solo author
  - **Target:** Milestone 9

## Milestone 10 — Implementation (separate repo)

- [ ] spin up compiler implementation repository
  - **Owner:** Solo author
  - **Target:** Milestone 10
- [ ] implement core compiler
  - **Owner:** Solo author
  - **Target:** Milestone 10
- [ ] implement runtime library
  - **Owner:** Solo author
  - **Target:** Milestone 10
- [ ] publish launch whitepaper(s) alongside initial release
  - **Owner:** Solo author
  - **Target:** Milestone 10

## Orthon for LLM

- [x] design: LLM_GENERABILITY_GATE — 7th gate for Decision Validation
      (`docs/notes/llm-generability-gate.md`, recorded in
      `EDR-011-llm-generability-gate.md`)
  - **Owner:** Solo author
  - **Target:** Phase 2
- [ ] research: what makes a grammar ML-friendly? (tokenizer alignment, ambiguity metrics)
  - **Owner:** Solo author
  - **Target:** Phase 2
- [ ] research: common LLM error patterns across programming languages
  - **Owner:** Solo author
  - **Target:** Phase 2
- [ ] design: Literate Programming as LLM-native interface
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] decide: Opinionated Code Generation mode (lint-level constraints for LLM output)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] decide: structured error format for LLM consumption (JSON schema)
  - **Owner:** Solo author
  - **Target:** Phase 3

