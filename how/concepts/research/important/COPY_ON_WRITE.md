# Copy-on-Write Ownership

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

How does a language manage memory safely without forcing all programmers through a borrow checker learning curve?

Existing approaches each have drawbacks:
- **Borrow checking** (Rust) — compile-time memory safety, but a steep learning curve. Ownership, lifetimes, borrowing, and NLL (non-lexical lifetimes) take significant time to master. Simple patterns (linked lists, graphs, self-referential structs) require advanced techniques.
- **Tracing GC** (Java, Go, Python) — automatic, but runtime overhead, unpredictable pauses, and no deterministic destruction.
- **Reference counting** (Swift, Python) — automatic, but cycle leaks and performance overhead on every reference operation.

The core problem: **memory safety should be automatic for common patterns** and explicit only when the programmer deliberately chooses shared mutable state.

## Principles

1. **Value semantics by default** — Assignment copies by value. No hidden sharing.
2. **Copy-on-write (CoW)** — Standard collections (lists, maps, sets) use CoW: assignment is cheap (shares data), mutation triggers a clone only if the data is shared.
3. **No borrow checker** — Ownership tracking is not part of the language. Safety comes from CoW semantics, not from linear type analysis.
4. **Explicit sharing** — The `shared` keyword makes a value's identity visible. Shared values are reference-counted (automatic).
5. **Concurrency safety** — `shared` values are marked as `!Send` and `!Sync` by the compiler, preventing data races in concurrent code.
6. **Deterministic destruction** — Values are destroyed at end of scope. No GC pause.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Determines whether ownership is tracked at compile time (borrow checker) or runtime (CoW / RC) |
| Mutability Policy | Governs when mutation creates a copy (CoW) vs mutates in place (shared) |
| Allocation Policy | Controls heap allocation strategy — arena, RC, or GC |
| Concurrency Safety Policy | Ensures `shared` values do not cross thread boundaries unsafely |
| Destruction Policy | Defines when destructors run (deterministic end-of-scope) |

## Model (What)

Assignment creates a logical copy, but the compiler optimises using **copy-on-write**: the underlying data is shared until one of the references mutates it.

```orthon
# Copy-on-write — cheap, no deep copy until mutation
data = [1, 2, 3]
data2 = data                    # shares the same buffer
data2.append(4)                 # CoW: data2 gets its own copy now
# data is still [1, 2, 3]
# data2 is [1, 2, 3, 4]

# Immutable binding — prevents all mutation
data3 := [1, 2, 3]             # := immutable — cannot mutate
# data3.append(4)              # compile error: data3 is immutable

# Explicit shared reference — identity is visible
shared_vec = shared [10, 20]    # shared mutable reference
v = shared_vec                  # v is a reference to the same buffer
v.append(30)                    # mutates the shared buffer
# shared_vec is now [10, 20, 30]

# Shared values cannot safely cross threads
# compiler treats shared<T> as !Send + !Sync
```

Key features:
- **`name = value`** — mutable binding, value semantics with CoW.
- **`name := value`** — immutable binding, no mutation allowed.
- **`shared value`** — explicitly shared mutable reference. RC-managed.
- **`shared<T>` is `!Send + !Sync`** — the compiler prevents data races by disallowing `shared` in concurrent contexts (or requiring explicit synchronisation).

### When does CoW trigger?

The compiler tracks the **reference count** of each value at runtime (or statically when possible). Mutation triggers a clone when:
1. The value has more than one live reference.
2. The binding is mutable (`=`).
3. Immutable bindings (`:=`) never trigger CoW — they reject mutation at compile time.

### Explicit sharing with `shared`

The `shared` keyword makes shared identity syntactically visible:

```orthon
# Without shared — value semantics
a = [1, 2, 3]
b = a              # logical copy, CoW internally
b[0] = 99          # only b changes

# With shared — reference semantics
a = shared [1, 2, 3]
b = a              # b is a reference to a
b[0] = 99          # a changes too
```

## Default Strategy

Copy-on-write for all standard collections. Primitive types (Int, Float, Bool, String) use value semantics with inline storage. `shared` uses reference counting with cycle detection (trial deletion or backup GC for cycles only).

## Alternative Strategies

| Strategy | Description |
|---|---|
| Borrow checker (Rust) | Compile-time ownership tracking. Zero runtime overhead, full memory safety. Higher cognitive load. |
| Tracing GC (Java, Go) | Stop-the-world or concurrent GC. No ownership annotations, but runtime overhead and pauses. |
| Automatic Reference Counting (Swift) | RC with compile-time optimisations. Cycle leaks need weak references. |
| Region-based (C++ arenas) | Manual region allocation. Deterministic, no overhead, but programmer-managed lifetimes. |

## Open Questions

1. Can CoW be optimised to purely static decisions in most cases? (Escape analysis, uniqueness inference.)
2. How to handle graph structures and cycles with CoW + shared?
3. Should `shared` use RC with cycle detection or defer to a backup GC?
4. Interaction with FFI — how does CoW data map to C memory layouts?
5. Should there be an `unsafe` escape hatch for bypassing CoW?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/EXECUTION_MODEL.md`
- [ ] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
- [ ] Other: `how/concepts/research/OWNERSHIP.md`, `how/concepts/research/MUTABILITY.md`, `how/concepts/research/ALLOCATION.md`, `how/concepts/research/CONCURRENCY.md`
