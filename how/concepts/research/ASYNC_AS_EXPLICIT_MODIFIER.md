# Hypothesis: Async as Explicit Deferred Execution Modifier

Within the proposed method classification (`proc` — mutating action, `fun` — pure computation, `new` — factory), we introduce an **additional dimension of execution time**. By default, a method executes **immediately** (synchronously) — this behavior is implicit and requires no marker. A method marked with the `async` keyword guarantees **deferred execution**: it returns control to the caller immediately, while the main logic executes asynchronously, with the result becoming available later (via `Future`, `Promise`, or coroutine).

---

## 1. Problem

- **Ambiguity of expectations.** Without an explicit marker, it is impossible to distinguish a method that blocks the thread from one that launches a background operation. The developer must read documentation or the implementation to determine whether the method is safe to call in a performance-critical context.
- **Synchronisation errors.** Async methods that mutate state (`async proc`) require special attention to concurrent access. The absence of an explicit indicator leads to undetected data races or deadlocks.
- **Composability and readability.** Mixing synchronous and asynchronous calls creates "callback hell" or a chaotic mix of `await` and plain calls. Clear marking at the signature level enables static analysis and automated correctness checking.

---

## 2. Proposal

Introduce a **mandatory `async` modifier** for all methods that do not return a result instantly. Synchronous methods remain unmarked (implicit `immediate`). The modifier combines with the semantic categories:

- `async proc` — an asynchronous action that mutates an object. Returns `Future<void>` (or equivalent).
- `async fun` — an asynchronous pure computation. Returns `Future<T>`.
- `async new` — an asynchronous factory. Returns `Future<NewObject>`.

**Rules:**
- Calling an `async` method **requires** the `await` keyword (or explicit `Future` manipulation) to highlight deferred execution.
- An `async` method cannot be called in a context that does not support asynchrony (e.g., calling from a synchronous method without `await` — compile error).
- State mutated by `async proc` is considered "unstable" until the returned `Future` completes — any other calls (even synchronous reads) must either wait for completion or be explicitly prohibited during execution (via locking or a "busy" idiom).

---

## 3. Trade-offs

| Trade-off | Description |
|-----------|-------------|
| **Learning curve** | Developers must learn both the `proc/fun/new` semantics and the `async`/`immediate` distinction, plus their composition. This raises the entry barrier. |
| **Increased verbosity** | `async` and `await` must be written everywhere asynchrony is used. This is the price of explicitness and predictability. |
| **Implementation constraints** | `async proc` requires thread safety or call serialisation guarantees. Implementation becomes more complex, with potential synchronisation overhead. |
| **Performance** | Asynchronous operations add overhead for `Future`/task creation and management. For micro-operations this may be excessive, but the explicit `async` marker forces conscious choice. |
| **Debugging and tracing** | Async call stacks are less linear; locating errors related to operation ordering is harder. Requires advanced tooling (e.g., context logging). |
| **Composition with existing code** | Methods originally designed as synchronous cannot simply be made `async` without changing callers. This requires refactoring, which can be costly. |

---

## 4. Related Concepts and Alternatives

**Related concepts:**
- **Futures/Promises** — standard representation of deferred results. Our hypothesis assumes their use but is not tied to a specific implementation.
- **Coroutines** — enable writing async code in sync style via `async/await`. The hypothesis fits naturally within this approach.
- **Effect systems** (e.g., Haskell's `IO` or `Async` monad) — where asynchrony is reflected explicitly in types. Our classification is similar but simplified for imperative languages.
- **Reactive streams** (Rx, Flows) — an alternative for handling multiple async events, but these complement rather than replace explicit `async` methods.

**Alternative approaches:**
- **Implicit asynchrony** — all methods are potentially async; the runtime decides when and how to execute (e.g., in some actor models). This reduces explicitness but simplifies syntax.
- **Coloured functions** (as in older Node.js with callbacks) — distinguish sync and async functions without strict semantic classification. Our hypothesis adds semantic categories for richer information.
- **Documentation-only annotations** — no compiler support. The simplest path, but provides no guarantees and offers no protection against errors.
- **Unified `Task` type for all deferred operations** — no `proc/fun/new` distinction. Simpler, but loses semantic clarity about side effects.

---

## 5. Conclusion

The hypothesis proposes an **explicit, compile-time-checkable distinction** between synchronous (`immediate`) and asynchronous (`async`) methods while preserving and even strengthening the semantic triad of `proc`/`fun`/`new`. This solves problems of implicit expectations, synchronisation, and composability, at the cost of some language complexity and developer burden. Alternatives are either less expressive or require radically different computation models. The proposed approach integrates well with modern async programming tools (coroutines, `Future`) and can be adopted as an evolutionary improvement to existing systems.
