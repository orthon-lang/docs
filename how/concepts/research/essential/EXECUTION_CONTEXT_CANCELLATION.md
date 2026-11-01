# Execution Context Cancellation (Hypothesis)

> **HYPOTHESIS — research input, not accepted Orthon semantics.**
> Raised from the follow-up questions in
> [EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md#open-questions-for-follow-up-decisions).
> No human sign-off or new EDR has been recorded.
>
> **Related:** [Execution Context Invocation](EXECUTION_CONTEXT_INVOCATION.md),
> [Execution Model](../../../../what/EXECUTION_MODEL.md),
> [Detached Execution](../important/DETACHED_EXECUTION.md),
> [System Error](../../../../what/concepts/SYSTEM_ERROR.md).

## Problem

EDR-085 requires pending `spawn()` and `fork()` work to be resolved before
context destruction: obtain results or call `stop()`. It describes `stop()`
as cancelling pending work, while leaving worker-shutdown mechanics to an
Implementation Strategy. It does not say whether `stop()` affects a worker
that is already running, how cancellation completion is observed, or what
happens to results produced before the stop request.

For `defer(obj)` and `delegate(obj)`, destruction drops the owned object
and discards unprocessed invocations. It is unclear whether a programmer
can cancel one deferred invocation or one queued message earlier, while
keeping the context and its owner alive.

## Accepted Constraints

- Cancellation cannot be an implicit consequence of dropping a
  `spawn`/`fork` context with unresolved work. Such a drop is a program
  error; explicit resolution remains required (EDR-085).
- `stop()` is part of the `spawn`/`fork` result-stream protocol and
  cancels pending work. The manner of worker shutdown and enforcement
  are Implementation Strategy concerns (EDR-085).
- `defer` and `delegate` each own exactly one object. The object's
  destruction follows ownership and lifetime; `take(ctx)` moves it out.
  Any earlier cancellation mechanism must preserve this contract.
- [Detached Execution](../important/DETACHED_EXECUTION.md) separately
  investigates deliberately abandoning the result obligation. Cancelling
  work and detaching it are different intents.

## Hypothesis

An explicit cancellation request should have a context-defined scope and
an observable completion outcome. It should not rely on context
destruction, and it should not erase already produced results without a
documented rule. The existing `stop()` may be sufficient for worker
contexts; cancellation of one `defer` invocation or `delegate` message
may require a different composition or may be unnecessary.

This is a question about observable language behavior. It does not
choose between cooperative cancellation, interruption, or termination
as implementation mechanisms.

## Cases to Resolve

| Case | Current guarantee | Unresolved behavior |
|---|---|---|
| Worker submission has not started | `stop()` cancels pending work | Whether a cancelled submission produces a stream element or other acknowledgement |
| Worker submission is running | Explicit resolution is required before context destruction | Whether `stop()` requests interruption, waits, or leaves that work to complete |
| Worker already produced a result | Results use the generator stream | Whether `stop()` preserves or discards unread results |
| `defer` invocation is suspended | `await(ctx)` is its observation mechanism | Whether one invocation can be cancelled while preserving the context |
| `delegate` message is queued | Submission returns `void`; owner is read through `take(ctx)` | Whether one queued message can be withdrawn without ending the owner |
| Context is destroyed | Owned object drops; unresolved worker work is a program error | Whether any explicit cancellation operation has completed before destruction |

The table records gaps rather than specifying new behavior.

## External Examples

- [Go `context.Context`](https://pkg.go.dev/context) carries a
  cancellation signal and deadline through a call chain. Cancellation
  signals that work should stop; it does not forcibly terminate a
  computation. This is useful evidence for separating a request from
  its completion, but Go's context is not Orthon's Execution Context.
- [Tokio task cancellation](https://docs.rs/tokio/latest/tokio/task/index.html#cancellation)
  is observed at yield points; aborting a task does not guarantee it
  loses a race with normal completion. This demonstrates why a
  cancellation request needs a defined completion outcome.
- [Java `Future.cancel`](https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/concurrent/Future.html)
  may prevent a queued task from starting without guaranteeing abrupt
  termination of a running task. Queue cancellation and running-work
  cancellation are separate cases.

## Alternatives to Test

1. **Keep `stop()` as context-wide cancellation.** This matches the
   existing result-stream protocol, but cannot express cancellation
   of one selected submission unless the context owns only that work.
2. **Add per-submission cancellation.** This is more precise, but
   `spawn`/`fork` submissions currently return `void`; introducing a
   handle would reopen an accepted EDR-085 decision.
3. **Use a cancellation signal passed as ordinary data.** This may
   compose with existing primitives but would need a rule for work
   that is queued, blocked, or ignores the signal.
4. **Restrict cancellation to queued work.** This is simpler but leaves
   running work to complete and requires a clear shutdown obligation.

No alternative is selected. Test whether the accepted `stop()` contract
and existing composition can cover the needed cases before proposing
new Core semantics.

## Open Questions

1. Is `stop()` a request or a completed transition? How does the caller
   know all workers and owned resources have stopped?
2. Does `stop()` affect running work as well as queued work? If so,
   what are its observable safe points and cleanup guarantees?
3. Are unread results retained, discarded, or reported when `stop()`
   is called? Can a stream be read after stopping?
4. Is cancellation of one `defer` invocation or `delegate` message
   needed without destroying the whole context?
5. Does cancellation itself produce a recoverable value, a distinct
   control outcome, or only a state change? How would this compose
   with [error handling](EXECUTION_CONTEXT_ERROR_PROPAGATION.md)?
6. What happens when cancellation races with completion, a domain
   error, or a terminal system error?

## Route to Decision

Apply the [Decision Pipeline](../../../process/DECISION_PIPELINE.md) before
designing a cancellation feature. Separate observable guarantees from
worker-shutdown strategy. Check any proposal against the
[Language Design Gate](../../../gates/_language-design.md), EDR-085,
and the detached-execution hypothesis. An EDR requires the author's
human sign-off; this document records none.
