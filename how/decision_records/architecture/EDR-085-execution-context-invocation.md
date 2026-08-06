# EDR-085: Execution Context Invocation — Unified Invocation Model

**Status:** Accepted

**Date:** 2026-08-06

**Category:** Architecture

**Scope:** Subsystem (Invocation & Execution Semantics)

---

### Context

How does Orthon invoke a computation? Today the language carries three
separate invocation mechanisms with distinct object models and execution
semantics:

| Mechanism | Syntax | Semantics |
|-----------|--------|-----------|
| `call` | `fn(args)` | Immediate execution |
| `delegate` send | `obj <- msg(args)` | Asynchronous message via mailbox |
| `spawn` | `spawn fn(args)` | Parallel execution |

This is compounded by `async`/`await` (EDR-047), which introduces coloured
functions — a function's declaration is tied to how it may be invoked, and
the programmer must decide *how* to represent computation before writing
*what* the computation is. Each mechanism (call / delegate send / spawn /
async) brings its own object model, breaking orthogonality and local
reasoning.

Orthon targets *both* humans and LLMs (VISION.md § LLM Readiness; the LLM
Generability Gate). A unified invocation model — one operation submitted to
an Execution Context — removes the colouring, unifies resource management,
and keeps a single orthogonal axis: **what** to invoke vs. **how** to
execute it.

The Decision Pipeline returned **ACCEPT** as a Language (Core) decision. The
proposal adds `execution_context` as the **10th primitive** (the first
extension of the EDR-016 primitive set), absorbs CONCURRENCY_MODEL (EDR-033)
into `defer`/`delegate`/`spawn`/`fork` contexts, and eliminates async/await
(EDR-047). This **retroactively modifies accepted concepts** — handled as a
documented cross-concept amendment (C-003), not silent drift.

**Research & pipeline trail:** [`EXECUTION_CONTEXT_INVOCATION.md`](../../concepts/research/essential/EXECUTION_CONTEXT_INVOCATION.md)
(concept research; OQ2–OQ8 resolved via Concept Design Review Convergence
Check; B1–B6 resolved 2026-08-06);
[`DECISION_LOG.md`](../../gates/DECISION_LOG.md) § Entry:
EXECUTION_CONTEXT_INVOCATION (per-gate reasoning).

---

### Decision

1. **One Invocation, two axes.** There is no function call vs. delegate
   send vs. parallel spawn vs. async/await. There is one operation —
   **Invocation** — with two independent axes: *what to invoke* (function /
   method) and *how to execute* (the Execution Context). Functions are
   **colourless**: the same function can be invoked immediately, deferred,
   or distributed without modification.

2. **`call` is the immediate invocation primitive — unchanged in scope.**
   `fn(args)` executes now, in the current execution environment, with no
   context and no operator. `call` is **not** parameterised by context. It
   answers only *what* to invoke; it never decides *how*.

3. **`execution_context` added as the 10th primitive.** *“Environment that
   executes an invocation according to a policy. Created by a constructor
   (`defer(obj)` / `delegate(obj)` / `spawn()` / `fork()`); owns zero or one
   state object; deterministic destruction (owned value drops per
   Ownership/Lifetime); pending stateless work on `spawn`/`fork` must be
   resolved explicitly (`next()`/`stop()`) before destruction.”* It answers
   only *how* to execute. The primitive set is now **10** (EDR-016 amended);
   minimality re-proved for the 10-element set (B2).

4. **Invocation in context** submits an invocation to an Execution Context
   via a **two-operator family** that encodes the ownership relationship:
   - `ctx <- fn(args)` — submit to the context's **single owned state**
     (`delegate(obj)`, `defer(obj)`), serialised, in order.
   - distribution operator (`ctx |> fn(args)`, glyph provisional) — submit
     to **stateless workers** (`spawn()`, `fork()`), independent, parallel.
   The operator set is **closed (2)**; the constructor set is **open** — a
   new context type resolves to one of the two by whether it owns state.

5. **Context constructors and materialisation vocabulary:**
   | Context | Constructor | Extraction |
   |---------|-------------|------------|
   | Immediate | — | `fn(args)` |
   | Coroutine | `defer(obj)` | `await(ctx)` |
   | Actor (mailbox) | `delegate(obj)` | `take(ctx)` |
   | Thread (shared memory) | `spawn()` | `next()` / `stop()`, `grab`/`gather` |
   | Process (isolated memory) | `fork()` | `next()` / `stop()`, `grab`/`gather` |
   `await`/`take` are context-specific named functions (compiler-intrinsic
   support permitted); `grab`/`gather` are StdLib sugar over the generator
   protocol. `delegate(obj)`/`defer(obj)` always wrap an object — bare
   contexts do not exist.

6. **Submission return is context-defined (OQ2, Variant C).** `delegate`
   returns `void` (a message; owner state read via `take`); `defer` returns
   a **deferred invocation** (a suspendable computation); `spawn`/`fork`
   return `void` and each submission joins the generator stream. The `defer`
   transport type name (Task/Future/Deferred) is a Phase 5 detail; the Core
   commits to the concept, not the name.

7. **`Send` / `Move` marker traits (OQ5, Variant B).** Compile-time
   auto-traits, Rust-style: `spawn` requires captured data be `Send` (safe
   across threads); `fork` requires `Move` = `Send` + serialisable (across
   processes). Errors surface at compile time. Stateless functions are
   trivially `Send`/`Move`.

8. **Fixed inline call semantics (OQ6).** `fn(args)` inside any context is
   context-independent — always a synchronous, inline call; it never blocks
   or yields by itself (the callee may suspend at an explicit extraction
   point). No self-delegation; long computations are submitted cross-context
   via the distribution operator.

9. **Context destructor contract (OQ8).** The wrapped object drops
   automatically as a consequence of ownership + lifetime (not a new rule);
   `take(ctx)` moves it out; `using` remains pure sugar. For `spawn`/`fork`,
   no safe automatic behaviour for pending work exists — explicit resolution
   (`next()`/`gather()`, or cancel via `stop()`) is required before
   destruction; destroying a context with unresolved work is a program
   error. Worker-shutdown manner and enforcement are Implementation
   Strategy concerns (Phase 7).

10. **`using` is syntactic sugar** over context + scope + deterministic
    destructor (`using x = open("f.txt")` desugars to
    `delegate(open("f.txt"))` + block + scope-bound destruction). It
    introduces no new semantics; `SCOPED_RESOURCE_LIFECYCLE.md` is
    superseded. Resource management, `async`/`await`, `spawn`, and
    `delegate send` are **not** separate features — they are the same
    Invocation pattern.

11. **Superseded:** CONCURRENCY_MODEL (EDR-033) and ASYNC_AWAIT (EDR-047)
    are superseded by this model. DELEGATE is rewritten as a context
    constructor. `EXECUTION_MODEL`, `SYNTAX`, `GLOSSARY`, `CORE_CONCEPTS`,
    `DESIGN_PRINCIPLES` (§ Uniformity), and `DECLARATIVE_CONSTRUCTS`
    (§ Resource Management) are updated per C-003.

---

### Consequences

- **Positive:**
  - One orthogonal axis (what / how) replaces three mechanisms plus
    async/await — removes coloured functions entirely.
  - Colourless functions: any function composes with any context.
  - Extensible: a new context type (GPU, cluster) needs only a new
    constructor — the operator set stays closed at two, so the syntax
    surface does not grow.
  - `using` unifies resource management with the invocation pattern;
    deterministic destruction per the Lifetime dimension.
  - `execution_context` as a first-class primitive honestly reflects that
    execution policy is irreducible — it cannot hide inside `call`.
- **Negative:**
  - Two-operator family is a documented trade-off (ownership axis): the
    reader still needs the context type for the exact policy within each
    operator (yield vs. block vs. dispatch) — mitigated by proximity,
    naming, and the compile-time operator/context pairing (OQ7).
  - Context boilerplate before non-immediate calls; one-shot-context sugar
    may be needed.
  - Four constructors grow the syntax surface (they remain Level 2
    patterns, not primitives).
  - Retroactive amendment of accepted concepts (C-003) and a documentation
    cost across examples.
  - Fire-and-forget is not expressible without `detach()` — deferred to a
    separate research hypothesis (DETACHED_EXECUTION.md).

---

### Compliance

- `execution_context` is the 10th primitive in `PRIMITIVE_BLOCKS.md`;
  `call` is documented as immediate invocation, not context-parameterised.
- `fn(args)` = `call(fn, args)` (no context); `ctx <- fn(args)` and
  `ctx |> fn(args)` = invocation in context (uses both primitives).
- The operator set is closed (2); the constructor set is open.
- `defer`/`delegate` always wrap an object; `spawn`/`fork` are stateless
  worker contexts.
- `Send`/`Move` marker traits are checked at compile time for `spawn`/`fork`.
- Destroying a `spawn`/`fork` context with unresolved work is a program error.
- `using` desugars to context + scope + destructor (no new semantics).
- C-003 amendments applied: PRIMITIVE_BLOCKS (10 primitives), CONCURRENCY_MODEL
  (EDR-033) and ASYNC_AWAIT (EDR-047) superseded, DELEGATE rewritten,
  EXECUTION_MODEL/SYNTAX/GLOSSARY/CORE_CONCEPTS/DESIGN_PRINCIPLES/
  DECLARATIVE_CONSTRUCTS updated, SCOPED_RESOURCE_LIFECYCLE superseded.
- Distribution operator glyph deferred to Phase 5 (Syntax Design).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| **Variant B — `call` parameterised by a hidden Execution Context (no new primitive)** | Hides an irreducible entity inside `call`, violating its single responsibility (what vs. how); the compiler-level semantics (state machine, mailbox, thread/process) cannot be composed from the 9 primitives, so the cost leaks into `call`'s definition instead of being declared. |
| **Per-context operators (`<~`, `<-`, `<=`, `<@`, `<#`)** | Operator proliferation — one glyph per policy; violates "one concept → one mechanism" and grows the syntax surface per new context. Rejected in favour of a closed two-operator family (OQ5). |
| **`.` for contextual invocation** | Conflicts with immediate method call — the reader cannot tell from syntax whether `x.foo()` blocks; loss of local reasoning. Rejected in favour of distinct `<-` / distribution operators. |
| **Symmetric extraction operators (`->`, reverse `<-`)** | `->` already means return type; reverse `<-` would give the same glyph opposite dataflow depending on operand types. Rejected in favour of context-specific named functions (`await`, `take`, `next`/`stop`). |
| **`<=` as the distribution glyph** | `<=` is already less-than-or-equal comparison; violates "one symbol → one meaning". Candidates `<||` (family-coherent) / `|>` (maximally distinct); glyph deferred to Phase 5. |
| **Dedicated `resource(obj)` constructor** | Every execution context already has a lifecycle; `delegate(open("f.txt"))` is already a resource-managing context. No new constructor type needed; `using` is the concise form. |
| **RAII only, no `using`** | Pure RAII fails for shared ownership (resource outliving the creating scope) and conditional cleanup; `using` is the declarative, scope-anchored form. |
| **Keep async/await (EDR-047)** | Coloured functions, syntax virus, ecosystem split — directly contradicts the orthogonal Invocation model and the colourless-functions property. Superseded. |

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Concrete benefit: removes coloured functions, unifies resource management, schema-visible execution context. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | OQ2–OQ8 remove contradictions; two operators = one axis (ownership); `.` vs `<-` syntactically distinct. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass (Flag) | 10th primitive justified (irreducible); flag: 4 constructors grow syntax surface (Level 2 patterns, not primitives). |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass (Flag) | `execution_context` = Level 1; operators/constructors = Level 2; materialisation = StdLib. Flag: retroactive amendment (C-003), applied at acceptance. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Context semantics strategy-agnostic; mechanisms (thread pool, cooperative scheduler, mailbox) are Strategy concerns (Phase 7). |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass (Flag) | Extensible (new constructors, no new operators); flag: two-operator family is a documented trade-off, pre-freeze reversible. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass (Flag) | Named constructors + arrow operators are LLM-friendly; flag: distribution glyph not finalised (Phase 5). |

**Gates not applied:** none — a new Core Language semantic (10th primitive)
requires the full catalogue of 7 gates (`DECISION_VALIDATION.md` § Gate
Selection).

**Detailed reasoning:** See `DECISION_LOG.md` § Entry:
EXECUTION_CONTEXT_INVOCATION for the per-gate reasoning trail and the B1–B6
resolution notes.
