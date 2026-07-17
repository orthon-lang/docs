# Orthon Glossary

> A unified reference of all project terminology.
> Each entry includes a definition, cross-references to source documents, and links to related terms.

---

## A

### Architecture

The layered structure of Orthon: Core Language → Syntax → Standard Library → Implementation Strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md`
- **See also:** [Core Language](#core-language), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

---

## C

### Canonical Form

One of the equivalent syntactic ways to express a language construct. All canonical forms of a feature must be documented together (see *Show All Canonical Forms* principle).

- **Source:** `DESIGN_PRINCIPLES.md` § Documentation Principle
- **See also:** [Operator Equivalence](#operator-equivalence)

### Core Language

The minimal, stable set of language semantics, independent of any particular implementation. The Core defines *what programs mean*, not *how they execute*.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Core Language, `DESIGN_PRINCIPLES.md` § Minimal Core
- **See also:** [Architecture](#architecture), [Implementation Strategy](#implementation-strategy), [Standard Library](#standard-library)

---

## D

### Data

The primary abstraction in Orthon. Values viewed without imposed semantic meaning — the raw material that modifiers transform.

```
(1, 2, 3)    → data, without semantic label
tuple(1, 2, 3) → the same data, now explicitly a Tuple
```

- **Source:** `concepts/CORE_CONCEPTS.md` § Data, `concepts/DATA_MODEL.md`
- **See also:** [Data Modifier](#data-modifier), [Representation](#representation)

### Data Modifier

A construct that transforms data from one representation to another. Modifiers express programmer intent; the compiler determines the most efficient implementation.

- **Source:** `concepts/CORE_CONCEPTS.md` § Data Modifiers
- **See also:** [Data](#data), [Representation](#representation)

### Declaration Model (Unified)

Variables, functions, types, classes, and modules follow the same declaration principles and modifier system. No special-case declaration syntax for different kinds of entities.

- **Source:** `../why/MANIFESTO.md` § A unified declaration model

### Deterministic Behavior

The same source code must produce identical observable behavior across optimization levels and implementations. Only performance characteristics may differ.

- **Source:** `DESIGN_PRINCIPLES.md` § Deterministic Behavior
- **See also:** [Explicit Semantics](#explicit-semantics), [Semantics Before Optimization](#semantics-before-optimization)

---

## D (cont.)

### Decision Validation

A framework of six independent validation gates that every language
design decision must pass. Each gate examines the proposal from a
different perspective: user value, logical consistency, conceptual
simplicity, architectural integrity, implementation independence,
and long-term maintainability.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Language Design Gate](#language-design-gate), [Validation Gate](#validation-gate)

---

## E

### Explicit Optimization

Performance-oriented execution strategies are enabled intentionally by the programmer, never applied silently. The default execution model favors predictability over performance.

- **Source:** `DESIGN_PRINCIPLES.md` § Explicit Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Semantics Before Optimization](#semantics-before-optimization)

### Explicit Semantics

Whenever an operation changes the meaning, lifetime, ownership, or behavior of data, it must be expressed explicitly in the syntax. No hidden conversions or implicit semantic changes.

- **Source:** `DESIGN_PRINCIPLES.md` § Explicit Semantics
- **See also:** [Deterministic Behavior](#deterministic-behavior)

---

## I

### Implementation Strategy

A concrete backend that implements language semantics (compiler backend, runtime, interpreter, etc.). Strategies are interchangeable: replacing one must not change program semantics.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Implementation Strategy, `../how/strategies/IMPLEMENTATION_STRATEGIES.md`
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Standard Library](#standard-library)

### Intent Over Implementation

The programmer describes *what* should happen; the compiler decides *how* to implement it efficiently.

- **Source:** `DESIGN_PRINCIPLES.md` § Intent Over Implementation
- **See also:** [Explicit Semantics](#explicit-semantics)

---

## L

### Language Consistency Principles

The set of cross-cutting design rules that ensure uniform behaviour across Orthon:

- **Representation Symmetry** — Reversible transformations use symmetric syntax (prefix creates a representation, postfix restores it).
- **Deterministic Behavior** — Same source, same observable behaviour (see [Deterministic Behavior](#deterministic-behavior)).
- **Stable Mental Model** — Programmers reason about language semantics, not compiler internals.

- **Source:** `DESIGN_PRINCIPLES.md` § Language Consistency

---

## M

### Minimal Core

Complex language features should emerge from composition of simple primitives rather than from introducing new keywords or execution models. The language is *grown*, not *invented*.

- **Source:** `DESIGN_PRINCIPLES.md` § Minimal Core, `../why/MANIFESTO.md` § Minimal core, maximum expressiveness
- **See also:** [Orthogonality](#orthogonality)

---

## N

### Named Before Symbolic

Every symbolic operator must have an equivalent named function. Symbols improve brevity; named functions improve readability. Both express the same semantics.

Example: `&x` and `ref(x)` are equivalent.

- **Source:** `DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Operator Equivalence](#operator-equivalence)

---

## O

### Operator Equivalence

Every language operator has an equivalent named function. Operators are syntactic sugar over ordinary functions.

```
&x          == ref(x)
x->         == sequence(x)
```

- **Source:** `concepts/CORE_CONCEPTS.md` § Operators and Named Functions, `DESIGN_PRINCIPLES.md` § Named Before Symbolic
- **See also:** [Canonical Form](#canonical-form), [Named Before Symbolic](#named-before-symbolic)

### Orthogonality

Each language construct solves exactly one problem and combines freely with other constructs. No special cases, no context-dependent syntax, no conflicting rules. What you learn in one part of the language transfers directly to every other part.

- **Source:** `DESIGN_PRINCIPLES.md` § Orthogonality, `../why/VISION.md` § Comfortable by Design, `../why/ZEN.md`, `../why/MANIFESTO.md`
- **See also:** [Minimal Core](#minimal-core), [Uniformity](#uniformity)

---

## P

### Principle of Least Astonishment

The behaviour of a construct must match what a competent programmer intuitively expects. Surprises belong in discovery, not in semantics.

- **Source:** `../why/VISION.md` § Comfortable by Design
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

---

## R

### Representation

A specific view of data. Orthon provides several fundamental representations:

- **Value** — A single, immutable value.
- **Tuple** — A fixed-size ordered collection.
- **Reference** — A handle to data (`&x` / `ref&`).
- **Sequence** — Values produced over time (see [Sequence](#sequence)).
- **Set** — An unordered collection of unique values.
- **Option** — A value that may be absent.
- **Result** — A value that may be an error.

- **Source:** `concepts/CORE_CONCEPTS.md` § Fundamental Data Types
- **See also:** [Data](#data), [Data Modifier](#data-modifier)

### Representation Symmetry

Reversible transformations use symmetric syntax. Prefix form creates an alternative representation; postfix form restores the original.

```
&value   → Reference
ref&     → Value

*values  → Pack
pack*    → Values
```

- **Source:** `DESIGN_PRINCIPLES.md` § Representation Symmetry
- **See also:** [Explicit Semantics](#explicit-semantics), [Named Before Symbolic](#named-before-symbolic)

---

## S

### Semantics Before Optimization

Language semantics are independent of optimization. A correct Orthon program must produce the same observable behavior regardless of optimization level.

- **Source:** `DESIGN_PRINCIPLES.md` § Semantics Before Optimization
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Optimization](#explicit-optimization)

### Sequence

A fundamental type representing a sequence of values produced over time. Unlike traditional generators or streams, a Sequence describes *what the result is*, not *how it is produced*. It is a normal object: it can be returned, stored, passed, transformed, or consumed incrementally.

- **Source:** `concepts/CORE_CONCEPTS.md` § Sequence
- **See also:** [Representation](#representation)

---

## V

### Validation Gate

One of six independent perspectives used in [Decision Validation](#decision-validation)
to assess a language design proposal. Each gate has its own criteria,
pass/fail conditions, and scope of examination.

- **Source:** `../how/gates/DECISION_VALIDATION.md`
- **See also:** [Decision Validation](#decision-validation), [Language Design Gate](#language-design-gate)

### Stable Mental Model

Programmers should reason about language semantics, not compiler internals. Users should never need to understand implementation details to predict program behaviour.

- **Source:** `DESIGN_PRINCIPLES.md` § Stable Mental Model
- **See also:** [Deterministic Behavior](#deterministic-behavior), [Explicit Semantics](#explicit-semantics)

### Standard Library

The layer that defines public abstractions exposed by the language. It specifies *behaviour*, not *implementation*. The Standard Library is the interface between user code and the Implementation Strategy.

- **Source:** `../how/architecture/ARCHITECTURE.md` § Standard Library
- **See also:** [Architecture](#architecture), [Core Language](#core-language), [Implementation Strategy](#implementation-strategy)

---

## U

### Uniformity

Equivalent concepts should be expressed in equivalent ways. Once a user learns a language pattern, the same pattern applies consistently throughout the language.

- **Source:** `DESIGN_PRINCIPLES.md` § Uniformity
- **See also:** [Orthogonality](#orthogonality)

---

## Z

### Zen of Orthon

The four guiding aphorisms of the language:

> Every special case creates complexity.
> Orthogonality removes exceptions.
> Consistency defeats complexity.
> Simplicity is the result of orthogonality.

- **Source:** `../why/ZEN.md`
- **See also:** [Orthogonality](#orthogonality)

---

## How to Maintain This Glossary

1. **Add terms proactively.** When a new concept is introduced in any design document, add it here.
2. **Keep definitions in sync.** If a term's meaning evolves, update this file and all source documents simultaneously.
3. **Prefer cross-references over duplication.** The Glossary defines *what* a term means; source documents explain *why and how* it matters.
4. **Alphabetical order.** Each section uses the term's first letter as a heading anchor.
