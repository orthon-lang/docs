# Language Design Principles

```
Design Principles
├── Orthogonality
├── Simplicity
├── Explicitness
├── Consistency
└── Documentation Principles
      ├── Show All Canonical Forms
      ├── One Concept — One Place
      ├── Examples Before Theory
      └── Explain the Rationale
```

# Core Philosophy

## Data First

Data is the primary abstraction.

Language constructs operate by transforming data into different representations.


## Intent Over Implementation

The programmer describes what should happen.

The compiler decides how to implement it efficiently.


## Orthogonality

Each language construct should solve exactly one problem.

Language features should be composable rather than overlapping. Complex
behavior should emerge from combining simple, orthogonal building
blocks.


## Uniformity

Equivalent concepts should be expressed in equivalent ways.

Once a user learns a language pattern, the same pattern should apply
consistently throughout the language.


## Explicit Semantics

Whenever an operation changes the meaning, lifetime, ownership, or
behavior of data, it should be expressed explicitly in the syntax.

The language should avoid hidden conversions and implicit semantic
changes.

## Named Before Symbolic

Every symbolic operator should have an equivalent named function.

Symbols improve brevity.

Named functions improve readability.

Both represent exactly the same semantics.


## Minimal Core

Complex language features should emerge from composition of simple primitives rather than from introducing new keywords or execution models.



# Language Consistency

## Representation Symmetry

Reversible transformations should use symmetric syntax.

Unary operators transform **representations**, not values.

-   Prefix form creates an alternative representation.
-   Postfix form restores the original representation.

Examples:

``` text
&value   → Reference
ref&     → Value

*values  → Pack
pack*    → Values
```

Any reversible transformation of a representation should be expressed
using symmetric syntax.

------------------------------------------------------------------------

## Deterministic Behavior

The same source code should behave identically across optimization
levels and implementations.

Program behavior must not depend on hidden implementation details such
as object interning, escape analysis, copy elision, object pooling, or
implementation-specific caching.

Only performance characteristics may differ.

------------------------------------------------------------------------

## Stable Mental Model

Programmers should reason about language semantics, not compiler
internals.

Users should never need to understand implementation details to predict
program behavior.

# Execution Model

## Semantics Before Optimization

Language semantics are independent of optimization.

A correct Orthon program must produce the same observable behavior
regardless of optimization level. Optimization may improve performance,
memory usage, or code generation, but it must never alter the program's
meaning.

------------------------------------------------------------------------

## Explicit Optimization

Optimization is explicit, never implicit.

Performance-oriented execution strategies are enabled intentionally by
the programmer rather than silently applied by the language.

The default execution model favors predictability over performance.

------------------------------------------------------------------------

## Correctness Before Performance

Correctness always takes precedence over performance.

Performance improvements are valuable only when they preserve clarity,
determinism, and the defined semantics of the language.

---

# Documentation Principle: Show All Canonical Forms

When documenting a language feature, always present all canonical ways to use it, not just the most common one.

If a feature has multiple equivalent forms, they should be shown together in the same section.

# Example

Generators can be created in two ways:

- **Generator function** — a function using yield.
- **Generator expression** — (x for x in range(n)).

## Rationale

Documentation should be complete rather than incremental. After reading a section, the reader should know every official way to accomplish the described task, without having to discover alternative forms later in unrelated chapters.
