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

    IS["Implementation Strategy"]

    subgraph EXEC["Execution Environment"]
        C["Compiler"]
        R["Runtime"]
        P["Platform"]
    end

    UC --> LANG
    LANG --> IS
    IS --> EXEC
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

Provides concrete implementations of language and library abstractions.
Different strategies may optimize for performance, memory usage, startup
time, determinism, embedded systems, or parallel hardware. Replacing a
strategy must not change program semantics.

### Execution Environment

The target where the Strategy runs. Consists of:

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
mechanism in the language:

| Aspect | Interface (Language / StdLib) | Implementation (Strategy) |
|---|---|---|
| **Memory allocation** | Declarative allocation semantics | Heap, arena, linear, or GC-backed allocator |
| **Expression evaluation** | Expression semantics and composition | AST walker, bytecode VM, JIT, or interpreter |
| **Algorithm selection** | What the program expresses | Sorting, searching, hashing strategy implementations |
| **Object representation** | Data model and structural contracts | Struct-of-fields, SOA, tagged unions, or pointer-based layout |
| **Concurrency model** | Task and communication semantics | Thread pool, work-stealing, event loop, or hardware dispatch |
| **Error handling** | Result and error propagation model | Stack unwinding, error codes, or return-value branching |

Each aspect is defined at the language level as a contract. The choice of
implementation is deferred to the strategy layer. This guarantees that
program semantics are independent of the execution strategy.

## Evolution Model

1.  Keep the Core Language minimal and stable.
2.  Add capabilities through the Standard Library.
3.  Improve execution through new Implementation Strategies.
4.  Extend the language itself only when higher abstraction layers
    cannot solve the problem.

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
