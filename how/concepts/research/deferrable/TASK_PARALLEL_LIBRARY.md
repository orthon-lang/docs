# Task Parallel Library

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How should a language expose a structured, task-based parallelism API without requiring developers to manage low-level threads directly?

Java historically provides `Thread`, `ExecutorService`, and `ForkJoinPool`, which are powerful but still expose low-level thread management concepts. C# introduced the Task Parallel Library (TPL) to make asynchronous and parallel computation composable through `Task`, `Task<T>`, and higher-level constructs such as `Parallel.For`.

The core problem is: should Orthon provide a built-in task-based concurrency model as part of its language-level abstractions, or should parallelism remain a library concern?

## Examples

| Language | Task abstraction | Data parallelism | Thread abstraction exposed | Composability |
|---|---|---|---|---|
| **Java** | `CompletableFuture`, `ExecutorService` | `parallelStream()` | Yes | Moderate |
| **C#** | `Task<T>`, `Parallel.For` | Yes | No, mostly hidden | High |
| **Rust** | `Future`, runtime libraries | Yes via crates | No | High |
| **Go** | Goroutines, channels | Yes | No, explicit goroutine | Moderate |
| **Python** | `concurrent.futures`, `asyncio` | Limited | Yes | Moderate |

### C# Example

```csharp
var tasks = new List<Task<int>>();
for (int i = 0; i < 10; i++) {
    int copy = i;
    tasks.Add(Task.Run(() => Compute(copy)));
}
int[] results = await Task.WhenAll(tasks);
```

### C# Parallel Loop Example

```csharp
Parallel.For(0, 100, i => {
    Process(i);
});
```

## Implications for Orthon

1. **Task as first-class abstraction** — Orthon can expose a task or future type for background computation that is composable and interoperable with async constructs.
2. **Hidden thread management** — The API should abstract over threads and executors, allowing developers to express parallel intent without manual thread creation.
3. **Data parallel primitives** — A `parallel` loop or collection parallel operations can reduce boilerplate for common parallel patterns.
4. **Separation from async/await** — Task parallelism is not the same as asynchronous I/O; Orthon should distinguish between CPU-bound parallel execution and suspend/resume async semantics.
5. **Cancellation and progress** — The task model should include cancellation and cooperative termination as first-class features.

## Open Questions

1. Should Orthon make task parallelism part of the core language, or leave it to a standard library built on more primitive concurrency constructs?
2. What is the default execution strategy for parallel tasks: work-stealing, thread pool, or user-specified executor?
3. How should Orthon handle shared mutable state in parallel loops to avoid data races?
4. Should the API support both task-based and data-parallel operations, or focus on one paradigm first?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [ASYNC_AWAIT.md](ASYNC_AWAIT.md)
- [CONCURRENCY.md](CONCURRENCY.md)
- [SAFE_SANDBOX.md](SAFE_SANDBOX.md)
