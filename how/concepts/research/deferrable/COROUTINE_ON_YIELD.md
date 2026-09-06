# Coroutine on Yield — Caller-Driven Bidirectional Value Exchange (Hypothesis)

> **⚠️ HYPOTHESIS — not part of the Orthon v0.1 language model.**
>
> **Status:** Hypothesis (research). Not accepted.
>
> **Provenance:** Formerly accepted as *bidirectional `yield`* inside the
> GENERATORS concept ([`what/concepts/GENERATORS.md`](../../../../what/concepts/GENERATORS.md),
> [EDR-050](../../../../how/decision_records/architecture/EDR-050-generators.md)).
> **Withdrawn from the language model 2026-09-06:** generators are emit-only
> (one-way). The capability was demoted here as a candidate future concept.
> Formal EDR: [EDR-091](../../../decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md).
>
> **See also:** [`LAZY_SEQUENCE_GENERATORS.md`](../../../../what/concepts/LAZY_SEQUENCE_GENERATORS.md)
> (EDR-021 — `emit`, one-way), [`ITERATOR_PROTOCOL.md`](../../../../what/concepts/ITERATOR_PROTOCOL.md)
> (EDR-022 — consumption), [`EXECUTION_CONTEXT_INVOCATION.md`](../essential/EXECUTION_CONTEXT_INVOCATION.md)
> (EDR-085 — `defer`, async coroutine).

---

## Problem

Some producer-consumer patterns are not one-way: the consumer needs to feed
context or configuration back to the producer *between* values, and each
produced value depends on the consumer's previous response. Examples:
interactive coroutines, two-party negotiation protocols, REPL-style
sessions, state machines driven round by round.

The one-way `emit` model (EDR-021) cannot express this. The question this
hypothesis records: **is there a language construct for a synchronous,
caller-driven, resumable computation that exchanges one value out and one
value in at every suspension — and is it worth having in Orthon?**

## Framing: this is not a generator

A pure producer is a **semicoroutine** (generator / iterator): control and
data flow in one direction, and the producer never receives a value at a
resume point. A construct where every suspension both produces a value and
receives a value is, in the PLT sense, a **coroutine** (asymmetric, full
duplex at the boundary). Generators are the one-way special case of
coroutines, not vice versa.

Consequence: calling the two-way form a *generator* (or a generator
feature) was a **category error**. It is a **synchronous yield coroutine** —
the sequence-world sibling of the async `defer` coroutine in the
invocation world (EDR-085).

## Why `defer` cannot host it

The `defer` coroutine context (EDR-085) looks adjacent but is a different
control shape:

- `defer` is **scheduler-driven**: a submitted invocation runs, and when it
  hits `await(ctx)` it suspends and control returns to the runtime; it is
  resumed when the awaited result is ready.
- There is **no caller-injected resume**: OQ4 of EDR-085 removed `yield`
  from the concept ("runtime yields on `await` automatically"); OQ6 forbids
  self-delegation / resumption-with-a-value. A submitted `defer`
  computation is not resumable with a value injected by its caller.
- A `defer` context "owns one suspendable computation" and yields one
  result per submission — it is not a pull-based exchange point.

A synchronous yield coroutine is **caller-driven pull**: the caller calls
`next()`/`send()` synchronously, and the producer's `yield` expression
*is* the value the caller injected. This primitive does not exist in the
EDR-085 model.

## Comparison: synchronous yield coroutine vs. async `defer` coroutine

| Axis | Synchronous yield coroutine (hypothesis) | Async `defer` coroutine (EDR-085) |
|------|------------------------------------------|-----------------------------------|
| Who drives | Caller (pull) | Runtime / scheduler |
| Suspension trigger | Producer hits `yield` | Function hits `await(ctx)` |
| Resumption trigger | Caller calls `next()` / `send()` | Awaited result is ready (event loop / executor) |
| Value out | Yielded value | `await(ctx)` result of the submitted computation |
| Value in | `send()` argument becomes the value of the `yield` expression | Arguments of the submitted invocation |
| Rendezvous per step | One `yield` = one value out + one value in (lockstep) | Submit → one final result; no per-step rendezvous |
| Shape | Sequence-shaped (`Iterator`-like, pull) | Invocation-shaped (submit / await) |
| Concurrency | None needed (single-threaded alternation) | Enables overlapping I/O and cooperative scheduling |
| State preservation | Compiler state machine across yields | Compiler state machine across awaits |
| Home in Orthon | Sequence layer (absent) | Execution-context layer (accepted, colourless) |

The decisive divergence: the two constructs suspend for different reasons
and are resumed by different actors. `defer` suspends to *wait for the
world*; a synchronous yield coroutine suspends to *exchange a value with
its caller*. Neither can stand in for the other.

## Why it was withdrawn (2026-09-06)

1. **Category error in the generator concept** — producers do not consume;
   a two-way exchanger is a coroutine, not a generator.
2. **Not expressible via `defer`** in its ergonomic form (see above).
3. **Composition suffices for the synchronous case** — an interactive
   session is expressible as an explicit stateful object with one method
   per round (`type` + `fun` + `match` on an explicit stage field). This is
   more explicit and arguably more LLM-generable than a hidden
   compiler-built state machine, and adds no language surface.
4. **Minimal core** — keep one production keyword (`emit`), one concept
   (generator = producer). Two keywords for closely related concepts
   (already flagged as a negative in EDR-050) is avoided.

## How other languages name the two-way construct

| Language | One-way producer | Two-way construct | Strategy |
|----------|------------------|-------------------|----------|
| Python | `generator` (`next()`) | same object via `send()`; historically "coroutine" (pre-`async`) | Conflate |
| JavaScript | `generator function*` | same object via `.next(value)` | Conflate |
| Lua | no distinct name | **`coroutine`** — `resume(v)` / `yield(v)` | Call it a coroutine |
| Ruby | **`Enumerator`** | **`Fiber`** — `resume(arg)` ↔ `Fiber.yield(v)` | Distinct names |
| C# | iterator (`yield return`) | none — `Channel` / async streams | Externalise to channel |
| Kotlin | `Sequence` / `Flow` | `Channel` | Externalise to channel |
| Go | goroutine + channel | goroutine + channel | Externalise to channel |

No mainstream language gives the two-way form a distinct word other than
*coroutine* (or Ruby's *Fiber*). The alternative strategy — keep the
producer one-way and move two-way communication to a channel/actor — is
how C#, Kotlin, and Go resolve the same need.

## Implications for Orthon (if revisited)

- If ever reintroduced, it is a **new construct** (a synchronous coroutine),
  not a generator extension — it needs its own keyword and trait story.
- Its only durable niche is a *synchronous, single-threaded, lockstep
  dialogue* where the producer's straight-line body is more readable than
  explicit state. If a round ever needs to suspend on I/O, the pattern
  crosses into the `defer` world (where two-way communication is a
  channel/actor concern) — which may eliminate the need entirely.
- Explicit-stateful-rounds composition (the current recommendation) should
  be the documented pattern; revisit this hypothesis only if that pattern
  proves painful in practice.

## Open Questions

1. Is there a real v0.2+ use case that explicit stateful rounds cannot
   express cleanly, justifying a new synchronous-coroutine construct?
2. If interactive rounds need I/O suspension, does the need collapse into
   `defer` + channel/actor, making a separate sync coroutine unnecessary?
3. Naming: the word *coroutine* is already associated with `defer` in
   Orthon (EDR-085). If this construct is ever accepted, what distinguishes
   its name?
4. LLM generability: are explicit-stage rounds genuinely easier for an LLM
   to produce correctly than a compiler-managed resumable body?
5. Ownership: if values are sent into a suspended producer, do they move or
   borrow (the former EDR-050 open question, still unresolved)?

## Decision History

- **2026-07-27** — Accepted as *bidirectional `yield`* (EDR-050), part of
  the GENERATORS concept.
- **2026-09-06** — Withdrawn from the language model: generators are
  emit-only; the bidirectional capability is demoted to this hypothesis.
  Formal EDR: [EDR-091](../../../decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md).
