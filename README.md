# Orthon

**Orthon** is a programming language designed using the same engineering
principles it encourages in its users — SOLID architecture, a small
orthogonal core, explicit semantics, and implementation-independent
evolution.

At its architectural heart is the **Execution Program / Execution Engine**
separation — a paradigm shift on par with Docker's decoupling of
applications from operating systems. Docker made the *environment*
portable; Orthon makes the *program's semantic meaning* portable,
decoupling it from the execution strategy entirely.

Interpreters, AOT compilers, OCI builders, and WASM builders are all
equal citizens consuming the same intermediate artifact. You write
Orthon once and produce a single **Execution Program** — an executable
program environment, not a binary. You choose the execution mode at
deployment time: interpretation in development, AOT compilation in
production, OCI packaging for delivery, WASM for the edge — all from
the same artifact, without rebuilding.

This is not just a new way to compile programs. It redefines how
software is built, shipped, and operated — collapsing the boundary
between development and DevOps.

Every consequential design choice is recorded as an **Engineering Decision
Record (EDR)** — documented with context, rationale, and alternatives
considered. The project treats design decisions as first-class artifacts.

The language draws inspiration from Python (readability, approachable
syntax) and Java (explicit semantics, static type safety), keeping what
works and fixing what doesn't. Every feature follows the Principle of
Least Astonishment: behavior should match what a competent programmer
intuitively expects.

Orthon is also designed for the era of LLM-generated code. The same
properties that make it comfortable for humans — orthogonality,
consistent rules, and explicit semantics — make it uniquely suited for
AI-generated code. Every language construct must pass an LLM Generability
Gate that validates whether an LLM can reliably produce correct code
using it.

This repository contains the language design documentation — the *why*,
*how*, and *what* of Orthon. There is no compiler or runtime here yet.

## Getting Started

| Document | What it tells you |
|---|---|
| [Manifesto](why/MANIFESTO.md) | The core beliefs Orthon is built on |
| [Vision](why/VISION.md) | The five pillars — architectural design, human-centric comfort, LLM era readiness, the Execution Program model, and its DevOps transformation |
| [Working Backwards](why/WORKING_BACKWARDS.md) | Why Orthon exists — derived from the programmer's perspective |
| [Design Philosophy](how/PHILOSOPHY.md) | How design decisions flow from vision to implementation |
| [Architecture](how/architecture/ARCHITECTURE.md) | The layered design — Language Core, Execution Environment, LLM Toolchain |
| [Execution Model](what/concepts/EXECUTION_PROGRAM.md) | How Orthon unifies interpreters, compilers, and builders under one Execution Program + Engine model |
| [Core Concepts](what/concepts/CORE_CONCEPTS.md) | The fundamental building blocks — Data and Data Modifiers |
| [Decision Records](how/decision_records/INDEX.md) | Engineering Decision Records — every consequential design choice documented with rationale |
| [LLM-Native Toolchain](what/concepts/LLM_NATIVE_TOOLCHAIN.md) | How Orthon is designed for LLM-assisted development — schema, generation, analysis, and feedback |

## For AI Agents

This project is designed to be navigated and contributed to by AI agents.
See [AGENTS.md](AGENTS.md) for the agent protocol, document map, and
contribution workflow.

## License

[MIT](LICENSE)






