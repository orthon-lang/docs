# Execution Model

> **⚠️ DRAFT — Placeholder for Phase 7.**
> This document defines what the language guarantees about *how* a
> program executes — separating language semantics from implementation
> strategy.
>
> **Status:** Placeholder — to be filled during Phase 7 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 7,
> [`OPTIMIZATION_MODEL.md`](OPTIMIZATION_MODEL.md),
> [`EXECUTION_PROGRAM.md`](../how/concepts/research/EXECUTION_PROGRAM.md),
> [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md),
> [`IMPLEMENTATION_POLICIES.md`](../how/IMPLEMENTATION_POLICIES.md),
> [`DEFAULT_STRATEGY.md`](../how/strategies/DEFAULT_STRATEGY.md)

---

## Semantics vs. Execution

The language defines **what programs mean**. The Execution Model defines
**how programs execute**. The boundary is the **Execution Program** —
a self-contained, semantically complete artifact.

| Aspect | Language (Semantics) | Implementation (Execution) |
|--------|---------------------|---------------------------|
| Evaluation order | Defined by semantics | May reorder if semantics preserved |
| Memory allocation | Declared intent | Stack, heap, arena — strategy choice |
| Concurrency | Task semantics | Thread pool, event loop — strategy choice |
| Error handling | Result propagation | Stack unwinding, error codes — strategy choice |

## Execution Targets

<!-- To be filled during Phase 7 — supported execution environments -->

| Target | Description | Strategy Profile |
|--------|-------------|-----------------|
| Interpreter | Direct execution | Default |
| AOT | Ahead-of-time compilation | High-Performance |
| JIT | Just-in-time compilation | High-Performance |
| WASM | WebAssembly target | Embedded |
| Container | Container-native execution | Default |

## Invocation Semantics (research)

The return type of contextual invocation is **context-defined** — per the
Execution Context Invocation concept (OQ2 Resolution,
[`EXECUTION_CONTEXT_INVOCATION.md`](../how/concepts/research/essential/EXECUTION_CONTEXT_INVOCATION.md)):
a `delegate` submission returns `void` (a message; the owner's state is
read via `take`), a `defer` submission returns a **deferred invocation**
(a suspendable computation, awaited via `await(ctx)`), and `spawn`/`fork`
submissions return `void` and join the generator stream (`next()`/`stop()`).
This section is filled during Phase 7 upon concept acceptance.

## EDR

- **EDR-NNN:** Execution Model acceptance
  <!-- Created during Phase 7 -->
