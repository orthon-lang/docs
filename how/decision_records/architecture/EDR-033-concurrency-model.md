# EDR-033: Delegate-Based Concurrency Model

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Language

---

### Context

Phase 4 must decide Orthon's concurrency model. The research document [`CONCURRENCY_MODEL.md`](../../how/concepts/research/essential/CONCURRENCY_MODEL.md) proposes a delegate-based model with message passing and no shared-state threads.

Several concurrent decisions constrain this choice:

- **ERROR_HANDLING (EDR-020):** Defines `Result<T,E>` as the universal error model. Error propagation across delegate boundaries must use this model.
- **TRAITS (EDR-019):** Defines nominal trait system. Delegates must support trait dispatch for polymorphic message passing.
- **OWNERSHIP model:** The `$` ownership transfer operator and `mut` keyword (Phase 3 D-10) define how data crosses isolation boundaries.
- **CONCURRENCY (Plan 04-06, important tier):** Will define StdLib concurrency utilities (channels, timers, async I/O) built on this model.

The key design tension: the model must be **implementation-independent** (not tied to a specific threading or async runtime) while providing strong safety guarantees (no data races, no shared mutable state across delegates). The model defines what concurrent execution means; the Implementation Strategy defines how it is realised.

### Decision

Adopt a **delegate-based concurrency model** with message passing:

1. **`act` modifier** — Types declared with `act` represent concurrent execution contexts. Instances have isolated state accessed via message passing.
2. **`delegate` keyword** — Creates a concurrent delegate instance. The runtime creates an isolated execution context with its own state.
3. **`<-` message operator** — Sends a message to a delegate. Asynchronous by default for void-returning messages; returns a future for value-returning messages.
4. **No shared-state threads** — Orthon has no `thread` keyword, no `Mutex<T>`, no `RwLock<T>`. All concurrency is through delegates and message passing.
5. **Explicit ownership transfer** — Data crossing delegate boundaries uses explicit `$` ownership transfer. No implicit shared access.
6. **Error propagation via Result** — Message processing errors use the `Result<T,E>` model (EDR-020). Delegates can return errors in messages.
7. **Trait dispatch on delegates** — Delegates implement traits; trait methods can be invoked on delegate references.
8. **Implementation-independent** — The model defines semantics (isolation, message ordering, ownership transfer) without referencing threads, fibers, or async runtimes.

### Relationship to CONCURRENCY (Plan 04-06)

This EDR defines the **language-level concurrency model**. Plan 04-06 will define **StdLib concurrency utilities** (channels, timers, async I/O, structured concurrency) built on top of this model. The language provides the `act`/`delegate`/`<-` foundation; the StdLib provides the ergonomic utilities.

### Relationship to ERROR_HANDLING (EDR-020)

Message processing errors propagate via `Result<T, DelegateError>`. The `?` operator works across delegate boundaries — a failed message send returns an error that can be propagated.

### Relationship to TRAITS (EDR-019)

Delegates implement traits like any other type. Trait method dispatch on delegates works through the `<-` operator, enabling polymorphic message passing.

### Consequences

**Positive:**
- No data races by construction — isolation is structural, not convention-based.
- No deadlocks — no explicit locks means no lock-ordering bugs.
- Automatic parallelism — independent delegates execute on different cores without programmer annotation.
- IMPLEMENTATION_INDEPENDENCE_GATE passed — the model is defined in terms of isolation and message passing, not threads or runtimes.
- Clear upgrade path to Plan 04-06 (StdLib concurrency utilities).

**Negative:**
- Message-passing overhead compared to shared-memory concurrency.
- No shared-state parallelism for performance-critical data structures (by design — Orthon does not support this pattern).
- `act` modifier and `<-` operator add language surface.
- Ownership transfer (`$`) is required for cross-delegate data — may be unfamiliar to programmers from other languages.

### Compliance

1. Every `delegate` instance must have isolated state — no two delegates share mutable memory.
2. Message processing must be single-threaded per delegate — at most one message processed at a time.
3. Cross-delegate data access must use explicit `$` ownership transfer — no implicit shared references.
4. Error propagation across delegate boundaries must use `Result<T,E>`.
5. Trait dispatch on delegates must work through the `<-` message operator.
6. The semantic specification must not reference threads, fibers, async keywords, or runtime-specific concepts.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| **Shared-memory threads + locks** | Violates "no shared mutable state" principle. Data races, deadlocks, and global reasoning requirements are the problems Orthon is designed to avoid. |
| **Async/await model (Rust/C#/JS)** | Function-coloring problem (sync vs. async functions). Requires runtime to define async executor. Implementation-dependent — tied to a specific async runtime model. |
| **CSP model (Go channels)** | Channels are a StdLib concern, not a language model. Go's goroutines are implementation-specific (M:N scheduling). Orthon's model must be implementation-independent. |
| **Software Transactional Memory** | Still allows shared mutable state (via transactions). Runtime overhead of conflict detection. Less proven than actor/delegate model. |
| **Pure actor model (Erlang)** | Proven model. Orthon adopts actor-like internals behind `delegate` keyword but diverges by not exposing actor primitives (no `spawn`, no `receive` pattern matching on messages). |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Concurrency without data races or deadlocks is a direct user need. Delegate model is familiar from actor frameworks. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Isolation guarantee follows from ownership rules. No shared mutable state is already a core principle — this extends it to concurrent execution. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | One model (delegate + message passing) replaces threads, locks, condition variables, atomics, and async/await. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | `act`/`delegate`/`<-` operate in the Core Language layer. StdLib concurrency (Plan 04-06) builds on top. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | **Critical — Pass** | TRIZ | The model is defined purely in terms of isolation, message passing, and ownership transfer. No reference to threads, fibers, async runtimes, or scheduling strategies. Every proposed Implementation Strategy (work-stealing, pinned-to-thread, single-threaded) can realise the semantics. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Evolution path: Plan 04-06 adds StdLib utilities. No shared-state threads means no future compatibility burden from thread-safety guarantees. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `act` modifier, `delegate` keyword, and `<-` operator are explicit and locally visible. The model follows a well-known pattern (actor model) that LLMs generate reliably. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-033 for per-gate reasoning trail.
