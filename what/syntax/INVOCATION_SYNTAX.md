# Invocation Syntax

> **Accepted — EDR-085 (Execution Context Invocation).** Record moved from
> `what/SYNTAX.md` into `what/syntax/` by EDR-087 (2026-08-15) so the hub
> stays lean. Full syntax is finalised in Phase 5; the shapes below are fixed
> by the semantic model.

Invocation has two syntactic forms, distinguished by whether an execution
context is present:

1. **Immediate invocation (call)** — `fn(args)`. No context, no operator.
   The base case of Invocation; executes now, in the current environment.

2. **Invocation in context** — the **two-operator family** that encodes
   the ownership relationship between the invocation and the context:

   | Operator | Meaning | Contexts | Glyph status |
   |----------|---------|----------|--------------|
   | `ctx <- fn(args)` | Submit to a **single owner** — serialised, in order | `delegate(obj)`, `defer(obj)` | Final |
   | `ctx |> fn(args)` | Submit to **stateless workers** — independent, parallel | `spawn()`, `fork()` | Provisional — final glyph deferred to Phase 5 |

   The operator set is **closed (2)**; the constructor set is **open** — a
   new context type resolves to one of the two by whether it owns state.

```orthon
add(1, 2)                 # immediate invocation — no context
counter <- increment()    # invocation in context — single owner, serialised
pool |> fetch(url)        # invocation in context — stateless workers, parallel
```

**Context constructors and materialisation** use named forms, not symbols:

```orthon
let task = defer(obj)     # coroutine context (wraps an object)
let result = await(task)  # materialise

let actor = delegate(Counter(0))   # actor context (wraps an object)
let state = take(actor)            # move the owned state out

let pool = spawn()        # thread context (stateless)
let data = grab(pool)     # materialise (StdLib sugar over generator protocol)
```

**`using` sugar** — `using` is pure syntactic sugar over context + scope +
deterministic destructor; it introduces no new semantics:

```orthon
# Canonical (context + scope + destructor)
using file = open("data.txt"):
    let content = file.read_all()
    process(content)
# desugars to delegate(open("data.txt")) + block + scope-bound destruction
```

Rejected glyphs: `.` (conflicts with immediate method call — the reader
could not tell whether `x.foo()` blocks), `<=` (already comparison), and
per-context operators (operator proliferation). `Send`/`Move` marker
traits are compile-time checks on captured data, not syntax.

## Cross-References

- [EDR-085](../../how/decision_records/architecture/EDR-085-execution-context-invocation.md) — deciding record.
- [`what/concepts/DELEGATE.md`](../concepts/DELEGATE.md) — delegate context (single owner).
- [`what/SYNTAX.md`](../SYNTAX.md) — hub.
- [`how/syntax/README.md`](../../how/syntax/README.md) — decision queue (resolved).
