# Vision

> **Canonical block.** This document is the single source of truth for
> Orthon's vision. Expanded discussions follow below.

```
Vision
    A programming language engineered like software — architected with
    SOLID principles, orthogonal by construction, and designed for both
    humans and LLMs as first-class users.

Mission
    Provide minimal, orthogonal, explicit semantics that compose freely.
    The language defines contracts; implementations fulfill them.
    Programs are fully-defined artifacts, not just source code.

Non-goals
    • Be another C syntax
    • Be the fastest language
    • Replace C++
    • Novelty for its own sake
    • Backward compatibility with any existing language
    • REPL-first or notebook-first interaction

Target Audience
    • LLMs — as first-class code generators
    • Library authors — who value stable semantics and composable abstractions
    • System programmers — who need explicit control without ceremony
    • Teams — who need decade-stable language foundations

Success Criteria
    • Stable semantics — programs from v1.0 run correctly on v10.0
    • Small core — all features decompose to a minimal primitive set
    • Predictable evolution — extensions through libraries, not core changes
    • LLM generability — LLMs can reliably produce correct Orthon code
    • Execution portability — same program runs identically in any conforming Engine
```

## Language Designed Like Software

The language should be architected like well-designed software.

Orthon is a programming language designed using the same engineering
principles it encourages developers to use when building software.

The language architecture follows SOLID principles.

The Core Language and Standard Library define interfaces and contracts;
the Implementation Strategy fulfills them. User programs depend on
language interfaces rather than implementation details. Evolution happens
primarily through new strategies and libraries instead of changes to the
language core.

## Learn from What Came Before

Orthon acknowledges its intellectual debt to Python and Java — and does
not repeat their mistakes.

From Python, Orthon inherits the value of readability and approachable
syntax, but rejects significant whitespace, scope ambiguity, unchecked
runtime type errors, and the concurrency model that surrendered
parallelism for an illusion of simplicity.

From Java, Orthon inherits the value of explicit semantics, static type
safety, and disciplined architecture, but rejects ceremonial verbosity,
the everything-is-an-object dogma, checked exception fatigue, and a
standard library that resisted modern paradigms for decades.

Orthon studies its predecessors not to imitate them, but to stand on
their shoulders — keeping what worked, fixing what didn't, and designing
for the lessons only hindsight provides.

## Comfortable by Design

Orthon aims to be a comfortable programming language — one where
constructs feel obvious, familiar, and predictable. Reading Orthon code
should require less effort than writing it.

The Principle of Least Astonishment governs every feature: the behavior
of a construct should match what a competent programmer intuitively
expects. Surprises belong in discovery, not in semantics.

This comfort is reinforced by **orthogonality**. Each language construct
solves one problem and combines freely with others. There are no special
cases, no context-dependent syntax, and no conflicting rules to
memorize. What you learn in one part of the language transfers directly
to every other part.

## Designed for the LLM Era

Orthon is designed for an era where code is increasingly written by
large language models — not instead of human programmers, but alongside
them. The same properties that make Orthon comfortable for humans make
it uniquely suited for AI-generated code.

### Small surface, consistent rules

LLMs predict code probabilistically. Every special case, exception, and
context-dependent rule expands the prediction space and increases the
chance of error. Orthon's orthogonal design minimizes this surface.
There are no special cases to hallucinate, no hidden conversions to
forget, and no context-dependent syntax to confuse.

What an LLM learns in one part of the language transfers directly to
every other part — the same principle that benefits human readers.

### Intent, not implementation

LLMs are better at describing *what* a program should do than at
making low-level implementation decisions. Orthon's
Intent-Over-Implementation philosophy meets this strength directly:
the LLM expresses intent through declarative constructs, modifiers,
and standard library calls; the compiler and Implementation Strategy
handle optimization, allocation, and execution details.

### Explicit semantics reduce hallucination

Orthon's Explicit Semantics principle means every behavior-changing
operation is visible in the syntax. An LLM generating Orthon code
cannot accidentally introduce implicit behavior — there is none.
What the LLM writes is exactly what the program does.

### Execution Program as LLM-native artifact

The Execution Program model dissolves the gap between "write code"
and "run code." An LLM can produce not just source code, but a
fully-defined program — a self-contained, reproducible artifact
that includes everything needed to execute it. The language handles
the boundary between code and environment; the LLM focuses on
expressing intent.

### Extensibility without language changes

Because Orthon evolves through libraries and Implementation
Strategies rather than new keywords, an LLM can generate code that
uses new capabilities without learning a changing language. The
semantic core remains stable; expressiveness grows through the
Standard Library and strategy profiles.

### A foundation, not a conclusion

The LLM era is just beginning. Orthon's design — orthogonality,
explicit semantics, intent over implementation, Execution Program —
provides a foundation that can grow with it. The language does not
claim to solve all problems of AI-generated code. It claims to be
designed for an age where those problems matter.

## Execution Program

Orthon believes a program should be fully defined — not just its source
code, but the context required to execute it. Language version, runtime,
dependencies, permissions, and resource requirements are part of the
program's definition, not external infrastructure bolted on after the
fact.

This extends the Principle of Explicitness from syntax to execution. A
program that declares what it needs is reproducible by construction,
portable across environments without modification, and debuggable
without guessing about its surroundings.

The language defines the contract, not the mechanism. Different
targets — development, testing, production — may fulfill the same
contract differently. But the program remains the same artifact,
regardless of where or how it runs. Portability is a property of how
the program is defined, not a configuration problem for deployment
pipelines to solve.

## Goals

The vision above defines six concrete goals, documented in
[`GOALS.md`](GOALS.md) with criteria and non-goals:

1. **Architectural Integrity** — the language itself is a SOLID system.
2. **Inherited Wisdom** — keep Python/Java's strengths, fix their flaws.
3. **Comfortable by Construction** — reading is easier than writing;
   Principle of Least Astonishment.
4. **LLM Readiness** — the same properties that help humans help LLMs.
5. **Execution Integrity** — programs are fully-defined, portable
   artifacts.
6. **Long-term Stability** — stable for decades; implementations
   improve without breaking existing programs.
