# Mutability

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

Mutation is the root cause of most difficult-to-trace bugs in software:
unexpected state changes, aliasing issues, data races, and violated
invariants. The core problem: **how does a language make mutation safe
and visible** while still allowing it where it is the natural solution
(e.g., counters, accumulators, in-place updates)?

Immutable-by-default languages (Rust, Clojure, Elixir) have shown that
making immutability the default substantially reduces bug rates. However,
too-strict immutability can force awkward patterns (e.g., building a new
collection for every modification). The language needs a balanced model:
immutable by default, with explicit, safe mutation when needed.

## Principles

1. **Immutable by default** — All data bindings are immutable unless
   explicitly declared mutable. This is the default because it is the
   safer, more predictable choice.
2. **Explicit mutation** — Mutation must be syntactically visible.
   The `mut` keyword (or equivalent) must appear at both the declaration
   and the mutation site.
3. **Aliasing control** — Mutable data must be protected against
   aliasing-induced bugs. The compiler tracks whether a value can be
   mutated through multiple references.
4. **No hidden mutation** — No implicit mutation via method calls,
   property setters, or operator overloading that is not visible at
   the call site.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Mutability Policy | Defines default mutability (immutable) and the `mut` keyword mechanism |
| Lifetime Policy | Controls how long mutable references live; prevents dangling mutable pointers |
| Ownership Policy | Governs how mutable access is transferred or shared |
| Concurrency Policy | Determines whether mutation is permitted across threads |

## Model (What)

All data bindings are immutable by default. Mutation is enabled with the
`mut` keyword at the binding site.

```orthon
# Immutable (default)
count = 42
count = 43        # Error: cannot assign to immutable binding

# Mutable (explicit)
mut counter = 0
counter += 1      # OK

# Mutable parameter
fn increment(mut value: Int) -> Int
    value += 1
    value

# Mutable reference (ownership transfer for mutation)
fn update(mut data: &mut Data)
    data.field = new_value
```

## Default Strategy

The compiler enforces immutability by default through the type system.
Mutable bindings are tracked and their mutation rights are verified at
compile time. No runtime checks are needed — all mutation safety is
static.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Full immutability | No mutation permitted anywhere | Pure functional style; embedded safety-critical |
| Reference counting with COW | Copy-on-write for shared mutable data | When mutation needs to coexist with sharing |
| Transactional memory | Mutation via atomic transactions | Concurrent mutation across threads |

## Open Questions

1. Should interior mutability (e.g., `Cell`/`RefCell` pattern from Rust)
   be available? If so, how is it expressed syntactically?
2. How does mutability compose with closures — can a closure mutate its
   captured environment?
3. Should there be a distinction between `mut` (mutable binding) and
   `&mut` (mutable reference), or should a single `mut` keyword cover both?
4. How does mutability interact with the ownership/borrowing model?

## Decision History

- **Immutable by default:** Adopted. Follows the principle that safer
  defaults reduce bug rates. Alternatives considered: mutable by default
  (rejected: contradicts explicit semantics), opt-in immutability via
  `const` (rejected: weaker guarantees, less adoption).
- **`mut` keyword:** Adopted for visibility. Alternatives considered:
  `var` keyword (rejected: different connotation in other languages),
  type-based mutability (rejected: violates Explicitness).
- **Deferred to Phase 3:** Interior mutability design, closure mutation
  rules, full borrowing/mutability interaction specification.
