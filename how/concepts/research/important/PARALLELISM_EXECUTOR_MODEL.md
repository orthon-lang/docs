# Hypothesis: Parallelism in Orthon — Execution Policy Model

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It examines how CPU-bound parallelism (using additional cores) can be
> integrated into Orthon without expanding the Core language surface.
>
> **Key claim:** The Core language needs only `await` as syntactic sugar
> for `.wait()`. Everything else — executors, task pools, parallel scopes,
> GPU dispatch — is stdlib.
>
> **Last updated:** 2026-07-29
>
> **Source:** Derived from `COLORING_FREE_CONCURRENCY.md` review feedback.
> Supersedes the TaskPool-in-stdlib-only model from that document.
>
> **See also:** [`DELEGATE.md`](../essential/DELEGATE.md),
> [`ASYNC_AWAIT.md`](ASYNC_AWAIT.md),
> [`ASYNC_AS_EXPLICIT_MODIFIER.md`](ASYNC_AS_EXPLICIT_MODIFIER.md),
> [`COLORING_FREE_CONCURRENCY.md`](COLORING_FREE_CONCURRENCY.md),
> [`CONCURRENCY_MODEL.md`](../../../../what/concepts/CONCURRENCY_MODEL.md)

---

## Issue (Why)

### The Problem

Orthon has `delegate` for serialised state access and `async`/`await` for
coroutine suspension. But neither provides **CPU-bound parallelism** —
the ability to use multiple cores for a pure computation:

```orthon
fun processDataset(data: List<Record>) -> List<Result>
    // Single core only — no way to say "use all 8 cores"
    // without threading infrastructure leaking into the language
```

The gap is not about syntax — it is about architecture:

1. **`delegate`** serialises access to a state owner. A mailbox processes
   one message at a time — the opposite of parallelism.
2. **`async`/`await`** enables suspension (non-blocking I/O), but creates
   no parallel tasks. Without `spawn` or an executor, `async` is just
   sequential coroutines.
3. **Existing precedents** (Go goroutines, Erlang `spawn`, Java
   `Executors`) embed concurrency infrastructure into the language or
   standard library — but every language makes different trade-offs
   between core surface, flexibility, and runtime coupling.

**The central design question:** Should Orthon introduce a new language
construct for parallelism, or can it be expressed through composition of
existing primitives plus stdlib infrastructure?

### Why This Matters for Orthon

Orthon's DESIGN_PRINCIPLES.md commits to **Minimal Core**, **Orthogonality**,
and **Implementation Independence**. A new parallelism keyword (`spawn`)
would violate all three if it could be expressed as a library.

At the same time, Orthon targets LLM-native code generation. LLMs need
to discover parallelism APIs reliably. A keyword is always discoverable;
a stdlib API depends on the LLM knowing which module to import.

The tension: **minimal core vs. LLM discoverability**. This hypothesis
resolves it by keeping parallelism entirely in stdlib but designing the
API to be as discoverable and idiomatic as a keyword.

---

## Impact on Semantic Model and Building Blocks

### SEMANTIC_MODEL.md — Six Dimensions

| Dimension | Impact | Severity |
|---|---|---|
| **Evaluation** | **Carve-out required.** Current text: "Eager by default… Expressions evaluate immediately at the point they are reached, in program order. Sub-expressions evaluate left-to-right." A `parallel { }` scope introduces expressions that evaluate on **different cores with no defined interleaving**. The model must acknowledge intra-task determinism vs. inter-task nondeterminism. | **BREAKING** — the "Deterministic Behavior" principle must be qualified as intra-task only |
| **Lifetime** | **Task-extended lifetimes.** A value passed to a parallel executor must outlive the spawning scope. This is the inverse of move-shortened lifetimes (which the model already handles). The model must formalise: a value's lifetime may be extended beyond its declaring scope when ownership is transferred to another executor. The mechanism already exists (`move`), but the semantic consequence is undocumented. | Major — new lifetime pattern to document |
| **Ownership** | **Already sufficient.** `move` transfers ownership. The parallel executor becomes the temporary owner. After `wait()`, ownership returns. No new rules needed. Semantic Invariant 1 ("every value has exactly one owner") and Invariant 6 ("transfer must be syntactically visible") both hold — `move` is the visible marker. | Minor — existing rules cover it |
| **Mutation** | **No change.** `fun`/`proc`/`new` remain the mutation-mode tags. Parallelism is orthogonal to mutation — a `fun` is spawnable, a `proc` on a delegated owner is already isolated. | None |
| **Identity** | **Minor.** `Task<T>` / future handles are identity-bearing by construction (they refer to an ongoing computation). This must be documented as a new stdlib reference type. | Minor |
| **Visibility** | **No change.** Parallelism does not affect scoping or encapsulation. | None |

### PRIMITIVE_BLOCKS.md — Nine Primitives

| Primitive | Impact | Severity |
|---|---|---|
| `literal` | None | None |
| `identifier` | None | None |
| `pack`/`unpack` | None | None |
| `assignment` | None | None |
| **`function`** | **No new tag needed.** Parallelism is a call-site concern, not a declaration-site one. The `fun`/`proc`/`new` tags remain sufficient. | None |
| **`call`** | **Clarification needed.** Currently: "Triggers evaluation of a function body with supplied arguments." With `parallel { }`, `compute(x)` inside the block is still a `call` — but the *timing* of evaluation is deferred and distributed. The primitive's definition does not change — it still triggers evaluation; the executor controls *when* and *where*. This is the same relationship as `call` inside an `async` function: the call still triggers evaluation; `async` controls suspension. | Minor — clarify that `call` is about *what* (trigger evaluation), not *when* (scheduling) |
| `attribute access` | None | None |
| **`scope`** | **No change needed.** `parallel { }` is a lexical scope. Values bound inside it live until scope exit (or until the parallel task completes and ownership returns). Same lifetime rules as any block — the only difference is that inner expressions may evaluate on other cores. | None |
| **`reference`** | **Lifetime verification.** References passed to parallel tasks must live ≥ task duration. This is the standard borrow-checker problem — already present with `delegate` and existing ownership rules. No new primitive behaviour needed. | None — existing rules apply |

**Key insight:** Parallelism introduces **no new primitives** and **no
changes to existing primitive definitions**. It only clarifies the
boundary between `call` (the trigger) and scheduling (the executor's
responsibility).

---

## Proposal

### Core Addition: `await` as Syntactic Sugar for `.wait()`

The only Core change:

```
await expr  →  let __tmp = expr; __tmp.wait()
```

This is ~3 lines in the language spec. The compiler does not need to know
about `Poll[T]`, `Waker`, `Context`, or `Pin`. All async runtime protocol
details live in stdlib.

```orthon
// Core: await sugar
// Stdlib defines:
class Task[T]:
    fun wait(self) -> T       // called implicitly by await
    fun try_wait(self) -> Option[T]
```

### Stdlib: Executor + Parallel Scope

```orthon
// stdlib/parallel.orthon

/// Configurable parallel executor.
/// Implementations may use a thread pool, work stealing, or
/// platform-specific mechanisms.
class Executor(workers: Int):
    /// Submit a closure for parallel execution.
    /// Ownership transfers to the executor via `move`.
    fun submit[T](self, task: move () -> T) -> Task[T]

/// Default shared executor (number of CPU cores).
fun default() -> &Executor
```

#### Idiom A: Parallel Scope

```orthon
use std::parallel

fun process(data):
    parallel:                           // stdlib macro
        let h1 = compute(chunk1)        // submitted to default executor
        let h2 = compute(chunk2)
    // scope exit: wait() called on all handles automatically
    return h1 + h2
```

`parallel { }` desugars to:

```orthon
fun process(data):
    let __exec = parallel.default()
    let h1 = __exec.submit(move compute(chunk1))
    let h2 = __exec.submit(move compute(chunk2))
    let __r1 = h1.wait()
    let __r2 = h2.wait()
    return __r1 + __r2
```

#### Idiom B: Explicit Executor

```orthon
let executor = parallel.Executor(workers = 4)
let h = executor.submit(move expensive(data))
doOtherWork()                   // main thread continues
h.wait()                        // block or suspend
```

#### Idiom C: Parallel Map

```orthon
let results = parallel.map(dataset, compute)
// Equivalent to: dataset.map(compute) but using all cores
```

### How `wait()` Composes with `async`

```orthon
async fun processAsync(data):
    parallel:                          // inside async function
        let h1 = compute(chunk1)       // submit to executor
    return h1                          // await h1.wait() — suspends
```

Inside an `async` function, `wait()` registers a waker and suspends the
coroutine instead of blocking the OS thread. This is a runtime property
of the stdlib implementation, not a language distinction.

---

## Tradeoffs

### Core Surface

| Approach | Core additions | Stdlib surface |
|---|---|---|
| **This proposal** | `await` as sugar (~3 lines) | `Executor`, `Task`, `parallel { }` |
| Core `spawn` keyword | `spawn` + `Task<T>` + ambient executor | None — everything in core |
| Core `Executor` trait | `Executor` + `Awaitable` + `Poll` traits | Concrete executor implementations |
| No change (manual threads) | 0 | User manages OS threads directly |

### Comparison Table

| Dimension | This proposal | Core `spawn` | Core `Executor` trait | Manual threads |
|---|---|---|---|---|
| **Core complexity** | Minimal (~3 lines) | Moderate (keyword + types + lifetime rules) | High (trait + poll + waker risk) | None |
| **Expressiveness** | GPU, remote, custom schedulers all via stdlib | Tied to ambient executor | Any executor impl via trait | Unlimited (raw threads) |
| **Safety** | Existing ownership rules suffice | New cross-task lifetime analysis | Same as this proposal | Unsafe by default |
| **LLM discoverability** | Stdlib import required | Keyword — always visible | Trait + method — moderate | Low (manual thread management) |
| **Runtime coupling** | None — runtime is a library | High — language tied to executor | Medium — trait in core | None |
| **Extensibility** | Third-party executors via stdlib conventions | Extending requires changing Core | Extending via new trait impls | Unlimited but unsafe |
| **Ecosystem precedent** | Java `Executors`, Python `concurrent.futures` | Go goroutines, Erlang `spawn` | Rust `tokio::spawn` | C/POSIX threads |

### Risk: `wait()` Duality

The same method name does different things in sync vs async contexts:

| Context | `wait()` behaviour |
|---|---|
| Sync function | Blocks the OS thread (parker / condition variable) |
| Async function | Registers waker, suspends coroutine (non-blocking) |

**Mitigation options:**
1. Document the duality (orthon is explicit — this is predictable behaviour)
2. Provide explicit variants: `blocking_wait()` vs `suspend_wait()`
3. Compiler warning: "`wait()` in sync context blocks the OS thread"

### Risk: Executor Discovery

Where does `parallel { }` get its default executor?

**Mitigation:** The stdlib provides a sensible default (number of CPU
cores) via a module-level global. Override is explicit:

```orthon
parallel::set_default(my_executor)
```

### Risk: LLM Discoverability

A keyword (`spawn`) is always discoverable. A stdlib API (`std::parallel::block`)
depends on the LLM knowing the import path.

**Mitigation:** The API is designed to be the shortest, most idiomatic
path to parallelism. If empirical data shows LLMs systematically miss
the stdlib API, a `parallel` keyword can be reconsidered with a Tier 1
EDR.

---

## Related Concepts & Alternatives

### Related Concepts

| Concept | File | Relationship |
|---|---|---|
| **DELEGATE** | `essential/DELEGATE.md` | State-owner serialisation. The serial analogue of parallel: `delegate` = one at a time; `Executor` = many at once |
| **ASYNC_AWAIT** | `important/ASYNC_AWAIT.md` | Coroutine model. `async` enables suspension; parallelism adds execution on another core |
| **ASYNC_AS_EXPLICIT_MODIFIER** | `important/ASYNC_AS_EXPLICIT_MODIFIER.md` | Modifier dimensions: `async` (suspension) ≠ `spawn`/parallelism (execution) ≠ `exclusive` (access). This hypothesis follows the same orthogonality |
| **CONCURRENCY_MODEL** | `what/concepts/CONCURRENCY_MODEL.md` (ACCEPTED, EDR-033) | Delegate-based, message-passing model. Independent of parallelism — delegates abstract execution strategy |
| **COLORING_FREE_CONCURRENCY** | `important/COLORING_FREE_CONCURRENCY.md` | Earlier hypothesis on spawn, now superseded by this document for parallelism |
| **SEMANTIC_MODEL** | `what/SEMANTIC_MODEL.md` (ACCEPTED, EDR-013) | Six semantic dimensions. This hypothesis adds intra-task determinism carve-out to Evaluation, task-extended lifetimes to Lifetime |
| **PRIMITIVE_BLOCKS** | `what/PRIMITIVE_BLOCKS.md` (ACCEPTED, EDR-016) | Nine primitives. No new primitives needed — only clarification that `call` triggers evaluation, not scheduling |

### Alternatives

#### Alternative A: Core `spawn` Keyword (Go model)

```orthon
fun process():
    let t1 = spawn compute(chunk1)
    let t2 = spawn compute(chunk2)
    return await t1 + await t2
```

**Pros:** Maximum discoverability — one keyword. Go-proven model.
**Cons:** Language is tied to an ambient executor. No way to configure
thread count, priority, or scheduling policy without Core changes. No
path to GPU or remote execution without more keywords.

#### Alternative B: Core `Executor` Trait (Rust model)

```orthon
trait Executor[F, T]:
    fun execute(self, task: F) -> T
```

**Pros:** Maximally extensible — any execution policy is a trait impl.
**Cons:** Risk of "trait-god" — one trait must cover `() -> A`,
`async () -> B`, streaming, cancellation. Either loses type safety or
expands into a family of traits that recreate complexity at the trait
level.

#### Alternative C: Manual Thread Management (no abstraction)

```orthon
let thread = Thread::spawn(move compute(data))
thread.join()
```

**Pros:** Zero abstraction. Full control.
**Cons:** Unsafe by default. No composition with `async`. No LLM-native
ergonomics. Contradicts Orthon's design philosophy.

---

## Open Questions

1. **`wait()` duality risk.** Is the same method name for blocking
   (sync) and suspending (async) acceptable, or should stdlib provide
   explicit `blocking_wait()` vs `suspend_wait()` variants?

2. **Executor discovery.** Should `parallel { }` resolve its executor
   from thread-local storage, a global, or an explicit parameter?

3. **Error handling in `parallel` scope.** If `compute(chunk1)` panics,
   does the entire `parallel` block abort? Should partial results be
   discardable? The stdlib design needs a clear policy.

4. **Structured concurrency.** Should `parallel { }` guarantee that all
   spawned tasks complete (or are cancelled) at scope exit? This is the
   current design assumption, but should it be enforced or configurable?

5. **Cancellation.** Can a `parallel` scope be cancelled from outside
   (e.g., timeout, user interrupt)? If so, what API surface is needed
   in `Task<T>`?

6. **GPU and remote executors.** Are they pure stdlib, or does the Core
   need to provide *any* abstraction that makes them first-class? This
   hypothesis assumes pure stdlib, but the question should be revisited
   when a concrete GPU executor is designed.

7. **LLM discovery validation.** The hypothesis assumes `parallel { }`
   in stdlib is discoverable enough. This should be tested empirically
   during prototyping — if LLMs systematically fail to generate
   correct parallel code, a keyword may be justified.

8. **Relationship to Execution Program model.** How does parallelism
   compose with Orthon's Execution Program concept (a program whose
   semantics are decoupled from execution strategy)? Can an Execution
   Program declare concurrency requirements declaratively?

---

## Decision History

| Date | Decision |
|---|---|
| 2026-07-29 | Initial hypothesis. Supersedes the TaskPool-in-stdlib model from COLORING_FREE_CONCURRENCY.md. Key changes: `await` as sugar (no Awaitable in Core), `parallel { }` idiom instead of TaskPool, `Executor` as stdlib abstraction |
