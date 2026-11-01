# Execution Context Error Propagation (Hypothesis)

> **HYPOTHESIS — research input, not accepted Orthon semantics.**
> Raised from the follow-up questions in
> [EDR-085](../../../decision_records/architecture/EDR-085-execution-context-invocation.md#open-questions-for-follow-up-decisions).
> No human sign-off or new EDR has been recorded.
>
> **Related:** [Execution Context Invocation](EXECUTION_CONTEXT_INVOCATION.md),
> [Error Handling](../../../../what/concepts/ERROR_HANDLING.md),
> [Error Union](../../../../what/concepts/ERROR_UNION.md),
> [System Error](../../../../what/concepts/SYSTEM_ERROR.md),
> [Error Representation Unification](ERROR_REPRESENTATION_UNIFICATION.md).

## Problem

EDR-085 defines how an invocation is submitted and how each context exposes
work: `await(ctx)` for `defer(obj)`, `take(ctx)` for `delegate(obj)`, and a
result stream for `spawn()` or `fork()`. It does not define how a submitted
function's recoverable error crosses that boundary, or which scope terminates
when the function encounters a terminal system error.

Without a boundary rule, the same fallible function might be handled
explicitly when called inline but become invisible, implicitly propagated,
or mistaken for a context failure when submitted. A `delegate` message is
especially unclear because submission returns `void` and `take(ctx)` moves
out the owner rather than returning a per-message result.

## Accepted Constraints

- A function's recoverable failure is a value: `Result<T, E>` or the
  currently accepted error union `!T`. The caller must handle it explicitly;
  unchecked exceptions are rejected (EDR-020 and EDR-023).
- A terminal `PROGRAM_ERROR`, `EXECUTION_ERROR`, or `PLATFORM_ERROR` is not a
  recoverable value (EDR-089). Its diagnostic and termination contract must
  remain distinct from a function's declared error result.
- `delegate` submission returns `void`; `defer` submission returns a deferred
  invocation; `spawn` and `fork` submissions return `void` and join a result
  stream (EDR-085). This hypothesis does not change those return types.
- The separate [Error Representation Unification](ERROR_REPRESENTATION_UNIFICATION.md)
  hypothesis may change the spelling of recoverable errors. This research
  concerns boundary behavior, independent of that spelling.

## Hypothesis

Submitting a fallible function should preserve the function's declared
recoverable result as data until a context-specific observation point. A
context should not turn an error value into an implicit exception or erase
it merely because execution moved elsewhere. The observation point and
the handling of terminal failures still need a decision for each context.

This is a candidate invariant, not a claim that the current documents
already satisfy it. In particular, it does not imply a new result handle
for `delegate` or a recoverable wrapper for a terminal system error.

## Cases to Resolve

| Context | Existing observation | Recoverable-error question | Terminal-error question |
|---|---|---|---|
| Immediate call | Function result | Existing explicit handling applies | Existing system-error contract applies |
| `defer(obj)` | `await(ctx)` | Does awaiting expose the function's `Result`/`!T`, and how are multiple submissions distinguished? | Does failure terminate the context or a wider scope? |
| `delegate(obj)` | Owner state through `take(ctx)` | How can the sender observe a failed message without inventing a per-message return? | Does the owner, mailbox, or program terminate? |
| `spawn()` / `fork()` | `next()` stream; `grab`/`gather` sugar | Does each stream element retain its own success/error value, including ordering and partial results? | Can later results still be observed after one worker fails? |

The cases above are questions. No row defines new syntax or accepted
behavior.

## External Examples

- [Rust `JoinHandle::join`](https://doc.rust-lang.org/stable/std/thread/struct.JoinHandle.html)
  returns a result and reports a thread panic as `Err`. This shows one way
  to make a worker failure observable, but Orthon must not silently turn
  its terminal system errors into ordinary recoverable values.
- [Python `asyncio.TaskGroup`](https://docs.python.org/3/library/asyncio-task.html#task-groups)
  waits for its tasks and groups exceptions. Its exception model is a
  comparison point, not a candidate for Orthon's error representation.

## Alternatives to Test

1. **Preserve each function result at materialisation.** This keeps
   recoverable errors explicit, but the `delegate` and `defer` observation
   contracts need precise definitions.
2. **Store failures in context or owner state.** This may fit
   `delegate`, but a generic error slot could hide which message failed
   or allow an error to go unhandled.
3. **Make one failed submission fail the entire context.** This is simple
   for callers but may conflate a recoverable domain error with a
   terminal system error and discard unrelated work.

None of these alternatives is selected here. Error behavior must first
be tested against EDR-085's context-defined submission returns and the
accepted no-silent-failure rule.

## Open Questions

1. What exact value does `await(ctx)` expose after a fallible `defer`
   submission, especially if the same context receives several calls?
2. Where does a `delegate` message's recoverable error remain visible,
   given `void` submission and owner extraction through `take(ctx)`?
3. Does a worker result stream contain each function's recoverable
   result as one element? What happens to other elements after an error?
4. What is the termination scope and diagnostic path for each of the
   three terminal system-error classes in each context type?
5. How do cleanup errors interact with context destruction and with a
   submitted function's earlier error?
6. Can the same boundary contract survive either accepted error spelling
   (`Result<T, E>` and `!T`) and the proposed unified representation?

## Route to Decision

Apply the [Decision Pipeline](../../../process/DECISION_PIPELINE.md) before
designing a new mechanism. First test whether existing function result
types, ownership, and iterator composition express the required behavior.
If a Core semantic change is necessary, review it against the
[Language Design Gate](../../../gates/_language-design.md) and record the
interaction with EDR-020, EDR-023, EDR-085, and EDR-089. An EDR requires
the author's human sign-off; this document records none.
