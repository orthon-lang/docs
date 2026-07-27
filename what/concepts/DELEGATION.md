# Delegation

## Issue (Why)

How does a type reuse behaviour from another type without inheritance — and how does a property get custom behaviour (lazy, observable, validated) without boilerplate? These are two distinct but related problems:

1. **Class delegation** — A type implements an interface by forwarding all calls to a contained instance. Without language support, this requires hand-writing every forwarding method.
2. **Property delegation** — A property's getter/setter behaviour is delegated to a helper object (lazy initialization, observable change notification, validated assignment).

Kotlin solves both with the `by` keyword. The core question: should Orthon have built-in delegation as a language construct, or should these patterns remain in userspace?

## Principles

1. **Composition over inheritance** — Delegation replaces many uses of implementation inheritance (decorator, proxy, adapter patterns) with explicit, composable forwarding.
2. **Explicitness** — Delegation is declared with the `by` keyword at the definition site, making forwarding relationships syntactically visible.
3. **No boilerplate** — The compiler or StdLib generates forwarding code, eliminating repetitive method stubs.
4. **Selective override** — Explicit method definitions in the delegating type take precedence over delegated methods.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Delegation Policy | Determines whether delegation is compiler-generated (Language) or composition-based (StdLib) |
| Method Resolution Policy | Governs precedence between explicit methods and delegated methods |

## Model (What)

### Class Delegation via StdLib Composition

A type implements a trait by composing with a contained instance. The standard library provides a `delegate` helper that generates forwarding at the composition site:

```orthon
trait List[T]:
    fn size(self) -> Int
    fn get(self, index: Int) -> T
    fn append(mut self, item: T)

struct LoggingList[T]:
    inner: List[T]

    @delegate(List[T]) to inner
    # Compiler generates forwarding for size(), get(), append()
    # Selective override:
    fn append(mut self, item: T):
        log("appending {item}")
        self.inner.append(item)
```

### Property Delegation

Properties can delegate their getter/setter to a delegate object:

```orthon
name: String by lazy { loadName() }
counter: Int by observable(0) { old, new -> log("$old -> $new") }
```

### Delegate Protocol

Any type implementing a standard `Get` (and optionally `Set`) protocol can be a property delegate:

```orthon
trait Get[T]:
    fn get(this_ref: Any, prop: PropertyMetadata) -> T

trait Set[T]:
    fn set(this_ref: Any, prop: PropertyMetadata, value: T)
```

## Default Strategy

Delegation is a StdLib pattern — the StdLib provides `@delegate` macros and delegate protocols (`lazy`, `observable`, `vetoable`, `map`). The compiler generates forwarding code for `@delegate` annotations but does not introduce new delegation-specific syntax.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Language-level `by` keyword | Compiler generates forwarding methods directly from `by` clauses (Kotlin model). More ceremony but makes delegation explicit in the declaration. |
| Implicit promotion (Go) | Struct embedding automatically promotes methods to the embedding type. Concise but implicit — forwarding is not visible at the definition site. |
| Manual forwarding | No delegation support. Every forwarding method written by hand. Verbose but zero magic. |

## Open Questions

1. Should class delegation compose with trait bounds — can a delegate be constrained by `where Inner: Trait`?
2. Should delegation support chaining (delegate whose delegate is another delegate)?
3. How does delegation interact with visibility — methods of different visibility levels?

## Decision History

- **EDR-057:** Delegation accepted as StdLib — composition via `@delegate` macro. Class delegation treated as syntactic sugar over composition + manual forwarding. Property delegation provided via StdLib protocols.
- **Classification per D-03:** StdLib. Delegation is expressible via composition of existing primitives (trait implementation + method forwarding). The `@delegate` macro desugars to explicit `impl` blocks, using the existing macro system (EDR-029).
- **Note:** DELEGATION (this concept) is distinct from DELEGATE (execution policy from EDR-036). The execution `delegate` creates concurrent execution contexts; delegation pattern composes types.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
