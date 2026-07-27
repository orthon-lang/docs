# Copy-on-Write

## Issue (Why)

How does a language manage memory safely without forcing all programmers through a borrow checker learning curve? Existing approaches each have drawbacks:
- **Borrow checking** (Rust) — compile-time memory safety, but a steep learning curve.
- **Tracing GC** (Java, Go) — automatic, but runtime overhead and unpredictable pauses.
- **Reference counting** (Swift, Python) — automatic, but cycle leaks and performance overhead.

Memory safety should be automatic for common patterns and explicit only when the programmer deliberately chooses shared mutable state.

## Principles

1. **Value semantics by default** — Assignment copies by value. No hidden sharing.
2. **Copy-on-write (CoW)** — Standard collections use CoW: assignment is cheap (shares data), mutation triggers a clone only if the data is shared.
3. **No borrow checker** — Ownership tracking is not part of the language. Safety comes from CoW semantics.
4. **Explicit sharing** — The `shared` keyword makes a value's identity visible.
5. **Concurrency safety** — `shared` values are `!Send` and `!Sync`, preventing data races in concurrent code.
6. **Deterministic destruction** — Values are destroyed at end of scope.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Determines whether ownership is tracked at compile time (borrow checker) or runtime (CoW / RC) |
| Mutability Policy | Governs when mutation creates a copy (CoW) vs mutates in place (shared) |
| Allocation Policy | Controls heap allocation strategy |
| Destruction Policy | Defines when destructors run (deterministic end-of-scope) |

## Model (What)

Assignment creates a logical copy, but the compiler optimises using **copy-on-write**: the underlying data is shared until one of the references mutates it:

```orthon
# Copy-on-write — cheap, no deep copy until mutation
data = [1, 2, 3]
data2 = data                    # shares the same buffer

# Mutation triggers copy if shared
data2[0] = 99                   # clone happens here if data2 is shared
```

### Explicit sharing with `shared`

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

Copy-on-write for all standard collections. Primitive types (Int, Float, Bool, String) use value semantics with inline storage. `shared` uses reference counting with cycle detection.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Borrow checker (Rust) | Compile-time ownership tracking. Zero runtime overhead, full memory safety. Higher cognitive load. |
| Tracing GC (Java, Go) | Stop-the-world or concurrent GC. No ownership annotations, but runtime overhead and pauses. |
| Automatic RC (Swift) | RC with compile-time optimisations. Cycle leaks need weak references. |
| Region-based (C++ arenas) | Manual region allocation. Deterministic, no overhead, but programmer-managed lifetimes. |

## Open Questions

1. Can CoW be optimised to purely static decisions in most cases via escape analysis?
2. How to handle graph structures and cycles with CoW + shared?
3. Interaction with FFI — how does CoW data map to C memory layouts?

## Decision History

- **EDR-061:** Copy-on-Write accepted as StdLib / Implementation Strategy — CoW is a memory optimisation, not new semantics. The value semantics and explicit sharing model are specified in the SEMANTIC_MODEL; CoW is the DEFAULT_STRATEGY's chosen mechanism.
- **Classification per D-03:** StdLib / Implementation Strategy. CoW is an implementation technique for value-semantics collections. The programmer does not write CoW-specific code — they write value-semantics code and the compiler/runtime optimises via CoW. Cross-reference VALUE_SEMANTICS (SEMANTIC_MODEL.md).

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/DEFAULT_STRATEGY.md`
