# Actor Implementation Taxonomy

> **Status:** 🟢 Reference
> **Related:** `how/concepts/research/ACTORS.md`, `DELEGATE.md`, `ACT_AS_ACTIVE_OBJECT.md`, `ACT_AS_FUNCTION.md`
> **Created:** 2026-07-26
>
> Catalogues 12 real-world approaches to implementing the actor model,
> categorized by architectural level (language, standard library, framework,
> manual pattern). Serves as the design-space map for Orthon's decision to
> relegate actor to a runtime implementation detail behind `delegate`.

## 1. Language-Level Actors

### 1.1 Lightweight Processes with Mailboxes

**Language:** Erlang / Elixir
**Runtime:** BEAM VM

The actor is the fundamental unit of concurrency at the virtual-machine level.
Each process owns a mailbox, isolated state, and behavior defined by functions.
No classes — actors are functions with tail-recursive loops. Isolation and
supervision are built into the OTP platform.

```erlang
% Spawn a process
Pid = spawn(fun() -> loop(InitialState) end).
% Send a message
Pid ! {add, 4}.
```

**Key trait:** Actor is part of the runtime, not an opt-in library.

### 1.2 `actor` as a Compiler-Enforced Type

**Language:** Swift

The `actor` keyword is a language-level reference type with compiler-enforced
isolation. The compiler statically prevents data races by requiring `await`
for cross-actor calls and rejecting direct state access from outside the
actor's execution context.

```swift
actor Counter {
    private var value = 0
    func increment() { value += 1 }
    func get() -> Int { value }
}
```

**Key trait:** Isolation checking at compile time; reentrancy via `async`/`await`.

### 1.3 Reference Capabilities (Type-Level Race Prevention)

**Language:** Pony

Actors are built-in, but race freedom is guaranteed not by isolation rules
but by reference capabilities (`ref`, `val`, `iso`, `tag`, `box`, `trn`).
The type system denies sharing of mutable data between actors at compile time.

```pony
actor Counter
  var _n: U32 = 0
  be inc() => _n = _n + 1
```

**Key trait:** No runtime isolation checks — the type system proves absence of
races. Actors are a separate type but require no inheritance.

---

## 2. Standard-Library Actors

### 2.1 Goroutines + Channels (CSP Style)

**Language:** Go

Each actor is a goroutine holding exclusive state and a channel for incoming
messages. Sending is an explicit channel operation. No built-in supervision —
the programmer must implement lifecycle management manually.

```go
func NewActor() chan<- int {
    ch := make(chan int)
    go func() {
        var list []int
        for x := range ch {
            list = append(list, x)
        }
    }()
    return ch
}
actorCh := NewActor()
actorCh <- 4   // fire-and-forget command
```

**Key trait:** Actors are not a language concept; they emerge from composition
of goroutines, channels, and `select`. Explicit channel management.

### 2.2 Thread-Safe Collections as Actor Building Blocks

**Languages:** Java (`ConcurrentHashMap`, `ConcurrentLinkedQueue`),
C# (`ConcurrentDictionary`)

These are not actors in the classical sense, but they provide lock-free,
thread-safe data structures that serve as the internal state of actor
implementations. Some frameworks (e.g., Microsoft Orleans) build
virtual actors on top of distributed caches.

```java
ConcurrentMap<String, Integer> map = new ConcurrentHashMap<>();
// Used inside an actor as its private, thread-safe state
```

**Key trait:** Low-level primitives, not an actor model. Provide the
foundations on which actor systems are built.

---

## 3. Framework / Library Actors

### 3.1 Class-Based Actor Library

**Frameworks:** Akka (Scala/Java), Akka.NET (C#)

The actor is an instance of a class inheriting from `Actor` with an overridden
`receive` method. Communication happens through `ActorRef` handles. Supports
distribution, clustering, persistence, and supervision.

```scala
class ListActor extends Actor {
  var list = List.empty[Int]
  def receive = {
    case Add(x) => list = x :: list
    case Get    => sender() ! list
  }
}
val ref = system.actorOf(Props[ListActor])
ref ! Add(4)          // command (fire-and-forget)
val future = ref ? Get // query (returns Future)
```

**Key trait:** Strong typing, distributed actors, supervision hierarchies.
Library-level — the language knows nothing about actors.

### 3.2 Virtual Actors (Identity by Key, Not Reference)

**Frameworks:** Microsoft Orleans (C#), Proto.Actor

An actor is identified by a key (GUID), not by an object reference. The
runtime creates an instance on first access and may deactivate it after
a period of idleness. Messages are routed by key. This enables scaling
to thousands of actors without explicit lifecycle management.

```csharp
var grain = GrainClient.GrainFactory.GetGrain<IListGrain>(id);
await grain.Add(4);   // async call; actor may be inactive or remote
```

**Key trait:** Automatic lifecycle, location transparency, massive scale.

### 3.3 Dynamic Proxy Wrappers

**Language:** Python (hypothetical / third-party libraries)

A class is decorated with `@actor`, and all method calls on its instances
are transparently converted into asynchronous messages. The decorator
intercepts attribute access and enqueues method invocations.

```python
@actor
class Counter:
    def __init__(self): self.n = 0
    def inc(self): self.n += 1

c = Counter()
c.inc()   # automatically async, non-blocking
```

**Key trait:** Transparency for the programmer, but requires metaclass /
`__getattribute__` magic and a background event loop.

### 3.4 Function `act(obj)` — Proxy Actor for an Arbitrary Object

**Language:** Python (dynamic proxy pattern)

Any object is passed to `act()`, which returns a proxy that intercepts method
calls and enqueues them in the actor's mailbox. The original object is held
exclusively by the actor.

```python
lst = [1, 2, 3]
proxy = act(lst)
proxy.append(4)               # async command
future = proxy.__getitem__(0)  # query via special method
```

**Key trait:** Works with any object, but does not distinguish commands from
queries without explicit `ask` patterns.

---

## 4. Manual / Pattern-Based Approaches

### 4.1 CQRS Actors (Command-Query Separation at the API Level)

**Applicable to:** Any actor framework (Akka, Orleans, Pykka)

The actor API explicitly separates **commands** (mutate state, no return
value) from **queries** (read state, return `Future`). This matches the
CQRS pattern and prevents accidental blocking on state reads.

```text
actor ! Add(4)          // command — fire-and-forget
result = actor ? Get(0)  // query — returns Future
```

**Key trait:** Discipline at the API level; clarifies boundaries between
side-effecting and pure-read operations.

### 4.2 Manual Queue + Thread Loop

**Applicable to:** Any language (C++, Python, Java, etc.)

The most basic implementation: create a thread with an infinite loop reading
from a `Queue` and dispatching to method calls on a guarded object. Full
control, but considerable boilerplate.

```python
import queue, threading

class ManualActor:
    def __init__(self, target):
        self._target = target
        self._q = queue.Queue()
        threading.Thread(target=self._loop, daemon=True).start()

    def _loop(self):
        while True:
            msg = self._q.get()
            method, args = msg
            getattr(self._target, method)(*args)

    def send(self, method, *args):
        self._q.put((method, args))
```

**Key trait:** Maximum control; no framework dependency. Boilerplate,
manual lifecycle, no built-in supervision.

### 4.3 Operator Overloading for Message-Send Syntax

**Languages:** Scala (`!`), Kotlin, Python (`<<` via `__lshift__`)

Operators are overloaded to make message-send syntax declarative and concise.
Purely syntactic sugar — does not change the underlying actor semantics.

```scala
actor ! Add(4)          // Akka standard operator
```

```python
actor << Add(4)          # via __lshift__
```

**Key trait:** Improves readability; does not affect semantics or
implementation.

### 4.4 Future / Promise for Query Responses

**Applicable to:** All actor frameworks (Akka, Orleans, Pykka, etc.)

A cross-cutting pattern: commands are fire-and-forget; queries return a
`Future` / `Promise` that the caller can `await`. This is not a separate
implementation style — it is the standard mechanism for request-response
interaction in actor systems.

```scala
val future: Future[Any] = actor ? Get
val result = Await.result(future, timeout)
```

---

## Summary Table

| # | Category | Examples | Implementation Level | Key Feature |
|---|----------|----------|---------------------|-------------|
| 1 | Lightweight processes | Erlang, Elixir | Language / VM | Actor as VM primitive; supervision trees |
| 2 | Compiler-enforced type | Swift | Language | Compile-time isolation checking |
| 3 | Reference capabilities | Pony | Language | Type-level race-freedom proof |
| 4 | Goroutines + channels | Go | Standard library | CSP composition; manual supervision |
| 5 | Concurrent collections | Java, C# | Standard library | Building blocks, not full actor model |
| 6 | Class-based library | Akka, Akka.NET | Framework | Inherit `Actor`; distribution, persistence |
| 7 | Virtual actors | Orleans, Proto.Actor | Framework | Identity by key; automatic lifecycle |
| 8 | `@actor` decorator | Python (third-party) | Framework | Transparent method → message conversion |
| 9 | `act(obj)` proxy | Python (pattern) | Framework | Wrap any object in an actor |
| 10 | CQRS separation | Any framework | Pattern | Explicit command vs. query API |
| 11 | Manual queue + thread | Any language | Manual | Full control; boilerplate |
| 12 | Operator overloading | Scala, Kotlin, Python | Syntax sugar | `!` / `<<` for message send |
| — | Future/Promise | All frameworks | Cross-cutting | Standard request-response mechanism |

---

## Relevance to Orthon

Orthon explored actors as a potential language concept:

- `ACTORS.md` — surveyed Swift, Erlang, Pony, Akka, and Kotlin approaches;
  identified the core problem: shared mutable state with explicit locking.
- `ACT_AS_ACTIVE_OBJECT.md` — explored `act` as a type-level construct with
  mailbox and delegate semantics.
- `ACT_AS_FUNCTION.md` — explored lifting `act` to the function level.

**Decision (2026-07-26):** Actor is removed from the language surface.
`DELEGATE.md` introduces a single execution policy — `delegate` — that
wraps any state-owning entity in a delegated execution context. The runtime
may implement this using actors, mailboxes, queues, executors, fibers, or
any other mechanism. The language specification does not mandate a concrete
implementation.

This taxonomy captures the full design space that was considered before
arriving at that decision. Each row in the summary table corresponds to
a point in the trade-off space that was weighed against Orthon's design
principles (orthogonality, minimal core, implementation independence).

---

## See Also

- [`../how/concepts/research/ACTORS.md`](../how/concepts/research/ACTORS.md) — original actor survey (superseded)
- [`../how/concepts/research/DELEGATE.md`](../how/concepts/research/DELEGATE.md) — current hypothesis (`delegate` as execution policy)
- [`../how/concepts/research/ACT_AS_ACTIVE_OBJECT.md`](../how/concepts/research/ACT_AS_ACTIVE_OBJECT.md) — `act` as active object (superseded)
- [`../how/concepts/research/ACT_AS_FUNCTION.md`](../how/concepts/research/ACT_AS_FUNCTION.md) — `act` at function level (superseded)
- [`../what/SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) — unified semantic model
- [`../how/DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) — 27 design rules
