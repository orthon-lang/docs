# Hypothesis: Command Pattern via Language-Level Delegate

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Problem

The GoF Command pattern encapsulates a request as an object:

```java
// Java — traditional Command pattern
interface Command {
    void execute();
}

class PrintCommand implements Command {
    private final String message;
    PrintCommand(String message) { this.message = message; }
    public void execute() { System.out.println(message); }
}

// Usage
Command cmd = new PrintCommand("hello");
cmd.execute();
```

This requires a separate class (or anonymous inner class) per command. For
trivial operations — a single function call with captured parameters — the
ceremony-to-meaning ratio is punishing. In a codebase of any size, this
produces class explosion and indirection that obscures the actual program
logic.

The same pattern appears across related use cases:

| Use case | What varies | Traditional OOP solution |
|---|---|---|
| UI event handlers | The action to perform on click | `ActionListener` interface + class |
| Undo/redo | The forward and inverse operations | `Command` + `UndoableCommand` interfaces |
| Task queues | The work item | `Runnable`/`Callable` + class |
| Callbacks | The post-completion logic | `Callback` interface + class |
| Middleware/chain | The processing step | `Handler` interface + class |
| Macro commands | The composite of sub-commands | `CompositeCommand` + `Command` list |

All are the same concept — **deferred invocation of behaviour** — distinguished
only by the number of parameters and presence of a return value.

Languages with first-class functions collapse these into a single concept:
**a callable value** (lambda, closure, delegate, function reference).

## Examples

### Languages Where the Pattern Disappears

**C# — delegates and lambdas:**

```csharp
// No Command interface needed. Delegate is the command.
Action<string> cmd = msg => Console.WriteLine(msg);
cmd("hello");

// Undo via paired delegates
var cmd = (execute: () => InsertChar(c),
           undo: () => DeleteChar());

// Queue of heterogeneous commands
List<Action> queue = new();
queue.Add(() => Log("start"));
queue.Add(() => Process(data));
```

**Python — first-class functions:**

```python
# Any callable is a command
cmd = lambda msg: print(msg)
cmd("hello")

# Or a function reference
cmd = print
cmd("hello")

# Undo via closure capturing previous state
def set_volume(new):
    old = current_volume
    current_volume = new
    return lambda: setattr(current_volume, old)  # undo

undo_fn = set_volume(50)
undo_fn()  # reverts
```

**JavaScript — callbacks everywhere:**

```javascript
// Event handler is just a function
button.onclick = () => console.log("clicked");

// Task queue
const tasks = [];
tasks.push(() => fetch(url));
tasks.push((data) => render(data));
```

**Rust — closures with trait coercion:**

```rust
// Command is Box<dyn FnOnce()>
let cmd: Box<dyn FnOnce()> = Box::new(|| println!("hello"));
cmd();

// Or generic over Fn trait
fn execute<F: FnOnce()>(f: F) { f(); }
```

### Languages That Still Need the Pattern

**Java (pre-8)** — No first-class functions. Anonymous classes are the only
option, producing the verbosity that the pattern was created to manage.

**Java (post-8)** — Lambdas help for simple cases, but checked exceptions and
erased generics mean functional interfaces still proliferate:

```java
// Better, but still requires @FunctionalInterface declaration
Command cmd = () -> System.out.println("hello");
```

### Pattern Isomorphism

In a language with delegates/callables:

| OOP pattern | Delegate form |
|---|---|
| `Command` | `T -> void` delegate |
| `Runnable` | `() -> void` delegate |
| `Callable<V>` | `() -> V` delegate |
| `ActionListener` | `Event -> void` delegate |
| `Comparator<T>` | `(T, T) -> int` delegate |
| `Supplier<T>` | `() -> T` delegate |
| `Consumer<T>` | `T -> void` delegate |
| `Function<T,R>` | `T -> R` delegate |
| `Predicate<T>` | `T -> bool` delegate |

All are **the same concept** — deferred invocation — differing only in arity
and variance. A language with a unified delegate type and closure support
can express all of them without a single interface declaration.

## Level

**Core language.** The question is not whether Orthon should add a Command
construct — it is whether Orthon's function/delegate model is complete enough
that the Command pattern never needs to be encoded as a separate abstraction.

The implications touch three levels:

| Level | Concern |
|---|---|
| **Core** | Are delegates first-class? Can they capture state (closures)? Do they have a type that can be abstracted over? |
| **Stdlib** | Provide `Undoable[T]`, `CommandQueue`, `Macro` compositors if the language itself doesn't need them. |
| **Framework** | UI frameworks, middleware, task schedulers — but these are downstream consumers of the core delegate model. |

## Proposal

Orthon already has `delegate` as a concept (see `DELEGATION.md`). The
hypothesis is:

> **A single delegate type combined with closure capture eliminates the need for a dedicated Command pattern construct in Orthon.**

Concretely:

### 1. Delegates as Commands

```orthon
// A command is just a delegate
cmd: () -> void = || print("hello")
cmd()
```

### 2. Parameterized Commands

```orthon
// Closure captures parameter
name: String = "world"
greet: () -> void = || print("Hello, {name}")   // captures name
greet()
```

### 3. Commands with Return Values

```orthon
fetch: () -> Result[Data] = || http.get(url)
data = fetch()
```

### 4. Undoable Commands (Paired Delegates)

```orthon
// An undoable command is a pair of delegates
struct Undoable:
    execute: () -> void
    undo: () -> void

// Usage
cmd := Undoable(
    execute: || buffer.insert(pos, char),
    undo:    || buffer.delete(pos)
)
cmd.execute()
cmd.undo()
```

### 5. Command Queue (Heterogeneous)

```orthon
// Queue of commands with different types — all () -> void
queue: List[() -> void] = []
queue.append(|| print("A"))
queue.append(|| process(data))
queue.append(|| notify(user))

for cmd in queue:
    cmd()
```

### 6. Macro Commands (Composition)

```orthon
// Composite via list of delegates
macro := [cmd1, cmd2, cmd3]
for cmd in macro:
    cmd()
```

The key insight: **no `command` keyword, no `Command` interface, no separate
class per command** — just delegates and closures.

### What the Language Must Provide

For the pattern to truly disappear, Orthon's core must guarantee:

1. **First-class delegates** — A function/delegate is a value with a type.
2. **Closure capture** — A delegate can capture variables from its enclosing scope.
3. **Type-safe delegate types** — `(Arg1, Arg2) -> Return` as a first-class type.
4. **Uniform invocation syntax** — `delegate(args)` works regardless of whether it's a named function, anonymous lambda, or bound method reference.
5. **No checked-exception interference** — A delegate type must not require the caller to handle exceptions that the delegate body may throw (or exceptions must be part of the type signature explicitly).

### What Becomes Possible

With the above, entire categories of OOP design patterns become **library code**
or **idiom** rather than language features:

| Pattern | Orthon equivalent |
|---|---|
| Command | `() -> T` delegate |
| Strategy | `(Args) -> Result` delegate |
| Observer | Delegate list + event dispatch in stdlib |
| Template Method | Higher-order function accepting delegate |
| Visitor | Pattern matching + delegates per case |
| Null Object | Optional[T] + default value |

## Trade-offs

### ✅ Advantages

- **No class explosion.** One delegate type replaces N command classes.
- **Minimal syntax.** Lambda `|| expr` is the lightest possible command declaration.
- **Composable.** Delegates compose naturally (list, pipe, chain).
- **Consistent with functional model.** Aligns with Orthon's data-first philosophy.
- **No new keywords.** Reuses existing delegate and closure concepts.
- **Undo/open/close patterns fall out naturally.** Paired delegates need no framework.

### ⚠️ Disadvantages

- **Serialization is harder.** A delegate's closure captures runtime references — serializing it for persistence (e.g., command journaling, CQRS event store) requires explicit serialization strategy or a separate data representation. Traditional Command classes with explicit fields serialize trivially.
- **No named intent at the type level.** `Command` as a class name conveys intent. `() -> void` conveys only arity. For large codebases, the loss of semantic type names may reduce readability.
    - **Mitigation:** Type aliases: `type LogCommand = () -> void` restores semantic naming.
- **Undo requires discipline.** The paired-delegate approach (`execute` + `undo`) requires the programmer to pass both explicitly. There is no compiler enforcement that an undo reverses the execute.
    - **Note:** This is inherent — even a `Command` interface with `undo()` does not guarantee correctness.
- **No built-in composability constraints.** A `List[() -> void]` can mix related and unrelated commands. Traditional `MacroCommand` classes enforce homogeneity at the type level.
    - **Mitigation:** This is a feature, not a bug — type-level restriction is opt-in via newtypes.
- **Tooling/debugging.** Anonymous closures are harder to name in stack traces and profiling. Named functions (or explicit delegate variables) mitigate this.
- **LLM generability consideration.** A lambda-based Command is simpler for an LLM to generate (one expression) than a class hierarchy. The mental model is "capture and defer" rather than "implement interface." This aligns with Orthon's LLM Readiness pillar.

## Related Concepts

| Concept | Relationship |
|---|---|
| **Delegation** (`DELEGATION.md`) | Provides the delegate type that enables the pattern. Without delegates, Command requires explicit classes. |
| **Functions** (`FUNCTIONS.md`) | First-class functions are the prerequisite — delegates are typed function references. |
| **Closures** (implicit in delegate model) | Closure capture is what makes a delegate a *parameterized* command without explicit parameter fields. |
| **Active Object** (`deferrable/ACT_AS_ACTIVE_OBJECT.md`) | Active Object *uses* commands by queuing them as delegates. The delegate-is-Command hypothesis is a prerequisite for the Active Object mailbox model. |
| **Async/Await** (`important/ASYNC_AWAIT.md`) | Commands queued for async execution are delegates. Async schedulers consume `() -> Future[T]` delegates. |
| **Pipelines / Chains** | Processing pipelines are composable delegate chains. Middleware is `(Request, () -> Response) -> Response`. |

## Alternatives

### A. Dedicated `Command` Keyword

```orthon
command Greet(msg: String):
    print(msg)

// Usage
cmd = Greet("hello")
cmd()
```

**Pros:** Named intent, potential for built-in undo/redo infrastructure,
serialization support.

**Cons:** New keyword, class explosion returns, violates minimal core
principle. Conflicts with `delegate` concept overlap — what makes
`command` different from `delegate`?

**Verdict:** Reject. A dedicated keyword adds ceremony without semantic
value. The delegate model already covers all use cases.

### B. Stdlib `Command[T]` Type Only (No Core Change)

Provide a standard library type:

```orthon
type Command = () -> void
type CommandWithResult[T] = () -> T
type Undoable[T] = (execute: () -> T, undo: () -> void)
```

**Pros:** Zero language changes. Pure library.

**Cons:** Already achievable with existing delegate model. The library
adds documentation value but no new capability.

**Verdict:** Accept as documentation convention, not a new feature.

### C. Trait-Based Command (Rust Style)

Define a `Command` trait that types implement:

```orthon
trait Command:
    fun execute(self) -> void

struct PrintCommand:
    message: String

impl Command for PrintCommand:
    fun execute(self):
        print(self.message)
```

**Pros:** Type-safe, named, extensible with undo/redo/logging.

**Cons:** Returns to OOP class-per-command explosion. Defeats the purpose
of the hypothesis.

**Verdict:** Reject. Available as a fallback for cases that genuinely need
named command types, but not the recommended idiom.

## Open Questions

1. **Should Orthon provide a stdlib `Undoable` compositor?** E.g., a type that composes two delegates and enforces the undo contract via documentation (not compiler).

2. **Does the delegate model need named-argument support for readability?**
   `Undoable(execute: || ..., undo: || ...)` vs unnamed tuple `(|| ..., || ...)`.

3. **How should command serialization (for event sourcing / CQRS / journaling) work?** Is this a language concern or a serialization library concern? If the latter, what constraints does serialization place on closures (no external captures, or captured values must be serializable)?

4. **Is there value in a `captures` annotation that makes closure captures explicit and checkable?** E.g., `cmd: () -> void = [capture: name] || print(name)` — useful for auditability of deferred commands.

5. **Does the Command pattern ↔ delegate isomorphism imply that other GoF creational/structural patterns also collapse?** If Factory Method is just a delegate (`() -> T`), and Proxy is just a delegate wrapper, then Orthon's design surface shrinks considerably — which is the goal.

## Decision History

Initial hypothesis — no decisions recorded yet.

---

### Affected Documents

- [ ] `DELEGATION.md` — update to note that delegates subsume the Command pattern
- [ ] `FUNCTIONS.md` — ensure first-class functions and closure capture are specified
- [ ] `deferrable/ACT_AS_ACTIVE_OBJECT.md` — note that the mailbox model consumes delegates as commands
- [ ] `what/GLOSSARY.md` — add Command (as idiom, not language construct) if needed
- [ ] `how/DESIGN_PRINCIPLES.md` — verify minimal core principle is satisfied
