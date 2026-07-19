# Language Design Principles

```
Core Philosophy
├── Data First
├── Intent Over Implementation
├── Orthogonality
├── Simplicity
├── Explicitness
├── Consistency
├── Uniformity
├── Explicit Semantics
├── Declarative With Static Guarantees
├── Named Before Symbolic
├── Minimal Core
├── Software Design Principles
├── Justified Solutions
├── Transparency
├── Principle of Parsimony

Language Consistency
├── Representation Symmetry
├── Deterministic Behavior
├── Stable Mental Model

Execution Model
├── Semantics Before Optimization
├── Explicit Optimization
├── Correctness Before Performance
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


## Simplicity

A language should be simple to learn, read, and reason about.

Every feature carries a cognitive cost. Simplicity is achieved not by limiting expressiveness, but by ensuring each construct earns its place.
Complexity belongs in the compiler, not the programmer.


## Explicitness

The meaning of code should be apparent from its surface form.

Behavior-changing operations — ownership transfer, mutability,
allocation — must be visible in the syntax. Code should not depend on hidden rules or implicit heuristics.

Explicitness is the goal; **Explicit Semantics** is the rule that enforces it.


## Consistency

Similar concepts should look similar. Different concepts should look different.

A consistent language reduces the learning surface area: once a pattern is learned in one context, it applies in all analogous contexts.

Consistency is the foundation of a **Stable Mental Model**.


## Uniformity

Equivalent concepts should be expressed in equivalent ways.

Once a user learns a language pattern, the same pattern should apply
consistently throughout the language.


## Explicit Semantics

Whenever an operation changes the meaning, lifetime, ownership, or
behavior of data, it should be expressed explicitly in the syntax.

The language should avoid hidden conversions and implicit semantic
changes.

## Declarative With Static Guarantees

The language provides declarative abstractions whose semantic guarantees
are verified at compile time — never left to the programmer's manual
discipline.

The language itself bears the burden of complexity. When a recurring
pattern can be captured as a declarative construct with a
statically-checkable contract, the language prefers this over an
imperative idiom that shifts correctness responsibility to the
programmer.

Modern language features — list comprehensions, context managers, Stream
API, Optional, pattern matching, records, coroutines — succeed because
they move incidental complexity from the programmer's manual discipline
to the language's automatic enforcement. Orthon follows this principle
systematically: what can be guaranteed statically should be guaranteed
statically.

Declarative abstractions are not merely syntactic sugar over imperative
code. They represent a transfer of responsibility: imperative code
requires the programmer to get every detail right every time; a
declarative abstraction with static guarantees transfers that
responsibility to the compiler.

## Named Before Symbolic

Every symbolic operator should have an equivalent named function.

Symbols improve brevity.

Named functions improve readability.

Both represent exactly the same semantics.


## Minimal Core

The language must have a **minimal, but sufficient, core** that does not
require change as new technologies, platforms, or optimisation techniques
emerge.

Language evolution occurs not through changes to the Core, but through
development of:

- **Implementation Strategy** — how semantics are realised
- **Standard Library** — higher-level abstractions built on primitives

Program semantics remain unchanged as implementations evolve freely.

### Responsibility Separation

```
User Program
       │
       ▼
Core Language — semantic primitives (what the program means)
       │
       ▼
Implementation Strategy — named set of Policies (how it is realised)
       │
       ▼
Standard Library — interface contracts (high-level abstractions)
       │
       ▼
Platform Implementation — concrete compiler, runtime, platform
```

Each level is responsible for its own domain.

- **Core** defines the meaning of the program — the semantic primitives.
- **Implementation Strategy** defines how each concern is realised
  through declarative Policies.
- **Standard Library** provides high-level abstractions and interface
  contracts built on Core primitives.
- **Implementation** leverages the capabilities of a specific platform.

### Consequence

The emergence of a new capability must not require changing the language.

For example, when GPU computing emerged:

```
sort(data)
```

does not change. The **Strategy** changes — an `Algorithm Policy` selects a
GPU-optimised sorting implementation. The same applies to NUMA, quantum
computing, new sorting algorithms, new garbage collectors, or a different
allocator: the **Strategy and Library** adapt. The **Core remains the same**.

Orthon Core should become a **stable interface between any program and
any implementation** — a semantic instruction set for the programmer.

### Core Change Criterion

**The Core changes only when new semantics cannot be expressed through
composition of existing Core primitives.**

In all other cases, the solution must live at the level of:

- Implementation Strategy (Policies)
- Standard Library (new abstractions on existing primitives)
- Platform Implementation (concrete realisation)

This rule protects the language from gradual specification bloat. If
enforced strictly, after many years the language specification remains
nearly unchanged, while implementations continue to evolve — similar to
RISC architectures: a small, stable set of primitives, with evolution
happening around them. In Orthon's case, these primitives are the
**Orthon Primitive Blocks** — the immutable foundation upon which
implementation strategies and the standard library are built.

The Core is not "assembly for the machine." It is a **Semantic ISA** —
a semantic instruction set for the programmer. The compiler translates
this semantics into concrete implementation.

## Software Design Principles

The same SOLID principles that guide good software engineering also guide the design of Orthon itself.

-   **Single Responsibility** — Core, Standard Library, and
    Implementation Strategy each have one responsibility.
-   **Open/Closed** — The Core Language is closed for modification;
    behavior is extended through the Standard Library (new contracts)
    and Implementation Strategies (new realisations).
-   **Liskov Substitution** — Different implementation strategies are interchangeable without changing program semantics.
-   **Interface Segregation** — Programs depend on a minimal, coherent language interface.
-   **Dependency Inversion** — High-level code depends on language abstractions, not runtime implementation details.

This architecture makes the language maintainable, evolvable, and implementation-independent.


## Justified Solutions

Every problem deserves its own tool.

A solution is justified by being the right match for the specific problem — not by familiarity, fashion, or convenience.


## Transparency

Every design decision must be traceable from its originating idea through
to the final decision.

When a new concept is introduced or an existing one is modified, the full
path must be documented:

- **Origin** — what idea, problem, or gap prompted the change
- **Considered alternatives** — what other approaches were evaluated
- **Rationale** — why the chosen solution was selected over alternatives
- **Gate review** — how the decision satisfies the Language Design Gate
- **Impact** — which existing concepts, documents, or decisions are affected

Transparency serves both as a process guard and an architecture health
metric:

**Gate guard.** If a proposal is similar to one already rejected by the
Language Design Gate, the record of that rejection must be consulted
before re-opening the discussion. Re-opening requires the proponent to
explicitly state what circumstances have changed and why the original
grounds for rejection no longer apply. A proposal that rehashes a
rejected idea without new context is dismissed without review.

**Architecture metric.** A concept that requires more than one
substantive semantic change after its initial Gate approval triggers a
mandatory architectural review. Frequent revision of a single concept
suggests poor factoring in the architecture itself, not merely an
incomplete design. Clarifications, documentation improvements, and
additive scope extensions do not count toward this threshold — only
changes that alter previously approved semantics.

Traceability without a decision journal is impossible. See
[`docs/how/gates/_language-design.md`](../how/gates/_language-design.md)
for the Gate checklist and
[`docs/how/architecture/FITNESS_FUNCTIONS.md`](../how/architecture/FITNESS_FUNCTIONS.md)
for the architecture fitness functions catalogue.

A design decision without a traceable path is indistinguishable from
arbitrary choice.


## Principle of Parsimony

When multiple valid solutions exist for a given problem, prefer the one
that introduces the fewest additional assumptions or entities.

Parsimony serves as the tiebreaker when the Language Design Gate and
Decision Validation gates produce ambiguous or evenly balanced results.
A solution that requires fewer new concepts, fewer special cases, or
fewer implicit rules is preferred over one that requires more — all else
being equal.

This principle also applies when evaluating controversial Engineering
Decision Records (EDRs): among competing alternatives with comparable
merit, the most parsimonious explanation and solution should be adopted.

> *Entia non sunt multiplicanda praeter necessitatem.*
> (Entities must not be multiplied beyond necessity.)

---

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
\nThe default execution model favors predictability over performance.

------------------------------------------------------------------------

## Correctness Before Performance

Correctness always takes precedence over performance.

Performance improvements are valuable only when they preserve clarity, determinism, and the defined semantics of the language.

---

# Documentation Principles

See [DOCUMENTATION_PRINCIPLES.md](../how/DOCUMENTATION_PRINCIPLES.md).
