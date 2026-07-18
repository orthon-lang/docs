# Architecture

> Orthon is a programming language designed using the same engineering
> principles it encourages developers to use when building software.
> The language architecture follows the same principles — SOLID, separation
> of concerns, abstraction, and dependency inversion — that developers use
> to design maintainable software.
>
> This principle extends across the entire platform: the Core Language
> defines interfaces and contracts; the Implementation Strategy fulfills them.
> The Standard Library becomes the analog of a set of interfaces, and each
> Implementation Strategy provides the concrete implementations.
> This pattern applies to memory allocation, expression evaluation,
> algorithm selection, object representation, and every other execution
> mechanism in the language.

## Overview

Orthon is structured as a layered architecture with a clear separation
between language definition and implementation.

``` mermaid
flowchart TD
    UC["User code"]

    subgraph LANG["Language"]
        CL["Core Language"]
        SX["Syntax"]
        SL["Standard Library"]
    end

    subgraph STRAT["Implementation Strategy"]
        direction LR
        STRAT_LBL["<b>Strategy</b><br/><i>named set of Policies</i>"]
    end

    subgraph EXEC["Execution Environment"]
        C["Compiler"]
        R["Runtime"]
        P["Platform"]
    end

    UC --> LANG
    LANG --> STRAT
    STRAT --> EXEC
```

## Layers

### Language

Contains three components that together define what the language provides
and how it is perceived by the programmer.

**Core Language** — defines the semantic contracts of the language: the
interfaces and rules that every implementation must fulfill. The core
specifies *what programs mean*, not *how they execute*. It is minimal,
stable, and implementation-independent. No implementation detail leaks
into the core.

**Syntax** — the human interface to the language. It should be obvious,
consistent, predictable, and free from implementation details.

**Standard Library** — defines the public interface contracts exposed by
the language. It specifies *behavior* and *contract* — not implementation.
It is the analog of a set of interfaces in classical software engineering:
user code programs against the Standard Library, not against any concrete
strategy.

### Implementation Strategy

The **Implementation Strategy** bridges language semantics and the
execution environment through a set of **Policies** — declarative
choices in domain-specific areas such as memory allocation, algorithm
selection, evaluation mode, lifetime management, and concurrency.

A **Strategy** is a named set of Policies. Each Policy is a
declarative constraint within its own area of responsibility.
Strategies are interchangeable: replacing one must not change program
semantics.

This relationship forms a **resolution chain**:

```
Language Layer (what the program means)
├── Core Language — semantic primitives (minimal, stable)
├── Standard Library — interface contracts (extensible)
└── Syntax — human interface
        │
        ▼
Implementation Strategy (named set of Policies)
        │
        ├── Allocation Policy
        ├── Algorithm Policy
        ├── Evaluation Policy
        ├── Lifetime Policy
        ├── Concurrency Policy
        └── ...
        │
        ▼
Concrete Implementation
```

**Language Semantics** defines *what* the program guarantees — for
example, `sort(data)` guarantees sortedness and stability if
specified, but never dictates *which* sorting algorithm is used.

**Policies** decide *how* each concern is realised. A Policy is
declarative, not procedural: it states a preference (e.g.,
`Allocation Policy = Arena`) rather than prescribing a conditional
branch (`if speed then TimSort else MergeSort`).

**Implementation** makes the final concrete choice, constrained by
the active set of Policies.

Different strategies may optimise for performance, memory usage,
startup time, determinism, embedded systems, or parallel hardware.
For concrete profiles, see
[`DEFAULT_STRATEGY.md`](../strategies/DEFAULT_STRATEGY.md),
[`EMBEDDED_STRATEGY.md`](../strategies/EMBEDDED_STRATEGY.md), and
[`HIGH_PERFORMANCE_STRATEGY.md`](../strategies/HIGH_PERFORMANCE_STRATEGY.md).

This decoupling is validated by the
[`IMPLEMENTATION_INDEPENDENCE_GATE`](../gates/DECISION_VALIDATION.md#gate-catalogue):
language concepts must be definable without tying to any specific
Strategy or Policy.

### Execution Environment

The architectural layer that hosts the **Concrete Implementation** — the final link in the resolution chain. Consists of:

**Compiler** — translates the program into executable form.

**Runtime** — provides services like memory management, scheduling, and
I/O that the Strategy relies on.

**Platform** — the underlying system: operating system, virtual machine,
WebAssembly runtime, or bare metal.

## Design Principles

Architecture follows SOLID. See
[`what/DESIGN_PRINCIPLES.md`](../what/DESIGN_PRINCIPLES.md#software-design-principles)
for the canonical mapping of each principle to the language layers.

## Scope of the Pattern

The interface/implementation separation applies to virtually every
mechanism in the language. See
[`IMPLEMENTATION_STRATEGIES.md`](../strategies/IMPLEMENTATION_STRATEGIES.md)
for the canonical mapping of each aspect to possible strategy implementations.

Each aspect is defined at the language level as a contract. The choice of
implementation is deferred to the strategy layer. This guarantees that
program semantics are independent of the execution strategy.

## Evolution Model

1.  Keep the Core Language minimal and stable.
2.  Add capabilities through the Standard Library.
3.  Improve execution through new Implementation Strategies.
4.  Extend the language itself only when higher abstraction layers
    cannot solve the problem.

### Semantic ISA

This evolution model mirrors the role of Instruction Set Architectures
in computing.

Assembly language is a stable interface between the program and the
processor. The ISA does not change when a new microarchitecture is
introduced — the same instructions run on different hardware
implementations.

Orthon Core is designed as a **Semantic ISA**: a stable semantic
interface between the program and any implementation. Just as a
compiler translates assembly into micro-ops for a specific CPU, the
Orthon compiler translates Core semantics into a concrete
implementation guided by the active Implementation Strategy.

| ISA Concept | Orthon Analogy |
|---|---|
| Instruction set | Semantic primitives (Core Language) |
| Microarchitecture | Implementation Strategy (Policies) |
| System library | Standard Library contracts |
| Silicon / hardware | Platform implementation |

A RISC-like philosophy applies: a small, stable set of semantic
primitives, with nearly all evolution happening in the strategy and
library layers around them.

## Goal

This architecture serves the goal described in
[`VISION.md`](../../why/VISION.md): keeping the language stable for
decades while allowing implementations to continuously improve without
breaking existing programs.

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
