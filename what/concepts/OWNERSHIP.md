# Ownership

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you efficiently manage memory and prevent leaks without runtime overhead or unpredictable pauses?

The evolution of memory management reveals a fundamental tension:
- **Manual management** (`malloc`/`free`) — full control, but use-after-free, double-free, and memory leaks are endemic. In C, these account for ~70% of critical CVEs.
- **Tracing garbage collection** (Java, Go, Python) — automatic safety, but stop-the-world pauses, unpredictable latency, and memory overhead (GC header per object).

The core problem: **memory safety requires automation, but GC's runtime cost is unacceptable for systems programming**. We need compile-time memory safety with no runtime overhead.

## Principles

1. **No runtime overhead** — Memory management decisions are made at compile time; there is no GC, no reference counting (by default), no runtime cost.
2. **Compile-time safety** — Use-after-free, double-free, and memory leaks are prevented by the type system and borrow checker.
3. **Single owner** — Every value has exactly one owner. Ownership can be transferred (move), but not duplicated implicitly.
4. **Temporary borrowing** — References to owned data are temporary and subject to lifetime rules.
5. **Deterministic destruction** — Destructors run at a well-defined point (end of scope or move) — no GC timing.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Allocation Policy | Determines how memory is allocated (heap, arena, static, GC) |
| Lifetime Policy | Governs how lifetimes are tracked — static borrow checking, regions, or runtime tracing |
| Mutability Policy | Controls whether borrowed references permit mutation (shared XOR mutable) |
| Destruction Policy | Defines destructor ordering and drop semantics |

## Model (What)

Ownership is a set of rules enforced by the compiler:

1. **Each value has exactly one owner** at any point in the program.
2. **Ownership can be moved** — when a value is assigned to a new variable or passed to a function, ownership transfers. The old binding is invalidated.
3. **Borrowing creates references** without transferring ownership:
   - Multiple shared read references (`&T`) are allowed concurrently.
   - Exactly one mutable reference (`&mut T`) is allowed, and no shared references exist simultaneously.
4. **Lifetimes** — every reference has a lifetime, checked by the compiler to ensure no reference outlives its referent.
5. **Destructors** run when a value goes out of scope, freeing resources deterministically.

```
// Move semantics
let a = createLargeData()
let b = a           // a is moved into b; a is now invalid
// print(a)         // compile error

// Borrowing
func length(s: &String) -> Int
    return s.len()

let msg = String("hello")
let len = length(&msg)   // shared borrow
// msg is still valid here
```

## Default Strategy

Arena-based allocation with ownership and static borrow checking (Rust-style). No GC. `&T` is shared borrow, `&mut T` is exclusive mutable borrow. Lifetimes are inferred or explicitly annotated.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Reference counting | `Rc<T>` / `Arc<T>` for shared ownership with runtime counting. |
| Tracing GC | Full garbage collection for cyclic graphs (opt-in). |
| Region-based | Bump allocation with region scope for short-lived workloads. |
| Linear types | Values must be used exactly once; no implicit drop. |

## Open Questions

1. Should the language support a garbage-collected mode for non-system targets?
2. Can we support non-lexical lifetimes (NLL) by default?
3. How does ownership interact with closures — can they capture by move or by reference?
4. Should there be a way to "pin" a value to prevent moves after initialisation?
5. Can the borrow checker support partial moves from compound types?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
