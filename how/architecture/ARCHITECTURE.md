# Architecture

## Overview

Orthon is structured as a layered architecture with a clear separation
between language definition and implementation.

``` text
User Program
      │
      ▼
Language Syntax
      │
      ▼
Standard Library (Interfaces)
      │
      ▼
Implementation Strategy (Implementations)
      │
      ▼
Compiler / Runtime / Platform
```

## Layers

### Core Language

Defines the semantics of the language: what programs mean, not how they
execute. The core is minimal, stable, and implementation-independent.

### Language Syntax

The human interface to the language. It should be obvious, consistent,
predictable, and free from implementation details.

### Standard Library

Defines the public abstractions exposed by the language. It serves as
the interface between user code and the implementation. It specifies
behavior, not implementation.

### Implementation Strategy

Provides concrete implementations of language and library abstractions.
Different strategies may optimize for performance, memory usage, startup
time, determinism, embedded systems, or parallel hardware. Replacing a
strategy must not change program semantics.

## Design Principles

-   **Single Responsibility** --- each layer has one responsibility.
-   **Open/Closed** --- the core remains stable while implementations
    evolve.
-   **Liskov Substitution** --- strategies are interchangeable.
-   **Interface Segregation** --- user code depends only on language
    interfaces.
-   **Dependency Inversion** --- implementations depend on abstractions,
    not the other way around.

## Evolution Model

1.  Keep the Core Language minimal and stable.
2.  Add capabilities through the Standard Library.
3.  Improve execution through new Implementation Strategies.
4.  Extend the language itself only when higher abstraction layers
    cannot solve the problem.

## Goal

Keep the language stable for decades while allowing implementations to
continuously improve without breaking existing programs.

## Fitness Functions

> Architectural fitness functions guard against decay of the core
> design. These metrics are checked when a concept is revised or a new
> concept is introduced.

- **Concept stability.** A concept that requires more than one
  substantive semantic change after its initial Gate approval triggers
  an architectural review. Frequent semantic revision of a single
  concept indicates the concept or its surrounding abstractions are
  poorly factored. (See
  [`what/DESIGN_PRINCIPLES.md`](../../what/DESIGN_PRINCIPLES.md) —
  Transparency principle.)

- **Layered isolation.** A change to one layer (Core Language, Standard
  Library, Implementation Strategy) must not ripple into another layer's
  semantics. If a strategy change forces a concept-level redefinition,
  the layer boundary is breached.

- **Composition surface.** The number of documented canonical forms for
  a concept must not decrease across revisions — composition should
  expand, not contract.

*Additional fitness functions may be added here as the language evolves.*
