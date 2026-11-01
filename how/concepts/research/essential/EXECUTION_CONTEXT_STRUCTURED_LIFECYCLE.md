# Execution Context Structured Lifecycle (Hypothesis)

> **HYPOTHESIS — research input, not accepted Orthon semantics.**
> This document develops lifecycle invariants suggested by Kotlin and Swift
> without changing the colourless Invocation model accepted in
> [EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md).
> No human sign-off or new EDR has been recorded.

> **Related:** [Execution Context Invocation](EXECUTION_CONTEXT_INVOCATION.md),
> [Execution Context Cancellation](EXECUTION_CONTEXT_CANCELLATION.md),
> [Execution Context Error Propagation](EXECUTION_CONTEXT_ERROR_PROPAGATION.md),
> [Detached Execution](../important/DETACHED_EXECUTION.md), and
> [Concurrency Approaches and Trends](../CONCURRENCY_APPROACHES_AND_TRENDS.md).

## Problem

EDR-085 makes an Execution Context an explicit owner of execution policy,
state, and deterministic destruction. It also requires unresolved `spawn()`
and `fork()` work to be materialised or stopped before its context is
destroyed. These rules prevent accidental abandonment, but they do not yet
answer every lifecycle question.

In particular, a program needs a locally understandable answer to these
questions:

- Which context owns submitted work?
- Which work must finish, be observed, or be stopped before that owner ends?
- How do cancellation and terminal failure affect related work?
- When does a task group add a real new execution policy rather than duplicate
  an existing Context and lexical scope?

Without clear answers, an implementation can be efficient while programs
remain difficult to reason about at context destruction or failure boundaries.

## Accepted Constraints

- Functions remain colourless. A function declares what it computes; an
  Execution Context declares how an invocation executes (EDR-085).
- `defer(obj)` and `delegate(obj)` each own one object. `spawn()` and `fork()`
  are stateless worker contexts.
- Context destruction is deterministic. Unresolved `spawn()` or `fork()` work
  is a program error unless it is explicitly materialised or stopped.
- `await`, `take`, `next`, `gather`, and `stop` are explicit materialisation or
  resolution operations. They must not become invisible side effects of an
  unrelated call.
- The language specifies observable lifecycle behavior, not whether a strategy
  uses an event loop, stackless state machine, fiber, virtual thread, or OS
  thread.
- Cancellation and error propagation remain separate open questions. This
  hypothesis does not decide either one.

## Hypothesis

**An Execution Context should be the sole semantic owner of the work submitted
to it, and each Context must expose an explicit, locally visible resolution
path before its lifetime ends.**

This preserves EDR-085's model: a lexical scope bounds the lifetime of a
Context value; it is not an independent concurrency construct. A hypothetical
`scope` form may only be syntactic sugar for Context creation, use, and
deterministic resolution. It must not introduce a second ownership tree.

Kotlin's structured-concurrency scopes and Swift's task and actor isolation
are useful evidence for the invariant, but their `suspend`/`async` function
colouring and their runtime-specific syntax are not proposed for Orthon.

## Candidate Invariants

| Invariant | Consequence for Orthon | Status |
|---|---|---|
| One work owner | Every submitted invocation belongs to one Execution Context until it is resolved, stopped, or explicitly detached. | Candidate |
| Bounded lifetime | The Context's lexical lifetime bounds the obligation to resolve its work. | Partly accepted by EDR-085 |
| Visible resolution | Result observation, cancellation, and detachment use named operations. | Partly accepted by EDR-085 |
| Isolated mutable state | Single-owner Contexts serialise access to their owned state; worker captures meet `Send` or `Move` constraints. | Accepted by EDR-085 |
| Strategy independence | No invariant depends on a particular scheduler, executor, or thread representation. | Accepted architectural constraint |
| No implicit task group | A task-group behavior that changes cancellation, joining, or error propagation needs an explicit Context policy or proven sugar. | Candidate |

## Model to Test

The existing model already supplies the lifecycle boundary:

```orthon
let workers = spawn()
workers |> fetch(left_url)
workers |> fetch(right_url)
let pages = gather(workers)
```

`workers` is the owner of the submitted worker invocations. `gather(workers)`
is visible resolution; `stop(workers)` is the alternative resolution path.
Leaving the lexical scope without either action is an error under EDR-085.

A block form, if Phase 5 later establishes a need for one, must desugar to the
same ownership and resolution behavior. It cannot make `gather` implicit or
silently introduce a second owner for the work. No block syntax is proposed by
this hypothesis.

If a program needs parent-child cancellation, join-on-exit, or failure
aggregation beyond this contract, the first test is whether a new Context
constructor can express that policy. A standalone `scope` primitive is not a
default answer because it would overlap with Context lifetime.

## Comparative Evidence

| Source | Useful idea | Not adopted by implication |
|---|---|---|
| Kotlin coroutines | Related work has a parent-child lifecycle; cancellation and completion are not orphaned. | `suspend` function colouring, coroutine dispatchers, or implicit scopes. |
| Swift concurrency | State isolation and explicit crossing of actor boundaries make concurrency reasoning local. | `async` function colouring, actor syntax, or Apple runtime behavior. |
| Java virtual threads | Synchronous-looking code can scale when waiting does not monopolise an OS thread. | JVM-specific execution semantics or implicit ownership of spawned work. |
| Go goroutines | Lightweight work units can be scheduled independently of OS threads. | Detached-by-default work or invisible lifetime obligations. |

## Preliminary Decision Pipeline

| Question | Preliminary answer |
|---|---|
| What problem is solved? | Make ownership and completion obligations of contextual work locally understandable. |
| Language or library? | Existing lifecycle rules are Core semantics; additional group behavior may be a Context policy or library sugar. |
| Can existing primitives express it? | EDR-085 already expresses explicit ownership and resolution for current contexts. |
| Does it add semantics? | Only if it changes cancellation, joining, failure propagation, or destruction obligations. |
| Is a new primitive justified? | Not yet. A separate `scope` primitive is presumed redundant until composition fails. |

The pipeline is intentionally preliminary. It is not an acceptance decision.

## Interaction With Existing Research

- [Execution Context Cancellation](EXECUTION_CONTEXT_CANCELLATION.md) decides
  the meaning and completion of cancellation operations.
- [Execution Context Error Propagation](EXECUTION_CONTEXT_ERROR_PROPAGATION.md)
  decides how recoverable and terminal failures cross a Context boundary.
- [Detached Execution](../important/DETACHED_EXECUTION.md) decides the explicit
  exception to the normal resolution obligation.
- A future task-group proposal must demonstrate a distinct execution policy or
  a valid desugaring over these concepts before it can add syntax.

## Open Questions

1. Does a `defer` Context accept one invocation or a sequence of invocations,
   and how does that choice determine its resolution obligation?
2. Can `spawn()` or `fork()` express a useful bounded worker group without a
   dedicated task-group Context?
3. What happens when one member of related work ends with a terminal system
   error while other work is still pending?
4. Is a context-level cancellation request sufficient, or are per-invocation
   handles necessary?
5. Can a future block syntax prove itself to be pure sugar, including failure
   and cancellation behavior?

## Decision History

- **2026-11-01:** Created from a review of Execution Context Invocation and
  comparative analysis of Kotlin, Swift, Java virtual threads, and Go. It
  records candidate invariants only; EDR-085 remains unchanged.

> **Human Sign-off:** Not applicable while this document remains a research
> hypothesis. No EDR may be filed without the author's explicit sign-off.

