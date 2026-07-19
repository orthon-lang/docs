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

```
sort(data)              # LLM says WHAT — sort the data
# Not: which algorithm, which memory, which comparison strategy
```

### Explicit semantics reduce hallucination

Orthon's Explicit Semantics principle means every behavior-changing
operation is visible in the syntax. An LLM generating Orthon code
cannot accidentally introduce implicit behavior — there is none.
What the LLM writes is exactly what the program does.

### Execution Program as LLM-native artifact

The Execution Program model dissolves the gap between "write code"
and "run code." An LLM can produce not just source code, but a
fully-defined Execution Program — a self-contained, reproducible
artifact that includes runtime, dependencies, permissions, and
target environment. The LLM does not need to know about Docker,
containers, or deployment pipelines. The language handles the
boundary between code and environment.

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

### DevOps Transformation

The Execution Program model reshapes how software is delivered, not
just how it is written.

**CI/CD becomes simpler.** A single pipeline produces one artifact type
— the Execution Program — regardless of target. No more matrix builds,
platform-specific branches, or conditional packaging logic. CI produces
one thing; deployment chooses the engine.

**Development-to-production parity is guaranteed by construction.**
Because the same artifact runs in every environment, the class of bugs
caused by "it worked on my machine" disappears. The artifact you
debugged in the interpreter is the same artifact the OCI builder
packages for production.

**Deployment becomes a configuration choice, not a rebuild.**
Switching from interpreted development to AOT-compiled production,
from server deployment to edge deployment, or from standalone binary
to OCI container — all become deployment-time engine selections on
the same artifact. No recompilation, no new CI run, no risk of
divergence.

**Observability and resource governance are part of the descriptor.**
The Execution Descriptor declares not just dependencies but also
permissions, resource limits, and observability contracts. The
Execution Engine enforces them. The platform does not need to guess
what the program needs — the program declares it explicitly.

**Microservices and monoliths are packaging choices, not architectural
commitments.** The same Execution Program can be deployed as a
standalone binary, a container, a serverless function, or a WASM
module — without code changes. The architecture is in the code, not
in the deployment topology.

In short: Docker containerized the *environment*. Orthon containerizes
the *program itself* — making every execution mode a first-class
citizen and every deployment path a configuration option.

## Goal

Keep the language stable for decades while allowing implementations
to continuously improve without breaking existing programs.
