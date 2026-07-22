# Native Compilation Model

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language deliver executable code to the target platform — through a virtual machine (bytecode, JIT) or through ahead-of-time native compilation?

Two fundamentally different execution models exist:

1. **Virtual machine / runtime-based (Java JVM, C# CLR, Python interpreter, JavaScript engine)** — source code is compiled to an intermediate representation (bytecode) that runs on a virtual machine or is interpreted directly. The VM provides cross-platform portability, runtime optimisation (JIT), and a managed runtime (GC, class loading, security manager). The cost: startup time (JVM warmup), distribution size (JRE required), and opaque deployment.

2. **Ahead-of-time native compilation (Go, Rust, Zig, C)** — source code is compiled directly to a native executable for the target platform. Dependencies are statically linked into a single binary. No runtime or VM is required on the target machine. The cost: cross-compilation must target each platform separately, and runtime optimisation is limited to what the compiler can do at build time.

The core problem: **what execution model does Orthon use**, and does it support both modes?

Go's approach is instructive: Go compiles to a single, statically-linked native binary with no external dependencies. Startup is near-instantaneous. Deployment is a single file. Cross-compilation is built into the toolchain (`GOOS=linux GOARCH=amd64 go build`). The trade-off: no JIT warmup, so peak performance may lag behind a well-tuned JVM, but consistent performance from the first instruction.

## Principles

1. **Instant startup** — Programs start with no warmup period. The first instruction executes at full speed.

2. **Single-binary deployment** — Distribution is a single executable file. No runtime, no JRE, no DLL hell.

3. **Static linking by default** — All dependencies are linked into the binary. Dynamic linking is opt-in for specific platform interop scenarios.

4. **Cross-compilation as a first-class feature** — Building for a different target is a matter of setting environment variables, not reconfiguring the build system.

5. **No hidden runtime** — The language runtime (GC, scheduler, etc.) is embedded in the binary, not installed separately.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Compilation Model Policy | Selects between AOT-native, JIT/VM-based, or hybrid (both) |
| Runtime Embedding Policy | Governs whether the runtime is statically linked into the binary or requires a separate runtime |
| Linking Policy | Determines static vs. dynamic linking defaults and interop strategy |
| Cross-Compilation Policy | Defines how cross-compilation targets are specified and validated |
| Startup Policy | Governs startup guarantees — instant, fast, or VM-warmup-required |

## Model (What)

The compilation model defines how source code becomes an executable:

1. **AOT compilation** — Source → IR → Native code (machine instructions) → Static binary
2. **VM compilation** — Source → Bytecode → JIT-compiled at runtime → Native code
3. **Hybrid** — AOT compilation by default with optional JIT for hot paths; or VM with AOT precompilation (like Android's ART or .NET Native AOT)

The native compilation model produces:
- A single statically-linked executable
- Embedded runtime (GC, scheduler, allocator)
- No external dependencies at runtime
- Instant startup with no warmup

## Default Strategy

The default strategy is **AOT native compilation** to a statically-linked binary. Cross-compilation is built into the toolchain. The runtime is embedded in every binary.

## Alternative Strategies

1. **JIT/VM-based** — Compile to bytecode, run on a VM with JIT. Provides platform portability and runtime profiling. Appropriate when peak throughput after warmup matters more than startup latency.

2. **Hybrid AOT+JIT** — AOT-compile the core, JIT-compile hot paths at runtime. Combines instant startup with adaptive optimisation. Higher implementation complexity.

3. **Interpreted mode** — Support an interpreter for scripts and REPL while the compiler produces native binaries for production deployment. Orthon's Execution Program model already separates semantics from execution strategy, making this tractable.

## Open Questions

1. Does Orthon require all implementations to use AOT native compilation, or is a VM-based implementation allowed?
2. How does the Execution Program model interact with the compilation model — can the same program be both interpreted and compiled?
3. Should the language support a REPL or script mode that runs without compilation (interpreted or JIT)?
4. How does native compilation interact with Orthon's LLM-native tooling — can the Schema Provider be embedded in the binary?

## Cross-References

- See `EXECUTION_PROGRAM.md` for how execution strategy is decoupled from program semantics
- See `ARCHITECTURE.md` for compilation pipeline and IR
- See [`../../DESIGN_PRINCIPLES.md`](../../DESIGN_PRINCIPLES.md) for execution model principles
- See [`../../strategies/DEFAULT_STRATEGY.md`](../../strategies/DEFAULT_STRATEGY.md) for the default compilation strategy
