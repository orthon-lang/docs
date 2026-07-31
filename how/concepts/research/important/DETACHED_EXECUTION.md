# Detached Execution (Fire-and-Forget)

> **⚠️ DRAFT — This document is a preliminary research hypothesis.**
> It has not passed Concept Design Review.
>
> **Source:** `EXECUTION_CONTEXT_INVOCATION.md` (OQ8 Resolution, 2026-07-31)

## Issue (Why)

The OQ8 Resolution in
[`EXECUTION_CONTEXT_INVOCATION.md`](../essential/EXECUTION_CONTEXT_INVOCATION.md)
establishes that destroying a `spawn()`/`fork()` context with unresolved
work is a program error: pending work must be explicitly resolved
(materialised via `next()`/`gather()`, or cancelled via `stop()`) before
the context is destroyed.

This makes **fire-and-forget** — submit work, deliberately abandon the
result, and let the worker run independently of the submitting context —
inexpressible. There is no way to say "this task's result is
intentionally unneeded."

Fire-and-forget is a real pattern:

- Background logging and telemetry emission
- Cache warm-up and pre-computation
- Notification and webhook delivery
- Periodic housekeeping that outlives the submitting scope

## Examples

| Language | Mechanism | Notes |
|----------|-----------|-------|
| Rust | `thread::spawn` returns a `JoinHandle`; dropping it detaches the thread | Detach by default; explicit `join()` to wait |
| Java | `ExecutorService.execute()` (fire-and-forget) vs. `submit()` (`Future`) | Separate entry points for detached vs. tracked work |
| Erlang | `spawn` is inherently fire-and-forget; results via message passing | Detachment is the default model |
| Go | `go fn()` goroutine, detached by default | No handle unless an explicit channel is used |

The comparison spans a design axis: **detach-by-default with explicit
tracking** (Rust, Go) versus **tracked-by-default with explicit detach**
(the direction OQ8 implies for Orthon).

## Hypothesis

Orthon needs an explicit **detach** mechanism: a way to surrender a
submitted invocation's result obligation so the worker continues
independently, decoupled from the submitting context's destruction. The
question to resolve is the *level* and *shape* of that mechanism.

## Open Questions

1. **Level of detach.** Per-invocation (`detach(handle)`) or
   per-context (the whole `spawn`/`fork` context becomes detached)? A
   per-invocation detach keeps the context reusable; a per-context
   detach is coarser but simpler.
2. **Ownership after detach.** Who owns the worker once detached — the
   runtime or the programmer? Is a detached task still cancellable, or
   is cancellation surrendered along with the result?
3. **Observability.** Can the program observe completion of a detached
   task (completion signal, counter), or is it fully invisible? Does
   process exit wait for detached workers, or may they be abandoned at
   shutdown?
4. **Interaction with `Send`/`Move` (OQ5).** Detached work is by
   definition not materialised — does its captured data still need to
   satisfy `Send` (`spawn`) / `Move` (`fork`)? Likely yes: the data
   crosses the thread/process boundary regardless of whether the result
   returns.
5. **Language or StdLib?** Does detach require compiler/runtime support
   (cancellation bookkeeping, exit-time worker tracking), or can it be
   a library function? The generator protocol (`next()`/`stop()`) is
   already classified as language-level in `EXECUTION_CONTEXT_INVOCATION.md`
   § Context Constructors.
6. **Relationship to the generator protocol (OQ4).** Is detach a third
   operation on the generator (`next()` / `stop()` / `detach()`), or a
   distinct mechanism on the submission handle?
7. **Syntax (deferred to Phase 5).** If detach exists, what is its
   canonical form — named function (`detach(handle)`), method
   (`handle.detach()`), or operator?

## Implications for Orthon

- A detach mechanism restores fire-and-forget without weakening the OQ8
  guard: the guard applies to *undetached* work; detached work is
  deliberately released.
- Detach must not reintroduce silent behaviour: abandoning a task must
  be a visible, explicit act (consistent with Semantic Invariant 6 and
  the Explicitness principle).
- The interaction between detach and worker shutdown policy
  (`DEFAULT_STRATEGY`) must be specified: does a detached worker inherit
  the pool's shutdown policy, or is it independent?
- [`what/GLOSSARY.md`](../../../../what/GLOSSARY.md) would need a
  `Detach` / `Fire-and-forget` entry.

## Decision History

Initial hypothesis derived from the OQ8 Resolution of
`EXECUTION_CONTEXT_INVOCATION.md` (2026-07-31) — no decisions recorded
yet.

## Affected Documents

- [ ] [`essential/EXECUTION_CONTEXT_INVOCATION.md`](../essential/EXECUTION_CONTEXT_INVOCATION.md) — OQ8 consequence (fire-and-forget not expressible)
- [ ] [`what/concepts/CONCURRENCY_MODEL.md`](../../../../what/concepts/CONCURRENCY_MODEL.md) — accepted concurrency model (EDR-033); generator protocol, worker lifecycle
- [ ] [`what/GLOSSARY.md`](../../../../what/GLOSSARY.md) — add `Detach` / `Fire-and-forget`
- [ ] [`what/PRIMITIVE_BLOCKS.md`](../../../../what/PRIMITIVE_BLOCKS.md) — generator protocol primitives
