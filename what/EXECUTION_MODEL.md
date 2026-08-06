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

## Invocation Semantics

> **Accepted — EDR-085 (Execution Context Invocation).**
> Unified invocation model: one operation — **Invocation** — with two
> independent axes: *what to invoke* (function / method) and *how to
> execute* (the Execution Context). Functions are **colourless**: the
> same function can be invoked immediately, deferred, or distributed
> without modification.

### Execution Contexts

An **Execution Context** (`execution_context`, the 10th primitive) is an
environment that executes an invocation according to a policy. It is
created by a constructor; it owns zero or one state object and has
deterministic destruction.

| Context | Constructor | Extraction | Policy |
|---------|-------------|------------|--------|
| Immediate | — | `fn(args)` | Executes now, in the current environment (no context, no operator) |
| Coroutine | `defer(obj)` | `await(ctx)` | Suspends/resumes; owns one suspendable computation |
| Actor (mailbox) | `delegate(obj)` | `take(ctx)` | Serialised messages to a single owned state |
| Thread (shared memory) | `spawn()` | `next()` / `stop()`, `grab`/`gather` | Independent stateless workers (shared memory) |
| Process (isolated memory) | `fork()` | `next()` / `stop()`, `grab`/`gather` | Independent stateless workers (isolated memory) |

`defer(obj)`/`delegate(obj)` always wrap an object — bare contexts do not
exist. `spawn()`/`fork()` are stateless worker contexts; captured data
must satisfy the `Send` (spawn) / `Move` = `Send` + serialisable (fork)
marker traits, checked at compile time.

### Submission Return (context-defined)

The return type of a contextual submission is **context-defined** (OQ2
Resolution): a `delegate` submission returns `void` (a message; the
owner's state is read via `take(ctx)`), a `defer` submission returns a
**deferred invocation** (a suspendable computation, awaited via
`await(ctx)`), and `spawn`/`fork` submissions return `void` and join the
generator stream (`next()`/`stop()`, `grab`/`gather`). The `defer`
transport type name is a Phase 5 detail — the Core commits to the
concept, not the name.

### Materialisation Vocabulary

`await`/`take` are context-specific named functions (compiler-intrinsic
support permitted); `grab`/`gather` are StdLib sugar over the generator
protocol. For `spawn`/`fork`, no safe automatic behaviour for pending
work exists — explicit resolution (materialise via `next()`/`gather()`,
or cancel via `stop()`) is required before destruction; destroying a
context with unresolved work is a program error. Worker-shutdown manner
and enforcement are Implementation Strategy concerns (Phase 7).

### Fixed Inline Call Semantics

`fn(args)` inside any context is context-independent — always a
synchronous, inline call; it never blocks or yields by itself (the
callee may suspend at an explicit extraction point). No self-delegation;
long computations are submitted cross-context via the distribution
operator.

### Resource Lifecycle

The wrapped object drops automatically as a consequence of ownership +
lifetime (not a new rule); `take(ctx)` moves it out. `using x = expr`
is pure syntactic sugar over context + scope + deterministic destructor
— it introduces no new semantics. Resource management, `async`/`await`,
`spawn`, and `delegate send` are not separate features; they are the
same Invocation pattern. `SCOPED_RESOURCE_LIFECYCLE.md` is superseded.

### Distribution Operator

The distribution operator (`ctx |> fn(args)`, glyph provisional) is
deferred to Phase 5 (Syntax Design); the semantic model — stateless
workers, independent, parallel — is fixed by EDR-085.

## EDR

- **EDR-NNN:** Execution Model acceptance
  <!-- Created during Phase 7 -->
