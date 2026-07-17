# ROADMAP — Orthon Project Roadmap

> Milestones and remaining work for the Orthon language design project.
> See [TODO.md](../TODO.md) for the detailed task tracking.

---

## Milestone 0 — Foundation 🔄

**Goal:** Establish the project's purpose, design philosophy, architecture, and core concepts.

| Area | Deliverables | Status |
|------|-------------|--------|
| **Why** | `MANIFESTO.md`, `VISION.md`, `ZEN.md`, `WORKING_BACKWARDS.md` | 🔄 In Progress |
| **How — Architecture** | `ARCHITECTURE.md`, `PARSER.md`, `TYPE_SYSTEM.md`, `NAME_RESOLUTION.md`, `IR.md` | 🔄 In Progress |
| **How — Strategies** | `IMPLEMENTATION_STRATEGIES.md`, `DEFAULT_STRATEGY.md`, `EMBEDDED_STRATEGY.md`, `HIGH_PERFORMANCE_STRATEGY.md` | 🔄 In Progress |
| **How — Methods** | Working Backwards, Socratic, Scientific, Logical Analysis, TRIZ, Einstein | 🔄 In Progress |
| **How — Gates & Policies** | `DECISION_VALIDATION.md`, `_language-design.md`, `IMPLEMENTATION_POLICIES.md` | 🔄 In Progress |
| **How — Templates** | ADR template, Design review template, Concept template | 🔄 In Progress |
| **What — Core Concepts** | `CORE_CONCEPTS.md`, `DATA_MODEL.md`, `EQUALITY.md`, `ALLOCATION.md`, `OWNERSHIP.md`, `MUTABILITY.md`, `FUNCTIONS.md` | 🔄 In Progress |
| **Meta** | `README.md`, `AGENTS.md`, `GLOSSARY.md`, Documentation Principles, Design Principles, Philosophy | 🔄 In Progress |

---

## Milestone 1 — Language Concepts

**Goal:** Design all remaining language-level concepts and produce specification documents.

Concepts are listed in approximate order of dependency (earlier concepts may be prerequisites for later ones).

| # | Concept | Description | Status |
|---|---------|-------------|--------|
| 1.1 | Modules & Namespaces | Import/export, visibility, encapsulation | ⬜ Pending |
| 1.2 | Types | Nominal & structural typing, generics, type aliases | ⬜ Pending |
| 1.3 | Control Flow | Conditionals, loops, pattern matching, early return | ⬜ Pending |
| 1.4 | Error Handling | Result types, error propagation, panic/recover | ⬜ Pending |
| 1.5 | Traits / Interfaces / Protocols | Abstraction mechanism | ⬜ Pending |
| 1.6 | Pattern Matching | Destructuring, exhaustiveness | ⬜ Pending |
| 1.7 | Concurrency | Async/await, threads, channels, futures | ⬜ Pending |
| 1.8 | Macros | Compile-time metaprogramming | ⬜ Pending |

**Dependencies:** Milestone 0 (all concepts build on the Core Concepts foundation).

---

## Milestone 2 — Standard Library & Interoperability

**Goal:** Define the standard library architecture and foreign-function interface.

| # | Item | Description | Status |
|---|------|-------------|--------|
| 2.1 | Standard Library | Collections, I/O, formatting, etc. | ⬜ Pending |
| 2.2 | FFI & Interoperability | C ABI, embedding API | ⬜ Pending |

**Dependencies:** Milestone 1 (standard library and FFI depend on the type system, control flow, error handling, and module system).

---

## Milestone 3 — ADRs, Build System & Tooling

**Goal:** Record all Architecture Decision Records and design the developer toolchain.

| # | Item | Description | Status |
|---|------|-------------|--------|
| 3.1 | ADRs | Architecture Decision Records — ADR-001 onward | ⬜ Pending |
| 3.2 | Build System | Build system & package manager design | ⬜ Pending |
| 3.3 | Tooling | Formatter, linter, LSP protocol | ⬜ Pending |

**Dependencies:** Milestones 1–2 (ADRs should be recorded as decisions are made during design; tooling and build system depend on the concrete language specification).

---

## Milestone 4 — Implementation

**Goal:** Build the compiler / runtime (separate repository).

| # | Item | Description | Status |
|---|------|-------------|--------|
| 4.1 | Compiler | Compiler implementation | ⬜ Pending |
| 4.2 | Runtime | Runtime library | ⬜ Pending |

**Dependencies:** Milestones 1–3 (implementation should not begin until the language design is sufficiently specified).

---

## Dependency Graph

```
Milestone 0 (Foundation)
    │
    ▼
Milestone 1 (Language Concepts)
    │
    ├──► Milestone 2 (Stdlib & FFI)
    │
    └──► Milestone 3 (ADRs, Build, Tooling)
              │
              ▼
         Milestone 4 (Implementation)
```

## Guiding Principles

- **One milestone at a time.** Each milestone should be substantially complete before the next begins (unless explicitly scoped otherwise).
- **Design before implement.** Milestone 4 (implementation) is deliberately last — the language must be specified before it can be built.
- **ADRs as you go.** Architecture Decision Records should be created alongside design decisions in Milestones 1–3, not deferred to Milestone 4.
- **Docs drive, code follows.** This repository contains *only* design documentation. The compiler and runtime live in a separate repository.
