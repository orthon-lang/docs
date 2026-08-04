# Frame Conditions

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-08-04
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

A function contract (`requires`/`ensures`/`invariant`) says *what* a
function guarantees, but not *what it does not touch*. In Separation
Logic, this is the **frame condition**: the set of locations a
computation is allowed to modify. Without a frame condition, a
postcondition like `ensures result * result ≈ x` says nothing about
whether the function also mutated a global logger, a database, or a
module-level cache. For modular reasoning — and for an LLM reasoning
about whether a call is safe — the *absence* of mutation is as important
as the presence of a guarantee.

The question this document resolves: **are frame conditions part of the
Orthon language (a keyword), or part of the public documentation (a doc
annotation)?** The working answer is: *documentation*, because the
declaration kinds (`fun`/`proc`/`new`) already cover the common case.

## What the Declaration Kinds Already Guarantee

Orthon's three mutually exclusive declaration kinds (see
[`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Mutation) are
already a *coarse frame condition* for `self`:

| Kind | Effect on `self` | Frame condition on `self` |
|------|------------------|---------------------------|
| **`fun`** | Read-only — never mutates `self` | `modifies nothing` (on `self`) — implicit |
| **`proc`** | Mutates `self`; identity preserved | `modifies self` — implicit |
| **`new`** | Never mutates `self`; produces a distinct value | `modifies nothing` (on `self`) — implicit |

```orthon
class List:
    fun len() -> Int            # guaranteed: self not mutated
        self.items.len()

    proc append(item: Int)       # guaranteed: self mutated
        self.items.push(item)
```

For methods, the declaration kind *is* the frame condition on `self`.
The compiler already enforces it: `fun` may not mutate `self`; `proc`
may. No `modifies` keyword is needed for this case.

## The Gap: Free Functions and Non-`self` State

The declaration kinds say nothing about *other* state: global
variables, I/O, dependency slots provided via `require`, or module
state. A free function has no `self`:

```orthon
fun logAndCompute(x: Float) -> Float
    # Does this touch the global logger? The database? Both?
    requires x >= 0.0
    ensures result * result ≈ x
```

Here the frame condition is genuinely unspecified. The question is how
to make it visible.

## Design Alternatives

### Alternative A: `modifies` keyword in the signature

```orthon
fun sqrt(x: Float) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
    modifies nothing          # free function, pure
```

| Pros | Cons |
|------|------|
| Machine-checkable | Noise on 95% of functions — most are pure `fun` and would repeat `modifies nothing` |
| Part of the type signature | Overlaps with what `fun`/`proc`/`new` already state for methods |
| Compiler can verify | Adds a fourth contract clause to an already rich surface |
| | Conflicts with "minimal core" and "simplicity" |

### Alternative B: `@modifies` doc annotation (recommended)

```orthon
/// A sqrt that also touches the audit log.
/// @modifies audit_log
fun sqrt(x: Float, log: AuditLog) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
```

```orthon
/// Pure function — modifies nothing.
/// @modifies nothing
fun sqrt(x: Float) -> Float
    requires x >= 0.0
    ensures result * result ≈ x
```

| Pros | Cons |
|------|------|
| Zero syntax cost; `modifies nothing` reads naturally in prose | Not machine-checkable by the type system |
| Only special cases need an annotation — no noise on the common path | Drift risk (doc vs. code), like any documentation |
| Follows precedents: JSDoc `@modifies`, Swift DocC `@mutates`, Dafny's documentation comments | Optional lint rule can partially close the gap |

### Alternative C: Effect system

A full effect system (e.g., "this function has the `io` effect, the
`mutates(global)` effect") generalizes frame conditions to all effects.

| Pros | Cons |
|------|------|
| Most expressive — covers I/O, nondeterminism, mutation | High complexity; contradicts minimal core |
| Static, machine-checked | Major Type System burden; deferred to v0.2+ at best |

## Recommendation

**Alternative B: `@modifies` as a doc annotation**, plus an optional
lint rule that warns when a `fun`'s body appears to mutate non-local
state. Rationale:

1. The declaration kinds already provide the machine-checked frame
   condition for `self` — the common case.
2. `modifies nothing` in the *signature* is noise; in *documentation* it
   is meaningful, especially for special cases (pure free functions,
   functions touching globals).
3. Orthon's Explicitness principle is satisfied at the *call site* by
   the declaration kinds; the doc annotation covers the residual cases
   without a language cost.

For LLM generation, the doc annotation is read as intent by the LLM,
matching how `requires`/`ensures` already serve as intent carriers.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Contract Enforcement Policy | Frame conditions as documentation are not enforced at runtime; optional lint verification |
| Documentation Policy | Defines `@modifies` annotation conventions for public contracts |
| Lint Policy | Optional compiler lint: warn on suspected non-local mutation in `fun` |

## Open Questions

1. Should `@modifies` be enforced by a lint rule, or remain purely
   documentary in v0.1?
2. Does `@modifies` need to interact with `require`/`using` dependency
   slots (does a function "modify" a provided context)?
3. Should the `@modifies` annotation be standardized in the doc-comment
   grammar (Phase 5, Syntax)?

## Decision History

Initial research — no decisions recorded yet.

## See also

- [`CONTRACTS.md`](../../../what/concepts/CONTRACTS.md) — `requires`/`ensures`/`invariant` (EDR-056)
- [`SEMANTIC_MODEL.md`](../../../what/SEMANTIC_MODEL.md) § Ownership (Formal foundation) — Separation Logic grounding for frame conditions
- [`CORRECTNESS_BY_CONSTRUCTION.md`](../important/CORRECTNESS_BY_CONSTRUCTION.md) — why frames matter for modular correctness
- [`EXCLUSIVE_DECLARATIONS.md`](../essential/EXCLUSIVE_DECLARATIONS.md) — the `fun`/`proc`/`new` model
- [`REQUIRE_USING_DEPENDENCY_SLOTS.md`](../../../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md) — `require`/`using` dependency slots
