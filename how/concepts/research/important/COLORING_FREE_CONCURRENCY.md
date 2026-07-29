# Hypothesis: Function-Coloring-Free Concurrency (Go/BAML Model)

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created as exploratory research (Milestone 1, BAML gap analysis)
> examining whether Orthon should drop the `async` modifier in favour of
> Go/BAML-style coloring-free concurrency. A concept is registered only
> after acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-29
>
> **Source:** Derived from BAML Concepts — Orthon Gap Analysis §3
> (`notes/baml-concepts-orthon-gap-analysis.md`).

---

## Issue (Why)

In "colored" async models (Python, JavaScript, Rust, C#), the `async`
modifier splits the function universe into two disjoint sets: `async`
functions and synchronous functions. This creates a **function coloring
problem** (Nystrom, 2015) with three concrete costs:

1. **Viral propagation.** Calling an `async` function from a sync
   function requires either rewriting the caller as `async` (which
   propagates upward) or using a blocking bridge (which risks deadlocks).
2. **Dual APIs.** Libraries ship two versions of the same function —
   `read()` and `read_async()`, `fetch()` and `fetch_async()`.
3. **LLM generation cost.** An LLM generating Orthon code must maintain a
   mental model of which functions are `async` and which are not. A
   single missing `await` or misplaced `async` produces a compilation
   error. The coloring split doubles the combinatorial space the LLM
   must navigate.

Go and BAML eliminate coloring: **any function** can be called with
`spawn` and awaited. The compiler/runtime decides whether the call is
concurrent; the programmer describes intent (`spawn`), not mechanism
(`async`).

### Why This Matters for Orthon

Orthon already has `ASYNC_AWAIT` (important-tier research) with `async`
as an explicit modifier, and `CONCURRENCY_MODEL` (ACCEPTED, EDR-033) with
delegate-based message passing. The question is whether Orthon should
follow BAML's lead and remove function coloring for LLM-ergonomic
reasons, or retain explicitness as a design principle.

---

## Relationship to Existing Concepts

| Concept | Status | Relation |
|---|---|---|
| **ASYNC_AWAIT** | Important-tier research | Defines `async` modifier + stackless coroutines. The `async` keyword is the point of contention |
| **ASYNC_AS_EXPLICIT_MODIFIER** | Important-tier research | Strengthened hypothesis: `async` = coroutine, `exclusive` = access serialization, `spawn` = explicit parallelism, `Future<T>` as first-class value |
| **CONCURRENCY_MODEL** | ACCEPTED (EDR-033) | Delegate-based, `act` modifier, ownership transfer. Independent of coloring — delegates already abstract over execution strategy |
| **DELEGATE** | Essential-tier research | `delegate(owner)` + `<-` operator. Separates "what runs concurrently" from "how it runs" |
| **SPAWN_OPERATOR** | Deferrable-tier research | Explicit task creation — orthogonal to `async` |

---

## Proposal

**Drop the `async` modifier from Orthon.** Any `fun` or `proc` can be
spawned:

```orthon
fun work(i: Int) -> Int { i * i }        // plain function

fun main() -> Int
    let a = spawn work(1)                // any function, no 'async' needed
    let b = spawn work(2)
    (await a) + (await b)
```

- `spawn` creates a concurrent task; the runtime manages suspension.
- `await` extracts the result.
- No function-level coloring — `work` is callable synchronously (`work(5)`)
  or concurrently (`spawn work(5)`) without modification.

The compiler infers suspendibility from call-graph analysis (do any
callees perform I/O or spawn?) rather than from an explicit modifier.

---

## Tradeoffs

| Dimension | Keep `async` (current) | Drop `async` (proposed) |
|---|---|---|
| **Explicitness** | Suspension points are syntactically visible (`async fn`) — aligns with DESIGN_PRINCIPLES "Explicitness" | Suspension is invisible — the caller cannot tell from the signature whether `work(5)` may block |
| **LLM generability** | LLM must track coloring — higher error rate on missing `await`/`async` | Simpler generation — no coloring to track |
| **Learnability** | Two categories of functions to teach | One category — conceptually simpler |
| **Tooling cost** | Low — compiler enforces coloring, errors are local | Medium — call-graph analysis needed to surface "this call may block" warnings |
| **Ecosystem** | Familiar to Python/JS/Rust/C# developers | Familiar to Go developers |
| **Refactoring safety** | Adding I/O to a function forces signature change (`async`) — compiler catches all callers | Adding I/O is silent — callers may unexpectedly block without any compiler feedback |
| **Runtime overhead** | Stackless coroutines — state machine allocation only at `async` boundaries | Every function potentially a coroutine — either universal overhead or complex "color inference" |
| **Composition with `exclusive`** | Clean: `exclusive async proc` — three orthogonal dimensions | `exclusive` still works, but "may suspend" is no longer explicit |

---

## Alternatives Considered

### Alternative 1: Hybrid Model (Go + explicit opt-in)

Keep `async` as an opt-in marker for *documentation*, not enforcement.
The compiler warns if a function that may suspend lacks `async`, but does
not block compilation. This preserves explicitness without coloring
lockdown.

**Downside:** Warnings that don't block compilation are easily ignored.
The semantic information ("this function may block") is still missing for
callers who don't read warnings.

### Alternative 2: Kotlin-style `suspend`

Use `suspend` instead of `async` — narrower semantics (only means "may
suspend", not "runs on a separate task"). Kotlin's `suspend` functions
are callable from any context via coroutine builders (`launch`, `async`,
`runBlocking`).

**Downside:** Renaming `async` to `suspend` does not solve the coloring
problem on its own. The real difference is that Kotlin allows `suspend`
functions to be called from non-`suspend` contexts via builders — which
is exactly what ASYNC_AS_EXPLICIT_MODIFIER already achieves with
`Future<T>` as first-class value.

### Alternative 3: Keep status quo + LLM tooling

Retain `async` as an explicit coroutine modifier. Invest in LSP
diagnostics, fix-its for missing `await`, and `baml describe`-style
tooling in the implementation repo to address LLM-generation friction.

**This is the recommended path.** See Evaluation below.

---

## Evaluation for Orthon

**Verdict: KEEP `async` modifier. Do not adopt coloring-free model.**

### Reasoning

1. **Explicitness is Orthon's constitutional principle.** DESIGN_PRINCIPLES.md
   (§ Explicitness): "Semantic changes must be syntactically visible."
   Dropping `async` makes suspension invisible — a function that blocks
   looks identical to one that doesn't. This contradicts a locked design
   principle and would require a Tier 1 EDR with extraordinary justification.

2. **The LLM-generability argument is unproven.** Modern LLMs (GPT-4o,
   Claude 4, Gemini 2.5) handle async/await correctly in Python,
   JavaScript, Rust, and C#. The compiler catches coloring errors
   locally — a missing `await` produces a type mismatch (`Future<T>` vs
   `T`), not a runtime bug. There is no evidence that coloring errors are
   a significant fraction of LLM-generated code failures.

3. **Orthon already mitigates coloring via `Future<T>` as first-class.**
   The ASYNC_AS_EXPLICIT_MODIFIER hypothesis already allows synchronous
   functions to call `async` functions and return `Future<T>` without
   `await`. This bridges the coloring gap without losing explicitness:
   ```orthon
   fun collectFutures(urls: List<Url>) -> List<Future<String>>
       return urls.map(fetchPage)  // fetchPage is async, no await needed
   ```

4. **The delegate model already separates "what" from "how."**
   CONCURRENCY_MODEL (EDR-033) uses `delegate` + `<-` for concurrent
   execution. The `async` modifier is about *suspension*, not
   *parallelism*. Dropping `async` conflates these two dimensions
   (exactly the error the ASYNC_AS_EXPLICIT_MODIFIER hypothesis
   corrected).

5. **The refactoring safety argument is strong.** When a developer adds
   I/O to a function, `async` propagation forces every caller to
   acknowledge the change. In a coloring-free model, `work(5)` silently
   becomes blocking — only runtime profiling would reveal the regression.

6. **Go's model has known costs.** Goroutine leaks, the inability to
   distinguish blocking from non-blocking calls by signature, and the
   "function color" problem inverted (everything is green, but some green
   functions block the OS thread) are well-documented Go pain points.
   Orthon should not import these without clear evidence of benefit.

### Recommendation

Retain `async` as an explicit coroutine modifier. Invest in LLM tooling
(LSP diagnostics, fix-its for missing `await`, `baml describe`-style
agent tooling) to address any LLM-generation friction, rather than
removing a principled language feature. If evidence of LLM coloring
errors accumulates during prototyping, revisit with a Tier 1 EDR and
measured data.

---

## Cross-References

- [ASYNC_AWAIT.md](ASYNC_AWAIT.md) — Current `async` modifier model
- [ASYNC_AS_EXPLICIT_MODIFIER.md](ASYNC_AS_EXPLICIT_MODIFIER.md) — Strengthened hypothesis with `exclusive`, `spawn`, `Future<T>`
- [CONCURRENCY.md](CONCURRENCY.md) — Concurrency model overview
- [DELEGATE.md](../essential/DELEGATE.md) — Delegate-based concurrency surface
- [SPAWN_OPERATOR.md](../deferrable/SPAWN_OPERATOR.md) — Explicit task creation
- [CONCURRENCY_MODEL.md](../../../../what/concepts/CONCURRENCY_MODEL.md) — ACCEPTED concurrency model (EDR-033)
- [BAML gap analysis](../../../notes/baml-concepts-orthon-gap-analysis.md) — Source document
