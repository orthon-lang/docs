# Orthon

**Orthon** is a programming language designed using the same engineering
principles it encourages in its users — SOLID architecture, a small
orthogonal core, explicit semantics, and implementation-independent
evolution.

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
| [Vision](why/VISION.md) | The three pillars — Language Designed Like Software, Learn from What Came Before, and Designed for the LLM Era |
| [Working Backwards](why/WORKING_BACKWARDS.md) | Why Orthon exists — derived from the programmer's perspective |
| [Design Philosophy](how/PHILOSOPHY.md) | How design decisions flow from vision to implementation |
| [Core Concepts](what/concepts/CORE_CONCEPTS.md) | The fundamental building blocks — Data and Data Modifiers |
| [LLM-Native Toolchain](what/concepts/LLM_NATIVE_TOOLCHAIN.md) | How Orthon is designed for LLM-assisted development — schema, generation, analysis, and feedback |

## For AI Agents

This project is designed to be navigated and contributed to by AI agents.
See [AGENTS.md](AGENTS.md) for the agent protocol, document map, and
contribution workflow.

## License

[MIT](LICENSE)






