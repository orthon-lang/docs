# EDR-049: Concurrency — StdLib Utilities Built on Delegate Model

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem

---

### Context

Orthon's concurrency model was settled in [EDR-033](./EDR-033-concurrency-model.md): delegate-based concurrency with `act` modifier, `delegate` keyword, `<-` message operator, and ownership transfer rules. The CONCURRENCY_MODEL defines the **language-level concurrency model** — what the compiler provides.

The research document at `how/concepts/research/important/CONCURRENCY.md` proposes the actor model with typed channels, select statements, supervision trees, and structured concurrency patterns. However, the `DELEGATE.md` analysis (2026-07-26) superseded the actor model as a language feature — delegated execution (`delegate`) replaces `actor`/`act` as the language primitive.

The question for this decision: **does CONCURRENCY add new semantics beyond the delegate model, or is it StdLib patterns built on top?**

The research document describes:
- Channels for message passing between delegates
- `select` statements for waiting on multiple channels
- Supervision trees for failure recovery
- Timers, async I/O wrappers
- Structured concurrency patterns (fan-out, fan-in, pipeline)

All of these are implementable as Standard Library utilities using the delegate model's primitives (`delegate`, `<-`, `$` ownership transfer). The channel model matches CSP-style concurrency (Go-style) built on top of the delegate runtime.

The Decision Pipeline classified CONCURRENCY as **StdLib**: Concrete async/concurrent patterns building on CONCURRENCY_MODEL (EDR-033) without adding new language semantics.

---

### Decision

Adopt the following StdLib concurrency utilities built on the delegate model:

1. **Typed channels** — `Channel<T>` for message passing between delegates. `send(value)` and `receive()` methods wrapping delegate message queues. Buffered and unbuffered variants.

    ```orthon
    let ch = Channel<String>(buffer: 10)
    delegate producer:
        ch.send("hello")
    delegate consumer:
        let msg = ch.receive()
    ```

2. **`select` expression** — Wait on multiple channels simultaneously. The first ready channel triggers the corresponding branch. `select` is a StdLib function/macro, not a language construct.

    ```orthon
    select:
        case msg = ch1.receive():
            handle(msg)
        case msg = ch2.receive():
            handle(msg)
        case default(timeout: 1s):
            handle_timeout()
    ```

3. **Fan-out / fan-in patterns** — StdLib functions distributing work across worker delegates and collecting results. Built on `delegate` + `Channel`.

4. **Supervision** — Delegate lifecycle management with optional restart policies (one-for-one, one-for-all, rest-for-one). Implemented as a StdLib supervisor delegate, not a language primitive.

5. **Timers** — `Timer` and `Ticker` delegates producing events on a schedule. StdLib, not language.

6. **Async I/O wrappers** — `AsyncReader`, `AsyncWriter` wrapping blocking I/O in delegates. StdLib, not language.

7. **Structured concurrency utilities** — `with_timeout`, `race` (first completed wins), `all` (all must complete). StdLib function composition, not new syntax.

8. **Backpressure support** — Channel capacity limits with optional blocking or dropping strategies (bounded channel semantics). StdLib configuration, not language.

**Key principle:** All concurrency utilities are implementable as Ordinary Orthon code using the delegate model. No new compiler-level constructs required.

---

### Consequences

**Positive:**
- Zero language additions — all utilities are StdLib, preserving Minimal Core.
- Delegate model provides the foundation; StdLib utilities are additive and optional.
- Users who prefer bare delegate messaging can ignore channels entirely.
- `select` as a macro (not language construct) keeps the core smaller.
- Supervision policies are configurable at the library level, not baked into types.
- Timers and async I/O wrappers are natural extensions of the delegate runtime.

**Negative:**
- Channels as StdLib may be less optimised than a built-in channel type (no special compiler knowledge).
- `select` as a macro has limitations compared to a language-level `select` (no custom syntax for timeouts).
- Supervision policies must be implemented manually or via a library rather than enforced by the compiler.
- Backpressure is a runtime consideration, not a type-system guarantee.

---

### Compliance

1. All concurrency utilities must be implementable using only `delegate`, `<-`, `$` ownership transfer, and existing language constructs.
2. No new keywords, syntax, or compiler changes are required for any StdLib concurrency utility.
3. Channels must be typed (`Channel<T>`) with compile-time type checking via generics.
4. Utilities must be strategy-aware — they must work across different delegate implementations (single-threaded, multi-threaded, embedded).
5. The delegate model's invariants (no shared mutable state, single-threaded per delegate) must be preserved by all utilities.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Language-level channels | Would add channel syntax to the compiler. Channels are fully implementable as StdLib types wrapping delegate mailboxes. |
| Language-level `select` | `select` is syntactic sugar over delegate message polling. A StdLib macro or function provides equivalent ergonomics. |
| Built-in supervision trees | Supervision is a runtime/library pattern, not a language concern. Different applications need different supervision policies. |
| Actor syntax (`actor` keyword) | Superseded by `delegate` as the language primitive (EDR-033). Actor semantics are a runtime implementation detail. |
| Go-style goroutines and channels | Goroutines require a language-level scheduler and channel type. Orthon's delegate model already provides the scheduling; channels are StdLib. |

### Gate Validation

Gates required per `DECISION_VALIDATION.md` § Gate Selection (Standard Library addition): `USER_VALUE_GATE`, `CONCEPTUAL_SIMPLICITY_GATE`, `LONG_TERM_MAINTAINABILITY_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I need to coordinate work across multiple concurrent delegates." Channels, select, and supervision are proven patterns (Go, Erlang, Elixir). Serves VISION.md's Architectural Integrity pillar — utilities built on the delegate model without extending the language. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Hypothesis: "All concurrency utilities are expressible as StdLib on top of the delegate model." Verification: channels wrap delegate mailboxes; select polls multiple channels; supervision is a delegate that monitors other delegates; timers are delegates on a scheduler; async I/O wraps blocking I/O in delegates. All are ordinary Orthon code. |
| `LONG_TERM_MAINTAINABILITY_GATE` | [Einstein's Method](../../gates/methods/EINSTEIN_METHOD.md) | Pass | One-sentence test: "Channels let delegates communicate; select lets them wait on multiple sources." StdLib classification means channels can evolve independently of the language. No conceptual debt — channels, select, and supervision are well-understood patterns with decades of production use. |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Channels are consistent with the delegate model — a channel is essentially a delegate that holds a buffer. `select` is consistent with pattern matching semantics. All terms are precisely defined. |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | All utilities live in the Standard Library layer, depending on the delegate model from the Core Language layer. No layer violations. |

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — StdLib additions do not require implementation independence (they are implementations). `LLM_GENERABILITY_GATE` — StdLib functions have standard LLM generability properties.

**Detailed reasoning:** See `DECISION_LOG.md` entry (2026-07-27) for per-gate reasoning trail.
