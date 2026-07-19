# Execution Image

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).

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

### Why Orthon should address this

Orthon already establishes a **Semantic ISA** — a stable contract between
the program and its implementation. The natural extension of this
contract is the *execution environment* itself: what runtime, what
standard library, what dependencies, what platform does the program
require?

If the language knows its own environment contract, then **packaging
and deployment become part of the compilation process** rather than
separate post-hoc steps. The boundary between "building" and "shipping"
dissolves.

## Principles

Which principles must not be violated?

### Minimal Core

The Core Language defines only the concept of an *execution environment
contract*. It does not define Docker, OCI, containers, virtual machines,
or any concrete deployment technology. Those belong to the
**Infrastructure Layer** and **Implementation Strategy**.

### Intent Over Implementation

The programmer declares the target environment characteristics (what
runtime, what dependencies, what permissions). The compiler and toolchain
decide how to materialise that environment.

### Deterministic Behavior

The same source code and environment description must produce the same
**Execution Image**. The image is a reproducible, content-addressed
description of the complete execution environment — not a mutable
build artifact.

### Explicitness

The environment contract is explicitly declared — either in the project
configuration or as part of the Strategy selection. No implicit
assumptions about the target platform.

### SOLID — Single Responsibility

- **Language** defines the semantic contract of the program.
- **Compiler** transforms source into intermediate representation.
- **Environment Resolver** determines the composition of the execution
  environment.
- **Artifact Builder** assembles the Execution Image.
- **Execution Provider** materialises the Image into a concrete format.

### SOLID — Dependency Inversion

High-level code depends on the abstract **Environment** interface.
Concrete providers (OCI, MicroVM, native) implement that interface.
The language never imports deployment-platform specifics.

### SOLID — Open/Closed

New execution providers can be added without changing the language,
compiler, or existing providers. Adding a MicroVM provider does not
require modifying the OCI provider.

### Minimal Core

The language core defines only *what an Execution Image is* and *how to
describe an environment*. It does not define how images are built,
stored, or distributed. Those are tooling concerns.

## Policy Footprint

Which Policy Types does this concept involve?

| Policy Type | Role in the concept |
|---|---|
| *New: Environment Policy* | Declares the execution environment contract (runtime, stdlib version, dependencies, platform constraints) |
| *New: Distribution Policy* | Governs how the Execution Image is packaged and distributed (format, compression, signing) |

*These Policy Types are provisional. They will be formally proposed
during Milestone 1 (Language Inventory) and validated through the
Language Design Gate in Milestone 2 (Concept Design Review).*

## Model (What)

### Core abstraction: Execution Image

An Orthon program is not compiled into a binary. It is compiled into a
**reproducible Execution Image** — a complete, content-addressed
description of everything needed to run the program.

An Execution Image includes:

```
Execution Image
├── Program            ← compiled code
├── Runtime            ← VM, interpreter, or native runtime
├── Standard Library   ← selected stdlib version
├── Dependencies       ← resolved, version-pinned
├── Implementation Strategy  ← active Policies
├── Metadata           ← name, version, author, description
├── Permissions        ← required capabilities (network, filesystem, etc.)
└── Resources          ← declared constraints (memory, CPU, storage)
```

### The unified pipeline

Instead of the fragmented pipeline, Orthon defines a single flow:

```
Source
    ↓
Resolve Environment  ← determine the environment contract
    ↓
Compile              ← translate source to intermediate form
    ↓
Bundle Runtime       ← assemble runtime + stdlib + dependencies
    ↓
Execution Image      ← complete, reproducible, content-addressed
```

### Environment abstraction

The **Environment** is an abstraction over the execution context.

``` text
Environment
    resolve()       ← determine the composition (what is needed)
    materialize()   ← make it available (download, build, link)
    run()           ← execute within this environment
```

This is an interface contract. The language knows *that* an environment
exists, not *how* it is realised.

### Image Providers

An **Image Provider** materialises an Execution Image into a concrete
output format.

```
Execution Image
        │
        ├── Native executable    (ELF / PE / Mach-O)
        ├── OCI Image            (container)
        ├── MicroVM Image        (Firecracker, Cloud Hypervisor)
        ├── WASM Module          (WebAssembly)
        ├── Shared Library       (dynamic link)
        └── Remote Bundle        (distributed execution)
```

The programmer does not say "build a Docker image." The programmer says
"target this environment." The Image Provider decides the concrete
format.

This mirrors the existing resolution chain:

```
Language (what)
    ↓
Environment (contract)
    ↓
Image Provider (how)
```

### Building from references

Because the Execution Image is content-addressed, much of the build is
simply assembling references:

```
Global Cache
├── Runtime vX.Y.Z       ← content hash
├── Stdlib vA.B.C        ← content hash
├── Dependency Alice      ← content hash
├── Dependency Bob        ← content hash
├── Dependency Charlie    ← content hash
└── ...

Project
└── Execution Image
    ├── Runtime ─────────→ vX.Y.Z hash
    ├── Stdlib ──────────→ vA.B.C hash
    ├── Alice ───────────→ hash
    ├── Bob ─────────────→ hash
    ├── Charlie ─────────→ hash
    └── Program ─────────→ new
```

If the environment hash has not changed, the Image is assembled from
cache. No re-resolution, no re-downloading, no rebuilding of unchanged
layers.

### Build acceleration via environment hash

The environment description (runtime + stdlib + dependencies + strategy)
has a deterministic hash. If the hash matches a cached environment, the
entire resolution and bundling step is skipped:

```
Source
    ↓
Compile
    ↓
Hash(Environment) = H
    │
    ├── H in cache → assemble from cache (instant)
    │
    └── H not in cache → resolve, materialise, bundle, cache
```

This is conceptually similar to Nix and BuildKit, but embedded in the
language model rather than bolted on as external tooling.

## Default Strategy

By default, the **Environment Resolver** targets the current host
platform and produces a **Native Executable** (ELF on Linux, Mach-O on
macOS, PE on Windows). The runtime and standard library are statically
linked, and dependencies are resolved from the project environment
description.

The default **Image Provider** writes the native executable directly to
the output path — indistinguishable from a traditional compiler output.

## Alternative Strategies

| Strategy | Image Provider | Use Case |
|---|---|---|
| `Default` | Native executable | Local development, traditional deployment |
| `OCI` | OCI Image provider | Container-based deployment |
| `MicroVM` | Firecracker / Cloud Hypervisor | Strong isolation, multi-tenant |
| `WASM` | WebAssembly Module | Browser, plugin, edge runtime |
| `Library` | Shared library (.so / .dylib / .dll) | FFI, embedding in other programs |
| `Remote` | Remote bundle | Distributed or serverless execution |

Each strategy corresponds to a different **Distribution Policy** value,
selected through the active Implementation Strategy profile.

## Responsibility separation

The unified pipeline introduces a natural separation of concerns:

```
Language
    defines the semantics of the program.

Compiler
    transforms source code into an intermediate representation.

Environment Resolver
    determines the composition of the execution environment
    (runtime, stdlib, dependencies, strategy, permissions, resources).

Artifact Builder
    assembles the Execution Image from resolved components.

Execution Provider (Image Provider)
    materialises the Execution Image into a concrete distributable
    format (OCI, MicroVM, native executable, WASM, etc.).
```

Each component has a single responsibility and can be replaced
independently — in full alignment with SOLID principles.

## Relationship to existing architecture

```
User Code
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
Concrete Implementation
    │
    ├── Compiler
    ├── Runtime
    ├── Platform
    └── Image Providers ← EXTENDED
```

The **Environment Policy** and **Distribution Policy** sit alongside
existing Policy Types in the Implementation Strategy layer. The
**Image Provider** extends the Concrete Implementation layer, providing
the final materialisation of the Execution Image.

## Open Questions

- Should the Environment description be part of the project manifest
  (like `Cargo.toml`), or should it be expressible inline in source code?
- What is the canonical format for an Environment description?
- How does signing and verification work for Execution Images?
- How does the permission model interact with the platform's security
  model (seccomp, capabilities, MAC)?
- Can Execution Images be composed (e.g., a base image + application
  layer)?
- What is the upgrade story for Execution Images — can they be patched
  without rebuild?
- How does the cache invalidation work for the Global Cache?

## Decision History

*This section will be populated as decisions are made through the
Language Design Gate process.*

---

### Affected Documents

- [ ] `CORE_CONCEPTS.md`
- [x] `DATA_MODEL.md`
- [x] `DESIGN_PRINCIPLES.md`
- [x] `VISION.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] `GLOSSARY.md`
- [ ] `ARCHITECTURE.md`
