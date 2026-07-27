# EDR-047: Async/Await with Async as Explicit Modifier

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Platform

---

### Context

Orthon needs an asynchronous execution model — the ability to perform blocking I/O without blocking the OS thread. The research documents at `how/concepts/research/important/ASYNC_AWAIT.md` and `how/concepts/research/important/ASYNC_AS_EXPLICIT_MODIFIER.md` jointly establish the hypothesis.

The core problem: **expressing suspension without syntactic overhead**. The programmer wants to write straight-line code; the compiler should manage the suspend/resume lifecycle. Two issues were identified and corrected in the strengthened hypothesis:

1. **"`async` ⇒ object state is unstable"** is incorrect. Asynchrony alone does not create state instability — it only indicates the possibility of suspension. Concurrent access is a separate dimension expressed explicitly via `exclusive`.

2. **"`async` cannot be called from a synchronous context"** is too restrictive. Returning `Future` without `await` is a useful idiom for functional composition. `Future` is a first-class value — `await` is required only when the current code needs the result.

The strengthened hypothesis defines `async` as an **orthogonal execution modifier** on `proc`/`fun`/`new`, not a separate abstraction. It composes freely with other modifiers (`exclusive`, `transaction`, `cached`, etc.).

The Decision Pipeline classified ASYNC_AWAIT + ASYNC_AS_EXPLICIT_MODIFIER as **Language** (combined): Async modifies function/proc/new semantics with compiler-level state machine transformation. Not expressible via primitives alone.

---

### Decision

Adopt **stackless coroutines with `async` as an orthogonal modifier** for Orthon:

1. **`async` is an execution modifier on `proc`/`fun`/`new`** — Any declaration kind can be async: `async proc`, `async fun`, `async new`. Async does not create a new semantic category; it is an orthogonal dimension of execution.

2. **`async` means coroutine (suspension only)** — An `async` method may suspend at `await` points. It says nothing about parallelism, multithreading, or concurrent access.

3. **`await` marks suspension points** — The only visible marker of suspension. The compiler transforms the function body into a state machine.

4. **Colourless by default** — `async` functions return `Future<T>`. Calling an `async` function without `await` creates a `Future` and continues immediately. `await` is required only when the current code needs the result. This eliminates artificial context colouring.

5. **Stackless coroutines** — Function becomes a state machine; no separate stack per coroutine. This is the default strategy.

6. **Cooperative scheduling** — No pre-emption; yield points are at `await` boundaries only.

7. **Executor-agnostic** — The async runtime is swappable (single-threaded event loop default, multi-threaded work-stealing alternative).

8. **Structured concurrency via `scope`** — A block where all spawned tasks are automatically awaited or cancelled on exit:

    ```orthon
    scope {
        t1 = spawn async loadA()
        t2 = spawn async loadB()
    }
    # t1 and t2 guaranteed completed (or cancelled on error)
    ```

9. **Explicit `spawn` for parallelism** — `spawn` creates a new concurrent task (running in parallel) and returns `Task<T>` (which is also a `Future`). Without `spawn`, calling an `async` function executes sequentially in the current context.

10. **`exclusive` modifier for access serialisation** — Separates concurrency (`async`) from access control (`exclusive`). A method marked `exclusive` requires exclusive access to the object for its duration.

11. **Async lambdas** — Anonymous coroutines compose with higher-order functions:

    ```orthon
    numbers.map(async fun(x) -> await fetch(x))
    ```

12. **Task cancellation and timeouts** — `cancel()` on `Task` with `is_cancelled` check. Timeout syntax: `await socket.read(timeout: 5s)`.

---

### Consequences

**Positive:**
- `async` as a modifier on `proc`/`fun`/`new` preserves Orthon's three-kind declaration system — async is orthogonal, not a new category.
- Colourless model (Future as first-class value) enables functional composition and eliminates the async/sync function colouring problem.
- `exclusive` modifier cleanly separates suspension from access serialisation — two concerns that are often conflated in other languages.
- `spawn` makes parallelism explicit — no implicit thread creation.
- `scope` provides structured concurrency with automatic lifecycle management.
- Stackless coroutines are zero-overhead when not suspended.
- Modifier composition (`async proc`, `exclusive async fun`, etc.) enables extensibility without altering base semantics.
- LLM-friendly: the pattern matches familiar async/await syntax from Python, JS, Rust, and C#.

**Negative:**
- State machine compilation adds compiler complexity (control-flow graph transformation with suspension points).
- Debugging async functions is harder than debugging sync functions (execution is split across `await` boundaries).
- `Future` as first-class value means the type system must track future types — additional type complexity.
- `scope` requires compiler support for lifetime tracking of spawned tasks.
- The `exclusive` modifier adds a new keyword.
- Timeout syntax requires runtime support for timer scheduling.

---

### Compliance

1. The `what/concepts/ASYNC_AWAIT.md` specification defines the canonical semantics.
2. `async` must be a modifier on `proc`/`fun`/`new`, not a separate keyword introducing a new declaration category.
3. `await` must be the only suspension point (cooperative scheduling).
4. `Future<T>` must be a first-class type — calling an async function without `await` must compile and produce a `Future<T>`.
5. `spawn` must be required for parallel execution; `async` alone implies sequential execution in the current context.
6. `scope` blocks must guarantee automatic awaiting or cancellation of all spawned tasks on scope exit.
7. Stackless coroutines must be the default state machine strategy.
8. All async constructs must be implementable without depending on a specific threading or OS async runtime (implementation independence).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| `async` as a separate declaration kind | Would create a fourth declaration category alongside `proc`/`fun`/`new`, violating the orthogonality principle. Modifier approach is more orthogonal. |
| Strict async colouring (Rust model) | Creates the "what colour is your function" problem — async functions cannot be called from sync contexts without a runtime bridge. Colourless model (Future as value) is more flexible. |
| Stackful coroutines (Go goroutines) | Each coroutine has its own stack — higher memory overhead. Stackless is more memory-efficient and integrates better with Orthon's allocation model. |
| Implicit parallelism (`async` = spawn) | Would make every async call potentially parallel, violating the principle of Explicit Semantics. `spawn` makes parallelism syntactically visible. |
| `async` implies state instability | Incorrect — async is about suspension, not concurrency. The `exclusive` modifier separately handles access serialisation. |
| No structured concurrency | Would leave task lifecycle management to libraries, increasing cognitive load and risking resource leaks. `scope` provides deterministic lifecycle. |

### Gate Validation

All seven gates are required per `DECISION_VALIDATION.md` § Gate Selection (new language construct).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I want to perform I/O without blocking." Async/await is the proven solution across Python, JS, Rust, C#. The strengthened hypothesis (colourless, `exclusive`, `spawn`, `scope`) addresses real pain points in existing async models. Serves VISION.md's Comfortable by Design pillar — linear syntax for non-blocking I/O. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | All terms are precisely defined: `async` = coroutine modifier, `await` = suspension point, `Future` = first-class value, `spawn` = parallel execution, `exclusive` = access serialisation, `scope` = structured lifecycle. No self-referential paradoxes. Behaviour is deterministic across contexts — `async` + `spawn` + `await` have consistent semantics regardless of context. Composition with `proc`/`fun`/`new` is explicitly defined. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "The async modifier model is minimal — removing any component creates a gap." Removing `async` would force callback-based async. Removing colourless futures would force strict colouring. Removing `spawn` would conflate async with parallelism. Removing `scope` would leave lifecycle management to libraries. All components justify their existence. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | `async` as modifier operates within the Core Language layer — it modifies execution semantics of existing declaration kinds. The state machine transformation is a compiler-level operation at the Core → Syntax boundary. No layer violations — async does not depend on the Standard Library or any specific runtime. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | [TRIZ](../../gates/methods/TRIZ_METHOD.md) | Pass | Apparent contradiction: async requires runtime support (event loop), yet must be implementation-independent. Resolution: the *semantic definition* of async is "a function that may suspend at `await` points" — the event loop/runtime is an Implementation Strategy choice. Default strategy: single-threaded event loop. Alternative: multi-threaded work-stealing, or even blocking IO with threads for embedded targets. The colourless model (Future as value) is strategy-independent. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Async marks a function that may suspend; await marks where it suspends; the compiler handles the rest." Remove-one-thing test: removing async would force callback-based I/O — a known anti-pattern. The model matches proven patterns (Python, JS, Rust, C#) and addresses known pain points (colour, lifecycle, parallelism) with orthogonal solutions. Evolution path: async streams, async iterators, and async generators can be added without changing the core model. |
| `LLM_GENERABILITY_GATE` | [Empirical Analysis](../../gates/methods/EMPIRICAL_ANALYSIS_METHOD.md) | Pass | Structural analysis: `async` modifier and `await` expression have single, unambiguous meanings. Schema round-trip: `async fun name() -> Future[T]` is fully expressible in the type system. Hallucination surface: low — async/await is one of the most LLM-generable patterns (Python, JS, Rust). Self-correction: missing `await` where `Future` value used as `T` is a compile error; `await` outside async function is a compile error; combining `async` with wrong declaration kind is a parse error. Common LLM mistakes produce clear compile errors. |

**Gates not applied:** None — all seven gates are required.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
