# Execution Program

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

### The fragmented pipeline problem

Modern languages treat **compilation**, **packaging**, and **deployment**
as separate stages, each with its own tool and its own configuration.

```
Source
    ↓  [compiler]
Binary / bytecode
    ↓  [package manager]
Dependency bundle
    ↓  [Docker]
Dockerfile
    ↓  [BuildKit]
OCI layer
    ↓  [registry push]
Published image
    ↓  [Kubernetes]
Running container
```

The same information — versions, dependencies, runtime requirements,
platform constraints — is described multiple times across incompatible
formats:

- `Cargo.toml` / `package.json` / `requirements.txt`
- `Dockerfile`
- `docker-compose.yml`
- Kubernetes manifests
- CI pipeline config

This fragmentation creates a gap between *what the program needs* and
*what the deployment environment provides*. Discrepancies between these
descriptions are a leading source of production failures.

### A deeper problem: wrong central abstraction

The root cause is not just tool fragmentation. It is that **source code**
is treated as the central artifact, while the execution environment is
treated as external infrastructure.

In most languages:

```
Program
    +
Environment (external: venv, Docker, Conda, Nix, VM...)
        ↓
Execution
```

Every tool that needs to run the program (interpreter, REPL, notebook,
test runner, debugger, CI/CD, container builder) must reconstruct the
environment independently. Each one does it differently.

### Why Orthon should address this

Orthon already establishes a **Semantic ISA** — a stable contract between
the program and its implementation. The natural extension is to make the
*execution environment* part of the program contract, not external
infrastructure.

If the language owns the concept of a fully-defined program, then every
consumer — interpreter, REPL, test runner, debugger, CI/CD, OCI builder,
MicroVM builder — works from the same canonical representation. The
boundary between "building", "packaging", "deploying", and "running"
dissolves.

## Principles

Which principles must not be violated?

### Minimal Core

The Core Language defines only the concept of an *execution environment
contract* (Execution Descriptor) and the *canonical representation of a
fully-defined program* (Execution Program). It does not define
interpreters, compilers, Docker, OCI, containers, virtual machines, or
any concrete execution technology. Those belong to the **Infrastructure
Layer** and **Execution Engines**.

### Intent Over Implementation

The programmer declares *what* the program needs (language version,
dependencies, permissions, resources). The **Program Enricher** decides
*how* to fulfil that contract. Execution Engines decide *how* to run.

### Deterministic Behavior

The same Program and Execution Descriptor must produce the same
**Execution Program** — a reproducible, content-addressed canonical
representation. Executing the same Execution Program on compatible
platforms must produce identical observable behavior.

### Explicitness

The Execution Descriptor is an explicit, first-class artifact. It is
not inferred from filesystem layout, installed tools, or environment
variables. The boundary between Program (source) and its context is
always visible.

### SOLID — Single Responsibility

- **Program** — the source code.
- **Execution Descriptor** — declarative manifest of requirements.
- **Program Enricher** — combines Program + Descriptor into Execution Program.
- **Execution Engine** — consumes an Execution Program (runs, compiles,
  materialises, debugs, tests...).

### SOLID — Dependency Inversion

High-level code depends on the abstract **Execution Program** interface.
All Execution Engines (interpreter, AOT, OCI builder, debugger) consume
the same interface. The language never imports engine specifics.

### SOLID — Open/Closed

New Execution Engines can be added without changing the language,
compiler, or existing engines. Adding a MicroVM builder does not require
modifying the interpreter.

## Policy Footprint

Which Policy Types does this concept involve?

| Policy Type | Role in the concept |
|---|---|
| *New: Environment Policy* | Declares the execution environment contract (runtime, stdlib version, dependencies, platform constraints) |
| *New: Distribution Policy* | Governs how the Execution Program is materialised (format, compression, signing) |

*These Policy Types are provisional. They will be formally proposed
during Milestone 1 (Language Inventory) and validated through the
Language Design Gate in Milestone 2 (Concept Design Review).*

## Model (What)

### Shift in central abstraction

Most languages treat **source code** as the primary artifact and the
execution environment as external infrastructure. Orthon inverts this:

> A program becomes fully defined only after it is enriched with its
> required execution context.

The central artifact is not the source code. It is the **Execution
Program** — a complete, reproducible, canonical representation of
everything needed to run.

### Key entities

#### Program

The source code and its structure. Contains no information about the
execution environment.

```
Program
├── Source files
├── Module structure
└── Type definitions
```

#### Execution Descriptor

A declarative description of *what* the program requires — not *how*
those requirements are fulfilled. It is the program's manifest.

```
Execution Descriptor
├── Language version
├── Runtime requirements
├── Standard library
├── Dependencies
├── Implementation Strategy
├── Target platform
├── Permissions
└── Resources
```

The Execution Descriptor is a first-class artifact. It is not inferred
from the environment — it is explicitly declared and versioned alongside
the Program.

#### Program Enricher

The component that combines a Program with its Execution Descriptor into
a fully-defined Execution Program.

```
Program
    +
Execution Descriptor
    │
    ▼
Program Enricher
    │
    ├── Dependency Resolver
    ├── Runtime Resolver
    ├── Strategy Resolver
    ├── Platform Resolver
    ├── Permission Resolver
    └── Resource Resolver
    │
    ▼
Execution Program
```

Internally, the Enricher coordinates multiple resolvers. Externally, it
is a single step — the boundary between "incomplete" and "fully-defined"
program.

#### Execution Program

The central abstraction. A fully-defined, reproducible, content-addressed
representation of everything needed to execute the program.

```
Execution Program
├── Program                  ← source code, compiled or not
├── Runtime                  ← VM, native runtime, or interpreter
├── Standard Library         ← selected stdlib version
├── Dependencies             ← resolved, version-pinned
├── Implementation Strategy  ← active Policies
├── Configuration            ← explicit settings
├── Permissions              ← required capabilities (network, FS, etc.)
├── Resources                ← declared constraints (memory, CPU, storage)
└── Metadata                 ← name, version, description
```

The Execution Program is the **canonical representation**. It is not a
binary, not a Docker image, not a virtual machine — it is an
infrastructure-independent model of an executable program.

#### Execution Engine

Any consumer that takes an Execution Program and does something with it.

```
Execution Program
        │
        ├── Interpreter
        ├── REPL
        ├── Notebook Kernel
        ├── Test Runner
        ├── Debugger
        ├── JIT Compiler
        ├── AOT Compiler
        ├── OCI Builder
        ├── MicroVM Builder
        ├── WASM Builder
        └── Remote Executor
```

The key insight: an interpreter and an OCI builder are the same kind of
thing. Both consume an Execution Program. Both produce executable
behavior. They differ only in *how* they execute — not in *what* they
execute.

### The unified pipeline

```
Program
    │
    ▼
Execution Descriptor
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
    ├── Notebook
    ├── Test Runner
    ├── Debugger
    ├── Native (AOT)
    ├── OCI
    ├── MicroVM
    └── WASM
```

### What this eliminates

Because every consumer works from the same Execution Program:

- **No `venv` as a language concept** — the environment is encoded in
  the Execution Descriptor, not in filesystem convention.
- **No repeated dependency descriptions** — one descriptor serves
  interpreter, CI, Docker, and debugger alike.
- **No separate CI environment setup** — CI consumes the same Execution
  Program as local development.
- **No IDE project model drift** — the IDE uses the same Enricher as
  the compiler.
- **No separate Docker build** — OCI Builder is an Execution Engine,
  same as the interpreter.
- **No separate MicroVM build** — MicroVM Builder is another Engine.

### Materialisation

Some Execution Engines produce a distributable artifact. This is called
**materialisation** — converting the canonical Execution Program into a
concrete, self-contained format.

```
Execution Program
        │
        ├── Native Executable   (ELF / PE / Mach-O)
        ├── OCI Image           (container)
        ├── MicroVM Image       (Firecracker, Cloud Hypervisor)
        ├── WASM Module         (WebAssembly)
        ├── Remote Session      (distributed execution)
        └── Embedded Package    (library)
```

Materialisation is not a separate build phase. It is what a specific
Execution Engine does when its *modus operandi* is to produce a
standalone artifact.

### Building from references

Because the Execution Program is content-addressed, most of its
composition is simply assembling references:

```
Global Cache
├── Runtime vX.Y.Z       ← content hash
├── Stdlib vA.B.C        ← content hash
├── Dependency Alice      ← content hash
├── Dependency Bob        ← content hash
├── Dependency Charlie    ← content hash
└── ...

Project
└── Execution Program
    ├── Runtime ─────────→ vX.Y.Z hash
    ├── Stdlib ──────────→ vA.B.C hash
    ├── Alice ───────────→ hash
    ├── Bob ─────────────→ hash
    ├── Charlie ─────────→ hash
    └── Program ─────────→ new
```

If the Execution Descriptor hash has not changed, the Program Enricher
produces the Execution Program from cache. No re-resolution, no
re-downloading, no rebuilding of unchanged layers.

### Build acceleration via descriptor hash

The Execution Descriptor (runtime + stdlib + dependencies + strategy +
platform + permissions + resources) has a deterministic hash.

```
Program
    │
    ▼
Hash(Execution Descriptor) = H
    │
    ├── H in cache → cached Execution Program (instant)
    │
    └── H not in cache → resolve, enrich, cache
```

This is conceptually similar to Nix and BuildKit, but embedded in the
language model rather than bolted on as external tooling.

## Default Strategy

By default, the **Program Enricher** targets the current host platform.
The resulting **Execution Program** is consumed by the default
**Execution Engine** — an **Interpreter** for development, or an
**AOT Compiler** producing a **Native Executable** (ELF on Linux,
Mach-O on macOS, PE on Windows) for production.

The runtime and standard library are resolved from the local cache or
downloaded on first use — indistinguishable from a traditional language
toolchain in the simple case, but fundamentally different in architecture.

## Alternative Strategies

| Strategy | Execution Engine | Use Case |
|---|---|---|
| `Default` | Interpreter / AOT | Local dev, traditional deployment |
| `OCI` | OCI Builder | Container-based deployment |
| `MicroVM` | MicroVM Builder | Strong isolation, multi-tenant |
| `WASM` | WASM Module | Browser, plugin, edge runtime |
| `Library` | Shared library | FFI, embedding in other programs |
| `Remote` | Remote Executor | Distributed or serverless execution |
| `Notebook` | Notebook Kernel | Interactive development |
| `Debug` | Debugger | Step-through debugging |

Each engine corresponds to a different **Distribution Policy** value,
selected through the active Implementation Strategy profile or the
developer's tooling choice.

## Responsibility separation

The unified pipeline introduces clear responsibilities:

```
Language
    defines the semantics of the program.

Execution Descriptor
    is the explicit, versioned manifest of program requirements.

Program Enricher
    resolves the Descriptor and produces the canonical Execution Program.

Execution Engine
    consumes the Execution Program — runs, debugs, tests, compiles,
    materialises, or deploys it.
```

## Relationship to existing architecture

```
User Code (Program + Execution Descriptor)
    │
    ▼
Language Layer (what the program means)
    │
    ├── Core Language — semantic primitives
    ├── Standard Library — interface contracts
    └── Syntax — human interface
            │
            ▼
Implementation Strategy (named set of Policies)
    │
    ├── Environment Policy ← NEW
    ├── Distribution Policy ← NEW
    ├── Allocation Policy
    ├── Algorithm Policy
    ├── Evaluation Policy
    ├── Lifetime Policy
    ├── Concurrency Policy
    └── ...
            │
            ▼
Program Enricher ← NEW
    │
    ▼
Execution Program ← NEW
    │
    ▼
Concrete Implementation
    │
    ├── Compiler
    ├── Runtime
    ├── Platform
    └── Execution Engines ← EXTENDED
```

The **Environment Policy** and **Distribution Policy** sit alongside
existing Policy Types. The **Program Enricher** is a new architectural
component between Strategy and Implementation. **Execution Engines**
extend the Concrete Implementation layer, broadening it from a single
compiler to a family of equal consumers.

## Open Questions

- Should the Execution Descriptor be a separate file (like `Cargo.toml`),
  embeddable in source, or both?
- What is the canonical serialisation format for an Execution Program?
- How does caching work across distributed teams — shared cache, remote
  cache, registry?
- How does signing and verification work for Execution Programs?
- Can Execution Programs be composed (e.g., a base + application layer)?
- What is the upgrade story — can an Execution Program be patched
  without full re-enrichment?

## Decision History

*This section will be populated as decisions are made through the
Language Design Gate process.*

---

### Affected Documents

- [ ] `FOUNDATIONAL_ABSTRACTIONS.md`
- [x] `DATA_MODEL.md`
- [x] `DESIGN_PRINCIPLES.md`
- [x] `VISION.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `GLOSSARY.md`
- [ ] `ARCHITECTURE.md`
