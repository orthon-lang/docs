# Orthon Manifesto

## Introduction

Orthon is built on the belief that a programming language should be small in its core, consistent in its rules, and extensible by design. The language favors orthogonality over historical compatibility and composition over special cases.

## Manifesto

### Consistency over historical compatibility
The language evolves as a coherent system, not as an accumulation of legacy decisions.

### One concept — one syntax
A single idea should have a single canonical representation.

### One syntactic element — one responsibility
Every symbol, operator, and keyword has exactly one meaning regardless of context.

### Semantics over syntax
Behavior is expressed through modifiers rather than special-case syntax.

### Declarative over implicit
Code should express intent rather than implementation details.

### No privileged language features
Every capability available to the compiler should, whenever practical, be available to the programmer.

### Extensibility over built-in magic
Language evolution should primarily happen through libraries and user-defined modifiers instead of introducing new keywords.

### Composition over exceptions
New capabilities should emerge by composing existing mechanisms instead of adding special cases.

### A unified declaration model
Variables, functions, types, classes, and modules follow the same declaration principles and modifier system.

### Minimal core, maximum expressiveness
The language minimizes fundamental concepts while maximizing expressiveness through composition.

### Build once, deploy anywhere
The language produces an Execution Program — a self-contained,
semantically complete artifact — not platform-specific binaries.
Execution strategy is a deployment-time decision, not a build-time
constraint. The language owns the boundary between code and
environment, removing the need for external DevOps tooling to
reconstruct what the program requires.

## Guiding Vision

Orthon is designed around a small set of orthogonal concepts:

- One symbol — one role.
- One concept — one syntax.
- Modifiers define semantics.
- Composition replaces special cases.
- Language features are extensible.
- Readability comes from consistency rather than convention.

## Scope

This manifesto defines the philosophy of the language.

Implementation strategies, performance trade-offs, standard library design, optimization techniques, and programming idioms belong to the Orthon Zen and are intentionally outside the scope of this document.

The DevOps implications of the Execution Program model — deployment pipelines, engine selection, and operational concerns — follow from the principles above and are detailed in the Vision and Architecture documents.
