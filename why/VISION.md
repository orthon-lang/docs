# Vision

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

## Execution Program

Most languages treat **source code** as the primary artifact and the
execution environment as external infrastructure — `venv`, Docker,
Conda, Nix, or a virtual machine bolted on after the fact.

Orthon inverts this. A program becomes **fully defined** only after it
is enriched with its required execution context.

The central abstraction is the **Execution Program** — a reproducible,
content-addressed, canonical representation of everything needed to
execute the program.

### Pipeline

```
Program
    │
    ▼
Execution Descriptor (manifest of requirements)
    │
    ▼
Program Enricher
    │
    ▼
Execution Program
    │
    ▼
Execution Engine
    │
    ├── Interpreter
    ├── REPL
    ├── Notebook Kernel
    ├── Test Runner
    ├── Debugger
    ├── JIT / AOT Compiler
    ├── OCI Builder
    ├── MicroVM Builder
    └── WASM Builder
```

An interpreter and an OCI builder are the same kind of thing: both
consume an Execution Program. They differ only in *how* they execute,
not in *what* they execute.

The language knows nothing about Docker, containers, or virtual
machines. It knows only about an **Execution Descriptor** — an explicit
declaration of language version, runtime, dependencies, strategy,
platform, permissions, and resources. Concrete execution technologies
are **Execution Engines**, selected through the Implementation Strategy
or the developer's tooling choice.

This approach extends the Semantic ISA into the deployment domain:
the same Program, enriched by the same Descriptor, produces the same
Execution Program anywhere — a developer workstation, a CI pipeline,
a notebook kernel, or a production orchestrator. Reproducibility,
portability, and isolation are no longer DevOps concerns. They are
properties of the language architecture itself.

## Goal

Keep the language stable for decades while allowing implementations
to continuously improve without breaking existing programs.
