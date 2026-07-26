# Singleton Pattern Analysis — Composition Over Dedicated Construct

> **⚠️ DRAFT — This document is a preliminary hypothesis.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **Related:** [`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md) (proposes
> `object` keyword — this hypothesis challenges that approach),
> [`DELEGATE.md`](../essential/DELEGATE.md) (execution policy),
> [`CLASS_WITH_ACT.md`](../essential/CLASS_WITH_ACT.md) (field isolation),
> [`TOP_LEVEL_DECLARATIONS.md`](TOP_LEVEL_DECLARATIONS.md) (module-level values),
> [`CONTEXT_PARAMETERS.md`](../important/CONTEXT_PARAMETERS.md) (dependency injection alternative),
> [`LIBRARY_BOUNDARY.md`](../../../what/LIBRARY_BOUNDARY.md) (level classification framework)
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

---

## Issue (Why)

The Singleton pattern ensures a type has **exactly one instance** with a
**global access point**. In traditional OOP languages (Java, C#), this
requires a design pattern — boilerplate that the language cannot express
directly.

| Concern | Java/C# solution | Why it's a pattern |
|---------|-----------------|-------------------|
| Single instance guarantee | `static` field, `private` constructor | Manual enforcement — easy to bypass (reflection, serialization) |
| Global access point | `static getInstance()` method | Static methods are class-level, not type-level |
| Lazy initialization | `Lazy<T>` or double-checked locking | No built-in lazy semantics |
| Thread-safe init | `synchronized`, `volatile` | Manual — error-prone under concurrent access |
| Interface implementation | Class implements interface | Works, but creates tension with `private` constructor |

Kotlin's `object` keyword eliminated much of this boilerplate by making
singleton a language-level declaration — a class with exactly one lazily
initialised instance. But this raises the question for Orthon:

> **Should Orthon have a dedicated singleton declaration form (`object`),
> or can singletons be expressed through composition of existing primitives?**

---

## Principles

1. **Minimal Core** — Every language construct must earn its place. If a
   use case can be expressed through composition of existing primitives, a
   dedicated construct is unjustified. See [`../../../why/MANIFESTO.md`](../../../why/MANIFESTO.md)
   § Minimal Core.

2. **Orthogonality** — Each construct solves exactly one problem.
   Singleton is not a single problem — it is a bundle of concerns
   (single instance, global access, lazy init, thread safety). Each
   concern should be addressed by the appropriate primitive. See
   [`../../../how/DESIGN_PRINCIPLES.md`](../../../how/DESIGN_PRINCIPLES.md)
   § Orthogonality.

3. **Explicitness** — Thread safety and ownership transfer must be
   visible in the syntax. A hidden "lazy thread-safe global" keyword
   makes concurrency invisible. See `DESIGN_PRINCIPLES.md` § Explicitness.

4. **Data First** — Data is the primary abstraction. A "single instance"
   is just a value at module scope. The Singleton pattern exists because
   OOP conflates data + behaviour + identity. See
   [`FOUNDATIONAL_ABSTRACTIONS.md`](../essential/FOUNDATIONAL_ABSTRACTIONS.md)
   § Data First.

5. **Composition over Exceptions** — No special-purpose syntax for a
   pattern that can be composed from general-purpose primitives. See
   [`../../../why/MANIFESTO.md`](../../../why/MANIFESTO.md) § Composition
   over Exceptions.

---

## Policy Footprint

Singleton is not a single concept with its own policy footprint — it
decomposes into the policies of the primitives that compose it:

| Policy Type | Primitive | Role |
|---|---|---|
| Lifetime Policy | Module-level `let`/`val` | Singleton lives for application lifetime |
| Execution Policy | `delegate` | Thread-safe access to shared mutable state |
| Concurrency Policy | `delegate` (mailbox) | Serialised access under concurrent execution |
| Dispatch Policy | `<-` operator | Explicit concurrent message send vs direct call |
| Evaluation Policy | `Lazy<T>` (stdlib) | Deferred initialisation on first access |
| Visibility Policy | Module scope | Global access point within module boundary |
| Dependency Policy | `using`/`given` | Alternative: dependency injection replaces global access |

---

## Model (What)

### Hypothesis: Singleton Does NOT Need a Dedicated Construct

Orthon can express every Singleton scenario through **composition of
existing and planned primitives**. The following sections enumerate the
canonical scenarios and their Orthon expression.

---

### Scenario A: Read-Only Configuration (Most Common)

```orthon
let config = Config(version: "1.0", debug: false)
```

**Analysis:** A module-level `let` provides:
- **Single instance guarantee** — module initialises once per process
- **Global access point** — accessible within the module via import
- **No thread safety needed** — read-only data is safe by construction

**This is a singleton. One line. No pattern.**

---

### Scenario B: Shared Mutable Service (Concurrent Access)

```orthon
let db = delegate(move ConnectionPool("postgres://..."))
db <- query("SELECT...")
```

**Analysis:** `delegate` provides:
- **Single instance** — module-level (via the `let`)
- **Thread-safe access** — mailbox serialises all mutations
- **Ownership transfer** — `move` ensures no direct access bypasses the mailbox
- **Visible dispatch** — `<-` operator makes every concurrent access
  syntactically distinct from direct calls

No locks, no `synchronized`, no double-checked locking. The language
specifies only that delegated calls preserve ordering and provide
isolation; the runtime may use actors, thread pools, fibers, or event
loops.

See [`DELEGATE.md`](../essential/DELEGATE.md) § Ownership Integration.

---

### Scenario C: Expensive Lazy Resource

```orthon
let model = Lazy(|| loadModel("model.pb"))
prediction = model().predict(input)
```

**Analysis:** A `Lazy<T>` wrapper (stdlib) provides deferred
initialisation. The language does not need `lazy` as a keyword — if
closures and deferred execution exist, `Lazy<T>` is implementable as a
standard library wrapper:

```orthon
// Hypothetical stdlib API
class Lazy[T]
    private let factory: || T
    private var value: Option[T] = None

    proc call(self) -> T
        if value is None
            value = Some(factory())
        return value.value
```

**Open question:** If Orthon has no closures, or if `let` binding order
is strictly eager, `Lazy<T>` may require compiler support. Decision
depends on the evaluation model (see `SEMANTIC_MODEL.md` § Evaluation).

---

### Scenario D: Singleton Implementing a Trait (Polymorphism)

```orthon
let default_formatter = DefaultFormatter()

proc render(doc)(using fmt: Formatter)
    fmt.format(doc)
```

**Analysis:** A top-level value satisfies a trait bound naturally —
no special singleton syntax needed. The `using` context parameter
(see `CONTEXT_PARAMETERS.md`) provides implicit resolution.

This also demonstrates that **global access point is not always
desirable**: context parameters let the caller provide the dependency
without global state:

```orthon
// Call site — explicit override
render(my_doc)(using CustomFormatter())

// Call site — automatic resolution from scope
render(my_doc)
```

---

### Scenario E: Singleton + Field-Level Isolation

If a singleton has both read-only and thread-isolated fields:

```orthon
class ConfigManager
    private act cache: Cache          // isolated — needs mailbox
    private version: String           // read-only, non-isolated

    act update_config(v: String)
        version = v
        cache.clear()
```

See [`CLASS_WITH_ACT.md`](../essential/CLASS_WITH_ACT.md) § Field isolation rules.

---

### Scenario F: Lazy + Thread-Safe Singleton

Composing `Lazy<T>` with `delegate`:

```orthon
let shared = delegate(move Lazy(|| ExpensiveResource()))
shared <- call()
```

Semantics:
1. `Lazy` defers resource creation
2. `delegate` serialises access
3. `move` transfers ownership to the delegate

---

### Summary Matrix

| Scenario | Orthon expression | Primitives used |
|----------|------------------|-----------------|
| Read-only config | `let x = Config(...)` | Top-level declaration |
| Shared mutable | `let x = delegate(move T(...))` | Top-level + delegate + move |
| Lazy resource | `let x = Lazy(|| ...)` | Top-level + lazy (stdlib) |
| Polymorphic singleton | `let x = T()` + trait bound | Top-level + trait + context params |
| Field-level isolation | `class` with `act` fields | Class + act modifier |
| Lazy + thread-safe | `let x = delegate(move Lazy(|| ...))` | Composition of all above |

---

## Default Strategy

When a programmer needs a singleton-like pattern in Orthon, the default
response is:

```orthon
let name = Type(...)
```

For read-only configuration, this is sufficient. For mutable or concurrent
scenarios, the programmer composes additional primitives as needed
(`delegate`, `Lazy`, `act` modifier). The compiler does not introduce any
hidden singleton semantics — the pattern is fully explicit.

---

## Alternative Strategies

### Strategy A: `object` Keyword (Kotlin-Style)

Proposed in [`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md):

```orthon
object Config:
    version: String = "1.0"
    debug: Bool = false
```

| Pro | Con |
|-----|-----|
| Single obvious form | Adds language surface for a pattern that can be composed |
| Lazy + thread-safe by default | Overlaps with top-level declarations |
| Can implement traits | Creates class/object duality — "is `object` a type or a value?" |
| Anonymous objects useful too | Anonymous objects are a separate concern from singletons |

**Conclusion:** The `object` keyword adds language surface without filling
a gap that composition cannot cover. The only use case not fully covered
by composition is **anonymous objects** (implementing a trait inline
without a named type), which is a separate concern.

### Strategy B: Library-Based Singleton (Rust-Style)

A standard library `OnceCell` or `OnceLock` wrapper, analogous to Rust's
approach:

```orthon
let config = OnceCell::new()
config.set(Config("1.0"))
```

| Pro | Con |
|-----|-----|
| No language changes | Still a library pattern, not a language guarantee |
| Familiar to Rust/Go users | More ceremony than composition approach |  
| Explicit initialisation | No lazy-by-default semantics |

**Conclusion:** Subsumed by the composition approach — `Lazy` + module-level
`let` covers the same use cases with less ceremony.

### Strategy C: No Singleton Support (Pure Data Model)

If Orthon adopts a pure data model where all values are immutable and
behaviour is expressed through functions over data (see
`FOUNDATIONAL_ABSTRACTIONS.md`), the Singleton pattern is irrelevant —
there is nothing to "share" in the mutable sense. A module-level constant
is just a named value.

```orthon
let config = Config(version: "1.0")
```

| Pro | Con |
|-----|-----|
| Maximum simplicity | Requires pure data model (not yet decided for Orthon) |
| No concurrency concerns | Cannot model shared mutable state at all |

**Conclusion:** Contingent on Orthon's data model decision. If Orthon
allows mutable state, the composition approach is needed.

---

## Open Questions

1. **Does Orthon need `lazy` at the language level?** If closures and
   deferred evaluation exist, `Lazy<T>` can be a standard library wrapper.
   But `let lazy x = ...` syntax is cleaner for the most common case.
   Decision depends on how important deferred initialization is for the
   v0.1 target audience.

2. **Does the `object` keyword still serve the anonymous-object use case?**
   Anonymous objects (`val x = object : Interface { ... }`) are a separate
   concern from singletons. If anonymous objects are needed, `object` may
   still be justified — but scoped to that use case, not for singleton
   declaration.

3. **How does `delegate(move x)` interact with module-level singleton
   initialisation?** If `delegate` transfers ownership via `move`, and the
   singleton is at module level, who owns the delegate context? The module?
   The process? This affects lifetime guarantees.

4. **Does Orthon even need the Singleton concept at all in a Data-First
   model?** If data is the primary abstraction and behaviour is separate
   (functions over data), then "one instance of a type" is just a value —
   there is nothing special about it. The Singleton pattern exists only
   because OOP conflates data + behaviour + identity.

5. **What is the canonical form for a singleton in Orthon?** If the
   composition approach is accepted, should there be a recommended
   convention (e.g., "use module-level `let` for read-only, add `delegate`
   for mutable shared state") documented in a style guide?

---

## Decision History

| Date | Decision |
|------|----------|
| 2026-07-26 | Hypothesis created. Proposes composition over dedicated `object` keyword. Challenges `OBJECTS_AND_SINGLETONS.md`. |

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `OBJECTS_AND_SINGLETONS.md` (may need supersession notice)
- [ ] `TOP_LEVEL_DECLARATIONS.md`
- [ ] `DELEGATE.md`
- [ ] `CLASS_WITH_ACT.md`
- [ ] `CONTEXT_PARAMETERS.md`
- [ ] `LIBRARY_BOUNDARY.md`
