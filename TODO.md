# TODO — Orthon Language

> All items owned by **Solo Author** unless otherwise noted.

## Milestone 0 — Vision

- [ ] review Manifesto (`docs/why/MANIFESTO.md`), Vision (`docs/why/VISION.md`), Zen (`docs/why/ZEN.md`) using Working Backwards rationale (`docs/why/WORKING_BACKWARDS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 2
- [ ] validate criteria-based algorithm selection idea (`docs/notes/criteria-based-algorithm-selection.md`)
  - **Owner:** Solo author
  - **Target:** Phase 2

## Milestone 1 — Language Inventory

- [ ] design Pattern Matching concept (`docs/what/concepts/PATTERN_MATCHING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Error Handling concept (`docs/what/concepts/ERROR_HANDLING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Ownership concept (`docs/what/concepts/OWNERSHIP.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Generics / Traits concept (`docs/what/concepts/GENERICS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Async/Await concept (`docs/what/concepts/ASYNC_AWAIT.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Object Initialization concept (`docs/what/concepts/OBJECT_INITIALIZATION.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Concurrency / Actors concept (`docs/what/concepts/CONCURRENCY.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Literate Programming concept (`docs/what/concepts/LITERATE_PROGRAMMING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Sorting Stability concept (`docs/what/concepts/SORTING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Unpacking / Destructuring concept (`docs/what/concepts/UNPACKING.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Generators / Yield concept (`docs/what/concepts/GENERATORS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Metaobjects concept (`docs/what/concepts/METAOBJECTS.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] design Span / Memory View concept (`docs/what/concepts/SPAN.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3

## Milestone 2 — Anti-pattern & Declarative Design Analysis

Analysis of imperative crutches (anti-patterns from Python/Java) and
projection of conclusions onto Orthon's design. Each topic has a file in
`docs/notes/imperative-crutch-*.md`.

- [ ] research: Collections & Loops — implications for Orthon collection API
      (`docs/notes/imperative-crutch-collections-loops.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Null handling — nullable types, safe navigation, Optional
      (`docs/notes/imperative-crutch-null-handling.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Resource management — RAII / defer / context managers
      (`docs/notes/imperative-crutch-resource-management.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Collection initialization — literal syntax, immutability defaults
      (`docs/notes/imperative-crutch-collection-init.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Type checking & casting — pattern matching, structural typing
      (`docs/notes/imperative-crutch-type-casting.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Date/time API — immutable types, thread-safety, formatters
      (`docs/notes/imperative-crutch-datetime.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Lazy sequences & generators — yield, lazy pipelines
      (`docs/notes/imperative-crutch-lazy-sequences.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Multi-key sorting — tuple ordering, composable comparators
      (`docs/notes/imperative-crutch-sorting.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Metaprogramming & reflection — compile-time vs runtime
      (`docs/notes/imperative-crutch-metaprogramming.md`)
  - **Owner:** Solo author
  - **Target:** Phase 3
- [ ] research: Serialization — auto-derive, validation, format-agnostic
      (`docs/notes/imperative-crutch-serialization.md`)
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

