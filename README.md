# Orthon

**Orthon** is an LLM-native programming language that applies the principles
of software engineering to the design of the language itself: a small
orthogonal core, explicit semantics, and a strict separation between
**what a program means** and **how it is realized**.

A program is a platform-independent semantic artifact. Execution —
interpretation, compilation, containerization — is an interchangeable
implementation detail, making portability, reproducibility, and delivery
language-level properties rather than pipeline problems. This decoupling
is realized through the **Execution Program / Execution Engine** separation:
Docker decoupled applications from operating systems, making the
*environment* portable; Orthon applies a similar decoupling — separating
a program's semantic meaning from its execution strategy, so the same
program can be interpreted, compiled, or containerized without modification.

You write Orthon once and produce a single **Execution Program** — an executable program environment, not a binary. You choose the execution mode at deployment time: interpretation in development, AOT compilation 
in production, OCI packaging for delivery, WASM for the edge — all from
the same artifact, without rebuilding.

This collapses the traditional boundary between development and
deployment: the artifact you debug in an interpreter is the same
artifact an OCI builder packages for production.

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
| [Goals](why/GOALS.md) | Concrete aims derived from the vision — what Orthon aims to achieve and what it explicitly does not |
| [Working Backwards](why/WORKING_BACKWARDS.md) | Why Orthon exists — derived from the programmer's perspective |
| [Design Philosophy](how/PHILOSOPHY.md) | How design decisions flow from vision to implementation |
| [Architecture](how/architecture/ARCHITECTURE.md) | The layered design — Language Core, Execution Environment, LLM Toolchain |
| [Core Concepts](what/CORE_CONCEPTS.md) | Registry of accepted Orthon concepts — the foundation for the language spec |
| [Decision Records](how/decision_records/INDEX.md) | Engineering Decision Records — every consequential design choice documented with rationale |

## For AI Agents

This project is designed to be navigated and contributed to by AI agents.
See [AGENTS.md](AGENTS.md) for the agent protocol, document map, and
contribution workflow.

## License

[MIT](LICENSE)






