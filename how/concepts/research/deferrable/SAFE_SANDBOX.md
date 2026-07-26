# Safe Sandbox for Untrusted Code

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Note:** This concept is primarily a **toolchain and runtime** concern,
> not a Core Language feature. The Core Language may define the *interface*
> for sandboxed execution, but the implementation belongs to the runtime
> and toolchain layers.

## Issue (Why)

How does a language execute untrusted or generated code safely — without risking the host process's state, data, or resources?

This is a critical capability for any language with LLM-native tooling:

- **LLM-generated code** — An LLM generates code snippets, suggests edits, or completes partial programs. Before integrating this code into the main program, the runtime should validate that it is safe to execute.
- **Plugin and extension systems** — User-provided scripts (macros, automation, custom rules) should run in a restricted environment.
- **Automated testing of generated code** — Test harnesses may execute generated code in a sandbox, observe its behaviour, and roll back state on failure or violation.

Traditional approaches:

- **Process-level isolation** (subprocess in Python, `child_process` in Node) — Simple but heavy. Spawning a new OS process for every untrusted fragment has high overhead. Data must be serialised and deserialised across process boundaries.
- **Language-level interpretation** (Python `sandbox` modules, Lua sandboxing) — In-process interpreters with restricted APIs. Lighter than process isolation but can be bypassed through the host language's reflection or metaprogramming.
- **WebAssembly sandbox** — Near-native performance with strong isolation guarantees. Well-suited for plugin systems but requires a WASM runtime and compilation target.
- **Virtual machine isolation** (JVMs with security managers, CLR with CAS) — Granular permission systems. Historically proven to have bypass vulnerabilities.
- **Capability-based security** (Pony, WebAssembly Interface Types) — No ambient authority; all capabilities are passed explicitly. The most principled approach but requires language-level design decisions.

The core problem: **executing generated code must preserve the host program's safety invariants — memory safety, data integrity, resource limits, and termination guarantees — without imposing prohibitive overhead**.

## Principles

1. **Sandboxing is opt-in, not implicit** — No code runs in a sandbox unless explicitly marked. Generated code runs in the same environment as hand-written code by default; the `sandbox` marker activates isolation.

2. **Resource limits are part of the sandbox declaration** — CPU time, memory allocation, file system access, and network access constraints are declared when the sandbox is created. Unlimited sandboxes are not allowed.

3. **State rollback on failure** — If sandboxed code fails a resource check or throws an unhandled error, the sandbox's state is rolled back to a checkpoint. Side effects that escaped before the checkpoint are the programmer's responsibility to manage.

4. **No ambient authority** — Sandboxed code has no access to host resources (filesystem, network, environment variables, host memory) unless explicitly granted. Capabilities are passed explicitly into the sandbox.

5. **Termination guarantee** — All sandboxed code is guaranteed to terminate (subject to the declared resource limits). Infinite loops are prevented by CPU time limits; infinite memory allocation is prevented by memory limits.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Sandbox Policy | Determines whether sandboxed execution is available and which isolation mechanism is used |
| Resource Policy | Governs default resource limits for sandboxed execution |
| Isolation Policy | Controls the isolation mechanism (process, WASM, interpreter, capability-based) |
| Recovery Policy | Specifies behaviour on sandbox violation (rollback, terminate, escalate) |

## Model (What)

### Sandbox Declaration (Core Language Interface)

The Core Language provides a `sandbox` construct that marks a block of code for isolated execution. The *implementation* of the sandbox belongs to the runtime and toolchain:

```orthon
let result = sandbox(resources: {cpu: "100ms", mem: "1MB"})
    # Code here runs in a sandbox
    # No access to host filesystem, network, or environment
    # Limited to 100ms CPU and 1MB memory
    let x = compute_risky_value()
    x * 2
end
```

### Explicit Capability Passing

Capabilities are passed explicitly into the sandbox:

```orthon
let file = File.open("output.txt")
let allowed_networks = ["https://api.example.com"]

let result = sandbox(
    resources: {cpu: "500ms", mem: "10MB"},
    grants: [file, Network(allowed_networks)]
)
    file.write("sandbox output")
    let response = http.get("https://api.example.com/data")
    process(response)
end
# file is closed/released after the sandbox exits
```

### Checkpoint and Rollback

The sandbox can be checkpointed and rolled back:

```orthon
let sbox = Sandbox.new(resources: {cpu: "1s", mem: "50MB"})

sbox.checkpoint("before_batch")
let batch_result = sbox.run(|ctx| process_batch(ctx, data))

if batch_result.is_error():
    sbox.rollback("before_batch")
    # State is restored to checkpoint
```

### Interaction with LLM Toolchain

The Sandbox is the execution environment for the LLM Toolchain's **verification loop**:

```orthon
# Toolchain internal flow:
let generated = llm.generate("fn sort(list) ...")
let sandbox = Sandbox.new(resources: {cpu: "200ms", mem: "5MB"})

let test_result = sandbox.run(|ctx|
    let compiled = ctx.compile(generated)
    ctx.assert(compiled.sort([3, 1, 2]) == [1, 2, 3])
    ctx.assert(compiled.sort([]) == [])
end)
# If test_result is failure, the sandbox is discarded with no host effect
```

## Core Inclusion Filter

Before entering formal validation, this concept is run through the Core Inclusion Filter procedure ([`CORE_INCLUSION_FILTER.md`](../../architecture/CORE_INCLUSION_FILTER.md)):

```
Hypothesis: Level 3 (Standard Library / Toolchain) — sandboxed execution
            is a runtime and toolchain concern, not a Core Language concern.

Library test:   Can Safe Sandbox be implemented as a Standard Library
                function using existing Core constructs?

                The sandbox construct requires:
                - Process isolation (OS-level)
                - Resource monitoring (CPU time, memory usage)
                - State checkpointing and rollback
                - Capability-based access control

                If the Core Language provides basic resource isolation
                (via an Execution Program or runtime API), the sandbox
                can be implemented as a Standard Library module + runtime
                component. The `sandbox` keyword in the example above is
                syntactic sugar over a library function that creates an
                isolated execution context.

                → PASS — can be implemented as a Standard Library + runtime
                  component, assuming the runtime provides isolation primitives.

Pattern test:   Can Safe Sandbox be expressed as a composition of existing
                Primitive Operations and Data Model constructs?

                The sandbox concept composes from:
                - `function` + `call` for sandbox entry/exit
                - `scope` for resource lifetime management
                - `function capture` for capability passing
                - Runtime-managed process/thread isolation
                No new primitive operations are needed at the language level.

                → PASS — composes from existing primitives + runtime layer.

Filter result: Level 3 (Standard Library / Runtime / Toolchain).
               The Core Language defines only the `sandbox` syntactic
               interface (or a library function call); all isolation
               semantics are provided by the runtime and toolchain layers.
```

## Default Strategy

The Default Strategy uses **in-process interpretation with resource monitoring**. The sandbox runs inside the host process but intercepts resource allocation (using the runtime's memory management and scheduling) and checks capabilities at every grant boundary. This provides a good balance of performance and isolation for the common case of LLM-generated code verification.

## Alternative Strategies

| Strategy | Description |
|---|---|
| **Process-level isolation** | Sandboxed code runs as a separate OS process. Maximum isolation but higher overhead per sandbox invocation. Suitable for production deployment of generated code. |
| **WASM-based sandbox** | Code is compiled to WebAssembly and executed in a WASM runtime. Strong isolation guarantees with near-native performance, but requires WASM as a compilation target. |
| **Capability-based isolation** | No sandbox keyword. Instead, capabilities are tracked through the type system (like Pony's reference capabilities). The *type system* guarantees that sandboxed code cannot access unauthorised resources. Highest integration with the language but requires deep type system changes. |
| **No sandbox (trusted execution only)** | All code runs with full host privileges. Simpler runtime but incompatible with LLM-native toolchain verification loops. Generated code must be manually reviewed before execution. |

## Open Questions

1. Should the sandbox be a syntactic keyword (`sandbox ... end`) or a library function (`Sandbox.run(...)`)?
2. Should rollback support all state types (heap, stack, file handles, network connections) or only program memory?
3. How does the sandbox interact with the Execution Program model (if execution strategy is decoupled from semantics, does sandbox isolation vary by strategy)?
4. Should sandbox violations produce compile-time or runtime errors?
5. Can the sandbox mechanism be used for distributed/remote execution (sandbox on a different machine)?
6. Should there be a "sandbox lint" mode where the compiler statically analyses whether code can run in a sandbox?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `what/LIBRARY_BOUNDARY.md`
- [ ] `how/architecture/ARCHITECTURE.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] `how/strategies/LLM_STRATEGY.md`
- [x] `how/concepts/research/LLM_NATIVE_TOOLCHAIN.md`
