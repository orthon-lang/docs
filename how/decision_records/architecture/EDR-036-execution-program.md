# EDR-036: Execution Program — Decoupling Semantics from Execution Strategy

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon's core innovation — the Execution Program model — decouples *what a program means* from *how it executes*. The research document [`EXECUTION_PROGRAM.md`](../../concepts/research/essential/EXECUTION_PROGRAM.md) establishes the full vision: a program becomes fully defined only after it is enriched with its execution context via a Program Enricher, and Execution Engines (interpreter, AOT, JIT, WASM, container builder) consume the same canonical representation.

**Per D-04:** The Execution Program is classified as **Policy** — it introduces a new Policy type (Execution Model Policy) that governs how programs are executed. It is not a Language construct because it does not add new semantics expressible via primitives; it is an implementation choice about HOW a program is run, not WHAT the program means.

This is a *new* Policy type, added to the existing Strategy system alongside Allocation Policy, Algorithm Policy, etc.

---

### Decision

The Execution Program model introduces a new **Execution Model Policy** type to the Strategy system. The Policy defines how a fully-defined Execution Program is materialised and consumed.

The Execution Model Policy defines five values:

| Value | Description | Use Case |
|-------|-------------|----------|
| `Interpreted` | AST walking or bytecode VM — no compilation stage | Development, REPL, prototyping |
| `AOT` | Ahead-of-time compilation to native code | Default — production deployments |
| `JIT` | Just-in-time compilation, profile-guided optimisation | Long-running applications, server workloads |
| `WASM` | Compilation to WebAssembly | Browser, edge, sandboxed environments |
| `Container` | Execution Program materialised as OCI container image (Docker-compatible) | Cloud-native, container orchestration |

**Key architectural insight:** The Execution Program is the *central artifact*. All Execution Engines consume the same interface. The Program Enricher combines Program + Execution Descriptor into the canonical Execution Program. This inverts the traditional pipeline where deployment configuration is fragmented across incompatible formats.

**New Policy Type:** Execution Model Policy is a peer of Allocation Policy, Algorithm Policy, etc. in the Strategy system. Each Implementation Strategy selects one execution mode.

---

### Why Policy, Not Language

1. **No new language semantics.** The Execution Program model changes how programs are packaged and executed, not what programs mean. Core language semantics (equality, ownership, mutation, etc.) are unaffected by whether the program is interpreted, AOT-compiled, or containerised.
2. **Implementation Strategy concern.** Execution is a *how* decision. The same source code, with the same Execution Descriptor, produces the same observable behaviour regardless of execution engine.
3. **Program Enricher is infrastructure.** The Enricher and Execution Engines are tooling and runtime concerns, not language concerns. The Language specification defines only the *shape* of the Execution Descriptor.

---

### Consequences

- **Positive:**
  - Full decoupling of program semantics from execution strategy — the same source can be interpreted, AOT-compiled, or deployed as a container without modification.
  - Execution Engines are pluggable — adding a new engine (MicroVM, remote executor) does not require language changes.
  - Execution Descriptor is a first-class artifact, eliminating the fragmented DevOps toolchain problem.
  - LLM-friendly — a single canonical artifact format replaces multiple incompatible configuration languages (Dockerfile, Kubernetes manifests, CI config).
- **Negative:**
  - New Policy type adds complexity to the Strategy system.
  - Execution Descriptor format and Program Enricher specification are deferred (tooling, not language).
  - Container execution model requires additional infrastructure (OCI builder tooling).
  - Interoperation with existing ecosystem (Docker, Kubernetes) requires adapter tooling, not just the Execution Program.

---

### Compliance

DEFAULT_STRATEGY.md must specify an Execution Model Policy value (default: `AOT`). Every Implementation Strategy must document its Execution Model Policy value. The Core Language specification must not assume any specific execution mode.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Execution model as Language feature (compiler intrinsics for runtime selection) | Violates Minimal Core — execution is infrastructure, not language. Program should not know whether it is interpreted or compiled. |
| No execution program — traditional toolchain fragmentation | Misses Orthon's core innovation. Maintaining separate tooling for interpreter, compiler, and container builder duplicates effort and introduces configuration drift. |
| Execution model as separate subsystem outside Strategy | Fragments the Strategy system — execution model IS a Policy choice and belongs in the unified Strategy profile. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Skipped | Execution model Policy — indirect user value through simplified deployment. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Decoupling semantics from execution is internally consistent with Orthon's Intent Over Implementation principle. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | Five values cover the full execution spectrum. Each is a well-understood technology. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | New Policy type integrates cleanly into the existing Strategy system as a peer of Allocation, Algorithm, etc. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Execution model is fully independent of any specific implementation. All values are realisable by different runtime implementations. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Adding new execution engines (MicroVM, FPGA, etc.) adds a new Policy value without changing the system. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Skipped | Execution model is a build-time/deployment concern — LLM never writes execution strategy code. |

**Gates not applied:** USER_VALUE_GATE (Policy concept); LLM_GENERABILITY_GATE (build-time concern, not LLM code generation target).
