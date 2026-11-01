# Concurrency Approaches and Trends

> **Research note — comparative evidence, not Orthon specification.**
> This note records the design space around concurrency and the direction that
> current Orthon decisions already imply. It does not amend
> [EDR-085](../../decision_records/architecture/EDR-085-execution-context-invocation.md).

> **Related:** [Execution Context Invocation](essential/EXECUTION_CONTEXT_INVOCATION.md),
> [Execution Context Structured Lifecycle](essential/EXECUTION_CONTEXT_STRUCTURED_LIFECYCLE.md),
> [Execution Model](../../../what/EXECUTION_MODEL.md), and
> [Implementation Strategies](../../strategies/IMPLEMENTATION_STRATEGIES.md).

## Purpose

Concurrency designs often mix three independent choices:

1. **Source semantics:** where code may suspend, create work, observe results,
   or access mutable state.
2. **Lifecycle semantics:** who owns work, cancellation, errors, and cleanup.
3. **Execution strategy:** event loop, work-stealing pool, fiber scheduler,
   virtual thread runtime, or OS threads.

Comparisons that call one language "async" and another "threaded" usually
combine these choices. The distinction matters for Orthon because its
Execution Context model places them at different layers.

## Contemporary Models

| Model | Examples | Source surface | Lifecycle model | Scheduler relation |
|---|---|---|---|---|
| Coloured futures | Python, JavaScript, Rust | `async`/`await` appears in function APIs. | Varies by framework; often explicit handles. | Python and JavaScript rely on event loops; Rust deliberately leaves the executor external. |
| Suspending coroutines | Kotlin, Swift | Suspension is marked; task creation is explicit. | Structured scopes or task trees are first-class ideas. | Runtime dispatches coroutines or tasks over threads. |
| Lightweight threads | Go, Java virtual threads | Task creation is explicit; ordinary calls usually retain synchronous shape. | Go leaves much lifetime management to libraries; Java adds structured APIs separately. | Runtime multiplexes many tasks over fewer OS threads. |
| Actor isolation | Erlang-family systems, Swift actors, Orthon `delegate` Context | Send work to a state owner rather than share mutation. | Owner lifetime and message semantics govern work. | Mailboxes and executors are implementation details after the isolation contract. |

## Historical Direction

The evolution is not a straight replacement of `async`/`await` by virtual
threads. It is a convergence on several separate lessons:

- Callback APIs made I/O concurrency possible but obscured control flow and
  error propagation.
- Futures and promises made completion values composable, then `async`/`await`
  restored linear source code over those values.
- Go made lightweight runtime tasks and communication channels mainstream.
- Kotlin, Swift, and structured-concurrency libraries showed that cheap tasks
  still need ownership, cancellation, and failure boundaries.
- Java virtual threads made blocking-looking source practical at high I/O
  concurrency, but did not remove the need to create and manage tasks.
- Modern designs increasingly hide the scheduler while making lifecycle and
  state-isolation boundaries explicit.

## What Orthon Already Chooses

EDR-085 already commits Orthon to these positions:

| Concern | Orthon position |
|---|---|
| Function API | Colourless: a function describes computation, not execution policy. |
| Execution policy | Explicit value: `defer(obj)`, `delegate(obj)`, `spawn()`, or `fork()`. |
| Invocation boundary | Explicit submission to Context, distinct from immediate `fn(args)`. |
| Result boundary | Named materialisation: `await`, `take`, `next`, `gather`, or `stop`. |
| Mutable state | Single-owner Contexts isolate owned state; distributed work requires `Send` or `Move`. |
| Scheduler | Implementation Strategy, not source semantics. |
| Worker lifetime | Unresolved `spawn()` and `fork()` work cannot be silently abandoned. |

This is neither a Python-style coloured coroutine language nor Java virtual
threads exposed as a language primitive. It is an explicit policy and
ownership model that can be implemented by either a coroutine scheduler or
lightweight threads.

## Useful Ideas From Kotlin and Swift

The useful import is not their syntax. It is a set of questions that every
Context policy must answer:

- What owns this work?
- Which work is still outstanding at destruction?
- How is cancellation requested and observed?
- How do recoverable errors and terminal failures cross the boundary?
- Which mutable state is isolated, and where may code resume after waiting?

For Orthon, these questions lead to the candidate structured-lifecycle
invariant: an Execution Context owns submitted work and exposes an explicit
resolution path. They do not imply `async` or `suspend` function modifiers.

## Useful Ideas From Virtual Threads and Goroutines

Virtual threads and goroutines demonstrate that the physical cost of a
waiting task need not shape every function signature. This aligns with
Orthon's colourless functions and strategy independence.

They belong below the semantic boundary:

```text
Orthon source:        Execution Context + submission + materialisation
Language guarantees:  ownership, isolation, lifecycle, observable results
Implementation:       event loop | fibers | virtual threads | M:N scheduler
```

A virtual-thread or goroutine-like scheduler may implement `defer` or
`spawn`; it is not, by itself, a reason to add another Context constructor.
A new constructor is warranted only when it exposes a different observable
policy, such as a distinct state-ownership or result-lifecycle contract.

## Current Tension

EDR-085 intentionally requires explicit resolution of worker work. Kotlin
and Swift often provide automatic parent-child completion and cancellation.
The design question is therefore not whether Orthon should copy a `scope`
keyword. It is whether a future Context policy or proven syntactic sugar can
offer bounded group behavior without duplicating Context ownership or hiding
result, cancellation, and error semantics.

That question belongs to
[Execution Context Structured Lifecycle](essential/EXECUTION_CONTEXT_STRUCTURED_LIFECYCLE.md),
[Execution Context Cancellation](essential/EXECUTION_CONTEXT_CANCELLATION.md),
and [Execution Context Error Propagation](essential/EXECUTION_CONTEXT_ERROR_PROPAGATION.md).

## Research Conclusion

The direction currently most consistent with Orthon is:

1. Keep functions colourless.
2. Keep the execution policy explicit and first-class as an Execution Context.
3. Keep materialisation, cancellation, and detachment visible operations.
4. Strengthen lifecycle and state-isolation guarantees using lessons from
   Kotlin and Swift.
5. Permit virtual threads, goroutines, fibers, or event loops as interchangeable
   Implementation Strategy choices.

This is a research conclusion, not a new language decision. Any new Context
policy, task-group behavior, or syntax must first pass the Decision Pipeline
and language-design gate.
