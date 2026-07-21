# Foundational Abstractions

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It presents a hypothesis about Orthon's foundational abstractions
> that has **not yet passed Concept Design Review** and is **not
> accepted via EDR**. This claim requires validation before it can be
> considered settled design.
>
> **Last updated:** 2026-07-21
>
> **See also:**
> - [`DATA_MODEL.md`](DATA_MODEL.md) — Detailed data model research (representations, default strategy)
> - [`GENERATORS.md`](GENERATORS.md) — Generator/yield research (alternative to `emit`)
> - [`FUNCTIONS.md`](FUNCTIONS.md) — Function model research (operator equivalence)
> - [`ARCHITECTURE.md`](../../architecture/ARCHITECTURE.md) § Semantic Dependency Architecture

## Issue (Why)

Every programming language has foundational abstractions — the primitive
building blocks from which all other constructs are composed. The core
problem: **what are Orthon's foundational abstractions**, such that every
language construct can be expressed as a composition of them, without
special cases, ad-hoc syntax, or fragmented mental models?

Languages provide different answers: Lisp has S-expressions (code as data);
Smalltalk has objects and messages; C has values, pointers, and structs.
Each answer shapes what is easy, what is hard, and what is possible in the
language.

Orthon's hypothesis is that **two abstractions suffice**: Data (what exists)
and Data Modifiers (how it is viewed). This is a claim that needs validation
through the Concept Design Review process.

## Principles

1. **Minimal Core** — Two fundamental abstractions should suffice. Every
   construct is built from Data and Data Modifiers. See
   [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Minimal Core.
2. **Orthogonality** — Data and Data Modifiers combine freely. Any Data
   can be viewed through any Modifier without special cases. See
   [`../../what/DESIGN_PRINCIPLES.md`](../../what/DESIGN_PRINCIPLES.md)
   § Orthogonality.
3. **Explicitness** — Representation changes are visible in the syntax.
   The programmer explicitly declares intent; the compiler optimises.
4. **Composition over Exceptions** — No special-purpose types or syntax.
   The two abstractions compose to produce all language features.

## Hypothesis: Two Fundamental Concepts

Orthon's foundational abstractions are:

1. **Data** — The primary abstraction. Values without imposed semantic
   meaning. Data is what exists.
2. **Data Modifiers** — Constructs that transform data from one
   representation to another. Modifiers express programmer intent;
   the compiler determines the most efficient implementation.

**Claim:** Every language construct can be expressed as a combination of
Data and Data Modifiers. No third fundamental abstraction is needed.

**Requires validation:**
- Is this truly complete? Can constructs like functions, control flow,
  error handling, and concurrency be expressed purely through Data and
  Data Modifiers?
- Are there edge cases where a third abstraction (e.g., executable code,
  metadata, or effects) is irreducible to Data + Data Modifiers?
- Does the LLM Generability Gate (§ LLM Generability) confirm this is
  learnable from two concepts?

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Representation Policy | Defines how data representations are mapped to runtime memory layouts |
| Allocation Policy | Determines where data values are stored (stack, heap, arena) |
| Equality Policy | Controls how data values are compared (structural vs. identity) |
| Evaluation Policy | Governs when data is produced (eager vs. lazy in Sequence) |

## Model (What)

### Data

Data is the primary abstraction. A bare collection of values without
imposed semantic meaning:

```
(1, 2, 3)
```

represents a collection of values without assigning any specific semantic
meaning. It is simply data — it becomes meaningful only when a Data Modifier
interprets it.

### Data Modifiers

A modifier transforms data into another representation, adding semantic
meaning. Examples:

```
tuple(1, 2, 3)       → Tuple representation
set(1, 2, 3)         → Set representation
sequence(1, 2, 3)    → Sequence representation
pack(1, 2, 3)        → Packed value representation
```

Each modifier expresses intent, while the compiler determines the most
efficient implementation.

### Fundamental Data Representations

> **Note:** The seven data representations below are already detailed in
> [`DATA_MODEL.md`](DATA_MODEL.md) § Model (What) and § Decision History.
> This section summarises the hypothesis; see `DATA_MODEL.md` for the
> full treatment including default strategy and alternatives.

Orthon provides several fundamental data representations, each created
by a Data Modifier:

- **Value** — A single, self-contained value
- **Tuple** — An ordered, fixed-size collection
- **Reference** — A handle to data elsewhere in memory
- **Sequence** — An ordered, variable-size collection (lazy or eager)
- **Set** — An unordered collection with uniqueness
- **Option** — A value that may be absent
- **Result** — A value that may be an error

These are not special language mechanisms. They are simply different
representations of data, accessed through Data Modifiers.

### Sequence and the `emit` Keyword

> **Open question:** This section presents an Orthon-specific design
> proposal. [`GENERATORS.md`](GENERATORS.md) proposes `yield` instead
> — a competing design choice. Both approaches need comparison during
> Concept Design Review.

Sequence is a fundamental representation: values produced over time.
Unlike traditional generators or streams, Sequence describes *what*
the result is, not *how* it is produced.

A Sequence can be:
- returned from functions;
- stored in variables;
- passed to other functions;
- transformed with library operations;
- consumed incrementally.

#### Canonical Forms

Three equivalent ways to produce a Sequence (the *Show All Canonical
Forms* principle — see [`DESIGN_PRINCIPLES.md`](../../what/DESIGN_PRINCIPLES.md)):

Readable syntax:
```
emit value
```

Compiler expansion:
```
return sequence(value)
```

Operator form:
```
return value ->
```

All three forms are semantically equivalent.

#### Compiler Behavior

If a function contains `emit`, the compiler automatically:
- creates a Sequence;
- transforms the function into a resumable state machine;
- preserves execution state between emitted values;
- performs all implementation-specific optimizations.

This behavior is entirely transparent to the programmer.

#### Sequence Typing

Sequence does not require explicit element types. Example:

```
numbers():
    emit 1
    emit 2
    emit 3
```

or

```
mixed():
    emit 1
    emit true
    emit "Hello"
```

The compiler infers the most appropriate internal representation. Static
type information remains an implementation detail rather than part of the
language syntax.

### Operators and Named Functions

> **See also:** [`FUNCTIONS.md`](FUNCTIONS.md) for the full function model
> research.

Every language operator has an equivalent named function. Examples:

```
&x          == ref(x)
x->         == sequence(x)
```

This principle provides:
- consistent semantics;
- easier metaprogramming;
- a minimal set of language primitives;
- readable APIs without relying on symbolic operators.

Operators are syntactic sugar over ordinary functions.

## Open Questions

1. **Completeness of the hypothesis.** Are Data and Data Modifiers truly
   sufficient as the only two fundamental abstractions? Can functions,
   control flow, error handling, modules, and concurrency all be expressed
   through composition of these two?

2. **`emit` vs `yield`.** The `emit` keyword proposed here is an
   Orthon-specific design choice. [`GENERATORS.md`](GENERATORS.md) proposes
   `yield` — closer to Python/Ruby convention. Which better serves Orthon's
   goals (learnability, LLM generability, consistency)? Which reads more
   naturally in the three canonical forms?

3. **Operator completeness.** Should *all* operators have named function
   equivalents? Are there operators that should remain symbolic-only
   (e.g., indexing, attribute access)? What is the rule for drawing the
   boundary?

4. **Seven representations — complete set?** The seven representations
   listed in § Model are a hypothesis. Are there missing representations
   (e.g., sum types, maps/dictionaries)? Are there redundant ones?
   See [`DATA_MODEL.md`](DATA_MODEL.md) § Open Questions.

5. **Relationship to Semantic Dependency Architecture.** The claim that
   Data and Data Modifiers form Level 0 (Data Model) of the Semantic
   Dependency Architecture (see [`ARCHITECTURE.md`](../../architecture/ARCHITECTURE.md)
   § Semantic Dependency Architecture) needs verification. Do Primitive
   Operations (Level 1) truly depend only on Data and Data Modifiers?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
