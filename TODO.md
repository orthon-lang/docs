# TODO — Orthon Language

## Milestone 0 — Vision

- [ ] review Manifesto (`docs/why/MANIFESTO.md`), Vision (`docs/why/VISION.md`), Zen (`docs/why/ZEN.md`) using Working Backwards rationale (`docs/why/WORKING_BACKWARDS.md`)
- [ ] validate criteria-based algorithm selection idea (`docs/notes/criteria-based-algorithm-selection.md`)

## Milestone 1 — Language Inventory

- [ ] design Pattern Matching concept (`docs/what/concepts/PATTERN_MATCHING.md`)
- [ ] design Error Handling concept (`docs/what/concepts/ERROR_HANDLING.md`)
- [ ] design Ownership concept (`docs/what/concepts/OWNERSHIP.md`)
- [ ] design Generics / Traits concept (`docs/what/concepts/GENERICS.md`)
- [ ] design Async/Await concept (`docs/what/concepts/ASYNC_AWAIT.md`)
- [ ] design Object Initialization concept (`docs/what/concepts/OBJECT_INITIALIZATION.md`)
- [ ] design Concurrency / Actors concept (`docs/what/concepts/CONCURRENCY.md`)
- [ ] design Literate Programming concept (`docs/what/concepts/LITERATE_PROGRAMMING.md`)
- [ ] design Sorting Stability concept (`docs/what/concepts/SORTING.md`)
- [ ] design Unpacking / Destructuring concept (`docs/what/concepts/UNPACKING.md`)
- [ ] design Generators / Yield concept (`docs/what/concepts/GENERATORS.md`)
- [ ] design Metaobjects concept (`docs/what/concepts/METAOBJECTS.md`)
- [ ] design Span / Memory View concept (`docs/what/concepts/SPAN.md`)

## Milestone 2 — Anti-pattern & Declarative Design Analysis

Analysis of imperative crutches (anti-patterns from Python/Java) and
projection of conclusions onto Orthon's design. Each topic has a file in
`docs/notes/imperative-crutch-*.md`.

- [ ] research: Collections & Loops — implications for Orthon collection API
      (`docs/notes/imperative-crutch-collections-loops.md`)
- [ ] research: Null handling — nullable types, safe navigation, Optional
      (`docs/notes/imperative-crutch-null-handling.md`)
- [ ] research: Resource management — RAII / defer / context managers
      (`docs/notes/imperative-crutch-resource-management.md`)
- [ ] research: Collection initialization — literal syntax, immutability defaults
      (`docs/notes/imperative-crutch-collection-init.md`)
- [ ] research: Type checking & casting — pattern matching, structural typing
      (`docs/notes/imperative-crutch-type-casting.md`)
- [ ] research: Date/time API — immutable types, thread-safety, formatters
      (`docs/notes/imperative-crutch-datetime.md`)
- [ ] research: Lazy sequences & generators — yield, lazy pipelines
      (`docs/notes/imperative-crutch-lazy-sequences.md`)
- [ ] research: Multi-key sorting — tuple ordering, composable comparators
      (`docs/notes/imperative-crutch-sorting.md`)
- [ ] research: Metaprogramming & reflection — compile-time vs runtime
      (`docs/notes/imperative-crutch-metaprogramming.md`)
- [ ] research: Serialization — auto-derive, validation, format-agnostic
      (`docs/notes/imperative-crutch-serialization.md`)

## Milestone 3 — Cross-cutting Review

- [ ] investigate Interaction Matrix format and finalise (`docs/notes/interaction-matrix-format.md`)

## Milestone 6 — Naming

- [ ] verify whether the name «Orthon» fits the language being developed, or whether a more suitable name should be chosen

## Milestone 7 — Freeze

- [ ] resolve or defer all validation issues with documented rationale
- [ ] verify specification is self-consistent and complete
- [ ] tag version (e.g., Orthon 0.1 Specification)
- [ ] draft whitepaper(s) — revisit `docs/notes/whitepaper-strategy.md` and commission whitepapers
- [ ] define launch / announcement strategy (audience, channels, timing)

## Milestone 8 — Standard Library & FFI (post-Freeze)

- [ ] design standard library architecture — collections, I/O, formatting
- [ ] design FFI & interoperability — C ABI, embedding API
- [ ] implement LLM Toolchain — Schema Provider, Code Completer, Code Generator, Static Analyser
- [ ] write standard library whitepaper if stdlib design introduces novel concepts

## Milestone 9 — Build System & Tooling (post-Freeze)

- [ ] design build system & package manager
- [ ] design developer tooling — formatter, linter, LSP (Language Server Protocol)
- [ ] evaluate whether build system whitepaper would benefit early adopters

## Milestone 10 — Implementation (separate repo)

- [ ] spin up compiler implementation repository
- [ ] implement core compiler
- [ ] implement runtime library
- [ ] publish launch whitepaper(s) alongside initial release

## Orthon for LLM

- [x] design: LLM_GENERABILITY_GATE — 7th gate for Decision Validation
      (`docs/notes/llm-generability-gate.md`, recorded in
      `EDR-011-llm-generability-gate.md`)
- [ ] research: what makes a grammar ML-friendly? (tokenizer alignment, ambiguity metrics)
- [ ] research: common LLM error patterns across programming languages
- [ ] design: Literate Programming as LLM-native interface
- [ ] decide: Opinionated Code Generation mode (lint-level constraints for LLM output)
- [ ] decide: structured error format for LLM consumption (JSON schema)

