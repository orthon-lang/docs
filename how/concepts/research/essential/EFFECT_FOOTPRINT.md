# Effect Footprint (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Home: `how/concepts/research/essential/` — semantic bedrock tier.
> Registered in the `how/concepts/research/README.md` tier table (2026-08-17).
>
> **Last updated:** 2026-08-17
>
> **Related:** `EXCLUSIVE_DECLARATIONS.md`, `CLOSURE_CAPTURE.md` (EDR-081),
> `FRAME_CONDITIONS.md`, `IDENTITY_BASED_SAFETY.md`,
> `../../../../what/SEMANTIC_MODEL.md` § Mutation,
> `../../../../what/SEMANTIC_MODEL.md` § Ownership,
> `OWNERSHIP_TRANSFER_OPERATOR.md`, `CONTRACTS.md` (EDR-056),
> `../../../../what/GLOSSARY.md`

## Problem

A callable contract (`requires`/`ensures`/`invariant`) states *what* a
callable guarantees, but not *what it does not touch* — and not *what it
does touch*. Orthon needs a single model for declaring a callable's effect
on state, so that the **ownership axis** (what a method does to `self`)
and the **effect axis** (whether a callable affects anything beyond its
arguments) are not conflated. Fragmented effect declarations — a method
kind here, a capture clause there, a doc annotation elsewhere — make it
harder for both humans and LLMs to reason about whether a call is safe.

## Model (What)

The **effect footprint** of a callable is the set of state it reads,
mutates, or consumes beyond its return value:

```
footprint = { self: mode?, captures: {name: mode}, globals: {name: mode} }
mode ∈ {read, mutate, consume}
```

One operation — read / mutate / consume — applies uniformly to every piece
of state a callable touches. The state is partitioned by **origin**, and
each origin has its own declaration position:

| State origin | Declaration position | read | mutate | consume |
|--------------|----------------------|------|--------|---------|
| `self` (methods only) | declaration kind | `fun` / `trans` | `proc` | `into` |
| capture | `using` clause | body reads the binding | body mutates the binding | (move) |
| global / I/O | `@modifies` (or effect system) | — | declared | — |

```orthon
fun  len() -> Int            # footprint = { self: read }
proc append(x: T)            # footprint = { self: mutate }
trans sorted() -> List<T>    # footprint = { self: read }  + returns a new value
into iter() -> Iterator<T>   # footprint = { self: consume }

|impure| total += x using total   # footprint = { captures: { total: mutate } }

fun sqrt(x: Float) -> Float  # footprint = {} — pure

fun len() -> Int
    log.trace("len")         # footprint = { self: read, globals: { log: mutate } }
    self.items.len()
```

## The Two Axes (orthogonal, not alternatives)

The footprint model rests on two orthogonal axes that were previously
conflated:

1. **Ownership axis (`self`).** What a method does to its receiver:
   read / mutate / consume. Encoded in the declaration kind
   (`fun`/`proc`/`trans`/`into`). Applies to methods only. This axis is
   what the no-borrow-checker ownership model depends on: only `proc`
   requires exclusive access to `self`; `fun`/`trans` allow shared
   access; `into` transfers ownership.
2. **Effect axis (outward).** Whether a callable touches anything besides
   `self` and its arguments: captures (`using`) and globals/I/O
   (`@modifies`). Applies to every callable, including methods.

The seed of this separation already exists in `IDENTITY_BASED_SAFETY.md`:
side effects that do not affect `self` are not `self`-mutation, and the
language may distinguish `read-only(self)` from `pure(world)` — the two
are orthogonal to memory safety.

## Purity

A callable is **pure** if and only if its footprint contains no `mutate`
or `consume` entry — regardless of whether the target is `self`, a
capture, or a global:

| Callable | Footprint | Pure? |
|----------|-----------|-------|
| `fun len()` | `self: read` | ✅ |
| `proc append()` | `self: mutate` | ❌ |
| `\|x\| total += x using total` | `captures: { total: mutate }` | ❌ |
| `fun sqrt(x)` | `{}` | ✅ |
| `fun len()` + `log.trace` | `globals: { log: mutate }` | ❌ |

## Callables without `self`

Lambdas, anonymous functions, and free functions have no receiver, so the
ownership axis is empty for them: the four declaration kinds do not
classify them. Their external link is expressed by `using` (captures) and
`@modifies` (globals/I/O). A lambda with no `using` clause is pure by
default.

## Why `self` gets a declaration kind

`self` is the implicit dispatch receiver, and its ownership fate
(share / mutate / consume) is the safety-critical contract of the
no-borrow-checker model. A binary pure/impure split cannot distinguish
`consume` from `borrow` — a `fun` and an `into` are both "pure" in the
mutation sense, yet one leaves the receiver alive and the other destroys
it. The kind exists because the receiver is the most common and most
safety-critical target of effects.

## Relation to Existing Mechanisms

- **Kinds** (`EXCLUSIVE_DECLARATIONS.md`, `SEMANTIC_MODEL.md` § Mutation)
  are the frame condition on `self`.
- **`using`** (`CLOSURE_CAPTURE.md`, EDR-081) is the frame condition on
  captures.
- **`@modifies`** (`FRAME_CONDITIONS.md`, Alternative B) is the frame
  condition on globals/I/O. A full effect system (Alternative C) is
  deferred to v0.2+.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Ownership Policy | Declaration kinds govern the receiver mode (share / mutate / consume) |
| Mutability Policy | `proc` requires exclusive access; `fun`/`trans`/`into` do not mutate |
| Documentation Policy | Defines `@modifies` annotation conventions for globals/I/O |
| Lint Policy | Optional lint: warn on suspected non-local mutation in a pure callable |

## Open Questions

1. Should `@modifies` be enforced by a lint rule, or remain purely
   documentary in v0.1?
2. Does `using`/`@modifies` interact with `require` dependency slots (does
   a function "modify" a provided context)?
3. Should the `@modifies` annotation be standardized in the doc-comment
   grammar (Phase 5, Syntax)?
4. Is a full effect system (Alternative C) the eventual v0.2+
   generalization?
5. Should the term "effect footprint" be added to `what/GLOSSARY.md`?

## See also

- [`EXCLUSIVE_DECLARATIONS.md`](EXCLUSIVE_DECLARATIONS.md) — the declaration kinds
- [`CLOSURE_CAPTURE.md`](CLOSURE_CAPTURE.md) — `using` slots (EDR-081)
- [`FRAME_CONDITIONS.md`](../deferrable/FRAME_CONDITIONS.md) — `@modifies` for globals/I/O
- [`IDENTITY_BASED_SAFETY.md`](IDENTITY_BASED_SAFETY.md) — `read-only(self)` vs `pure(world)`
- [`../../../../what/SEMANTIC_MODEL.md`](../../../../what/SEMANTIC_MODEL.md) § Mutation — receiver kinds
- [`OWNERSHIP_TRANSFER_OPERATOR.md`](OWNERSHIP_TRANSFER_OPERATOR.md) — use-site `$` for arguments/bindings
- [`../../../../what/concepts/CONTRACTS.md`](../../../../what/concepts/CONTRACTS.md) — `requires`/`ensures`/`invariant` (EDR-056)
