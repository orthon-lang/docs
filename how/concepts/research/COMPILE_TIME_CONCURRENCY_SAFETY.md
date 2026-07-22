# Compile-Time Concurrency Safety

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does a language guarantee freedom from data races in concurrent programs without runtime overhead?

The evolution of concurrency safety reveals a progression:

1. **Manual locking** (Java `synchronized`, C `pthread_mutex_lock`) — the programmer is responsible for acquiring and releasing locks in the correct order. Violations produce data races, deadlocks, and non-deterministic behaviour — all at runtime. The compiler offers no help.

2. **Higher-level abstractions** (Java `java.util.concurrent`, C# `Task`/`async`) — safer primitives (locks with structured acquire/release, thread pools, futures) reduce the surface area for errors but do not eliminate them. Deadlocks and data races are still possible; they are merely less frequent.

3. **Actor model** (Erlang, Elixir, Akka) — no shared state. Each actor owns its data; communication is via messages. Data races are impossible by construction, but the model requires designing the entire program around actor boundaries, which is not always natural.

4. **Compile-time type-level guarantees** (Rust's `Send` and `Sync` traits) — the type system proves that certain classes of concurrency errors cannot occur. `Send` types can be transferred between threads safely. `Sync` types can be shared between threads safely. The compiler checks these properties at compile time; if a program compiles, it is free of data races.

| Approach | Data Race Freedom | Deadlock Freedom | Runtime Overhead | Compiler Proof |
|---|---|---|---|---|
| **Manual locking** (Java) | 🔴 Not guaranteed | 🔴 Not guaranteed | Low (lock ops) | ❌ |
| **Higher-level abstractions** (Java concurrent) | 🟡 Reduced risk | 🔴 Not guaranteed | Medium | ❌ |
| **Actor model** (Erlang) | 🟢 By construction | 🟢 By construction | Medium (message passing) | ⚠️ Partial |
| **Type-level Send/Sync** (Rust) | 🟢 Compile-time proven | 🟡 Still possible (lock ordering) | None (zero-cost) | ✅ Full |

The core problem: **how does Orthon prove the absence of data races at compile time**, without incurring runtime overhead (GC, reference counting, or lock operations)?

## Principles

1. **Data races are compile-time errors** — No data race can occur in a program that compiles. The type system is the enforcement mechanism, not the runtime.

2. **Send and Sync as type-level properties** — Marker traits (or equivalent) classify types by their thread-safety guarantees:
   - **Send** — values of this type can be transferred to another thread.
   - **Sync** — values of this type can be shared between threads (accessed through a shared reference from multiple threads).

3. **Shared XOR mutable** — Data is either:
   - Shareable across threads but immutable (multiple readers allowed).
   - Mutable but exclusively owned by one thread (single writer).
   - Never both simultaneously without explicit synchronisation.

4. **Auto-derive** — Send and Sync are automatically derived for types composed entirely of Send/Sync components. Manual opt-out for types that contain interior mutability or thread-local state.

5. **Zero-cost** — All safety checks happen at compile time. No runtime overhead, no GC, no reference counting for concurrency safety.

6. **Composable** — Concurrency safety properties compose through generics. A generic function that works with `T: Send` works with any Send type, without specialisation.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Concurrency Model Policy | Selects between type-level Send/Sync, actor model, or shared-memory threading |
| Send/Sync Policy | Governs auto-derive rules, manual opt-out, and interaction with unsafe code |
| Isolation Policy | Controls whether state is isolated per-thread or shared with synchronisation |
| Mutability Policy | Shared XOR mutable interacts directly with the concurrency model — shared references imply immutability across threads |
| Ownership Policy | Ownership transfer (`move` semantics) enables Send; borrowing rules enable Sync |

## Model (What)

### Send: Thread Transfer Safety

A type `T` is **Send** if ownership of a value of type `T` can be safely transferred to another thread.

Types that are Send:
- Primitive types (`Int`, `Float`, `Bool`)
- Types whose fields are all Send
- `Arc<T>` where `T: Send + Sync`
- `Mutex<T>` where `T: Send`

Types that are **not** Send (marked `!Send`):
- `Rc<T>` (not thread-safe — reference counting is not atomic)
- `RefCell<T>` (runtime borrow checking is not thread-safe)

```
// A Send type can cross thread boundaries
@Send
struct Task
    id: Int
    data: String

// Error: attempting to send a non-Send type to a thread
let rc = Rc::new(42)
spawn(rc)  // compile error: Rc<Int> is not Send
```

### Sync: Thread Sharing Safety

A type `T` is **Sync** if a shared reference `&T` can be safely accessed from multiple threads simultaneously.

Types that are Sync:
- Primitive types
- Immutable types with no interior mutability
- `Arc<T>` where `T: Send + Sync`
- `Mutex<T>` where `T: Send`

Types that are **not** Sync (marked `!Sync`):
- `RefCell<T>` (runtime borrow checking is not thread-safe)
- `UnsafeCell<T>` (raw interior mutability — wrapper must provide synchronisation)

```
// A Sync type can be shared across threads
let config = Arc::new(AppConfig { ... })

// Multiple threads can read config simultaneously
for _ in 0..num_threads:
    let c = Arc::clone(&config)
    spawn(|| process(c))
```

### Auto-derive Rules

Send and Sync are **auto traits** — the compiler derives them automatically for types composed entirely of Send/Sync components. The programmer only needs to opt out explicitly for types that contain thread-unsafe interior mutability:

```
// Auto-derived: Task is Send because all fields are Send
struct Task
    id: Int
    data: String

// Manual opt-out: Cache uses UnsafeCell internally
unsafe impl !Sync for Cache
```

### Interaction with Ownership

- **Move semantics** are the default mechanism for Send: passing a value to `spawn()` transfers ownership, preventing the original thread from accessing the value after the transfer.
- **Borrowing** across threads requires Sync: a shared reference `&T` can only be sent to another thread if `T: Sync`.
- **Mutable references** `&mut T` are not `Sync` — two threads cannot simultaneously hold mutable references to the same value.

## Default Strategy

Type-level Send/Sync marker traits with auto-derive, explicit opt-out, and no runtime overhead. Combined with ownership model (move semantics prevent use-after-transfer). The compiler proves data-race freedom. Deadlock freedom is still the programmer's responsibility (lock ordering discipline, or async/await with cooperative scheduling).

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Actor model** | Erlang, Elixir | Data-race-free and deadlock-free by construction, but requires whole-program actor architecture. Message passing overhead. Not suitable for fine-grained concurrency. |
| **CSP (Communicating Sequential Processes)** | Go | Channels as first-class primitives. Deadlocks still possible (channel starvation). No compile-time guarantee of race freedom. |
| **Software Transactional Memory** | Clojure, Haskell | Optimistic concurrency with rollback on conflict. Runtime overhead (transaction bookkeeping). Not deterministic. |
| **Manual locking** | Java, C++ | Maximum control, minimum safety. Full responsibility on the programmer. All concurrency errors are runtime errors. |
| **Async/await with single-threaded runtime** | JavaScript, Python (asyncio) | No data races because there is no true parallelism (single-threaded event loop). Limited to I/O-bound concurrency. |

## Open Questions

1. Should Orthon use `Send`/`Sync` as named traits, or should thread-safety properties be implicit in the type system (e.g., all types are Send/Sync unless explicitly marked otherwise)?

2. How should interior mutability (analogous to Rust's `Cell`/`RefCell`) interact with Sync? Should Orthon provide safe abstractions or require unsafe opt-out?

3. Should deadlock freedom be addressed at the type level? (E.g., lock ordering enforced by the compiler, or a structured concurrency model that prevents resource cycles.)

4. How does compile-time concurrency safety interact with the Execution Program model — can an execution strategy opt into a different concurrency model without changing the source program?

5. LLM implications: `Send`/`Sync` bounds on generic code add complexity. Should the Schema Provider expose thread-safety properties so LLMs can generate correct concurrent code without guessing?

## References

- [`CONCURRENCY.md`](./CONCURRENCY.md) — actor model and channel-based concurrency (complementary to type-level safety)
- [`OWNERSHIP.md`](./OWNERSHIP.md) — ownership model provides the foundation for Send (move semantics)
- [`MUTABILITY.md`](./MUTABILITY.md) — shared XOR mutable is the core invariant behind Sync
- [`ERROR_HANDLING.md`](./ERROR_HANDLING.md) — error handling in concurrent contexts (error propagation across threads)
