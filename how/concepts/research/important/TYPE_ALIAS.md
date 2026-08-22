# Type Alias (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Home: `how/concepts/research/important/` — semantic hypothesis (important tier).
> Registered in the `how/concepts/research/README.md` tier table (2026-08-17).
>
> **Last updated:** 2026-08-21
>
> **Related:** `../essential/FUNCTIONS.md`,
> `STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`,
> `../essential/CLOSURE_CAPTURE.md` (EDR-081),
> `../../../../what/PRIMITIVE_BLOCKS.md`,
> `../../../../what/SEMANTIC_MODEL.md`,
> `../../../../notes/code-block-semantics.md`,
> `../../../syntax/FUNCTION_RETURNING_FUNCTION.md` (function-type notation),
> `../deferrable/USING_DIRECTIVES.md` (import aliasing — distinct concept),
> `../deferrable/REFINEMENT_TYPES.md` (invariant-bearing types — distinct concept),
> `../../../decision_records/architecture/EDR-019-traits.md` (no trait inheritance — motivates bound alias),
> `../../../decision_records/architecture/EDR-086-generics-syntax-revision.md` (`+` conjunction operator, `as` bound syntax),
> `../../../../what/concepts/GENERICS.md` (multi-bound generics — readability consumer),
> `../../../../what/concepts/TRAITS.md` (Principle #6: no inheritance)

## Problem

Orthon needs a way to give an existing type a second name — a **type synonym** —
for readability and for naming otherwise-verbose types. Without it, domain names
(`UserId`) and long signatures (function types) must be written in full at every
use site.

The term `alias` risks overloading: it is used in Orthon research for at least
three distinct ideas that must not be conflated:

| Idea | Meaning | Distinct concept? |
|------|---------|-------------------|
| **Type synonym** | Same type, new name; fully interchangeable | THIS hypothesis |
| **Import aliasing** | `import foo as bar` — rename at import time | Yes — `USING_DIRECTIVES.md` |
| **Refinement type** | `alias UserId = Int ..(where x > 0)` — invariant-bearing, NOT interchangeable | Yes — `REFINEMENT_TYPES.md` |

This hypothesis covers only the first: `alias` as a type synonym (syntactic sugar).

## Model (What)

`alias Name = Type` declares that `Name` is an **alias** for `Type`: the two are
fully interchangeable, and the compiler sees through the alias (no new type
identity). Consistent with `STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`, which contrasts
`alias` (same type, new name) with `struct` (distinct nominal type).

```orthon
alias UserId = Int                     # UserId ≡ Int — no type-safety boundary
alias LogCommand = (msg: String): Void # function-type synonym
```

Because an alias introduces no new type, it is **syntactic sugar**:
- It decomposes to the underlying type at every use site (primitive decomposition
  check passes — no new primitive).
- It is classified as **sugar** by the Decision Pipeline (Stage 2 exit path), not
  as a new Level-1 construct; no full Concept Design Review is required.

### Function-type synonym

`FUNCTION_RETURNING_FUNCTION.md` motivates naming a function type so that a
function-returning-function reads cleanly:

```orthon
alias SubFunc = (x: Int): String        # named function type
fun make_adder(n: Int): SubFunc         # no nested arrows / double colon
```

The positional form `Fn[Int, String]` was rejected (2026-08-17): it does not
reveal which parameter is input and which is output. The named form
`(x: Int): String` is the base notation; `alias` is an optional name on top.

### Bound conjunction alias

A second use case for `alias` is naming a **trait bound conjunction** — a set
of trait bounds joined by `+` (the conjunction operator fixed by
[EDR-086](../../../decision_records/architecture/EDR-086-generics-syntax-revision.md)
#3: "`+` combines multiple bounds on one parameter"). Without a name, a
generic function with three or more bounds becomes unreadable:

```orthon
# Inline — readable at two bounds, painful at three or more
fun first<Hash + Eq as T>(T items) -> Option<T>
fun process<Hash + Eq + Ord + Clone as T>(T value, T other) -> Int
```

A bound conjunction alias gives the set a name:

```orthon
alias Comparable = Hash + Eq
alias TotalOrd   = Eq + Hash + Ord + Clone

fun first<Comparable as T>(T items) -> Option<T>
fun process<TotalOrd as T>(T value, T other) -> Int
```

Semantics:

- **Transparent** — the alias introduces no new trait, no new type identity,
  and no nominal boundary. The compiler sees through it to the underlying
  bound conjunction at every use site, exactly as for type synonyms.
- **RHS grammar** — the right-hand side must be a valid bound conjunction:
  one or more trait names joined by `+`. A single trait on the RHS is
  permitted (`alias Key = Hash`) and is a trivial (one-element) conjunction.
- **Use position** — a bound alias may appear wherever a bound is expected:
  inline in `<>` (`<Comparable as T>`) and in `where` clauses
  (`where T as Comparable`). It desugars to the underlying conjunction
  (`where T as Hash + Eq`).
- **No new mechanism** — this is sugar over the existing `+` conjunction
  operator fixed by EDR-086. No new delimiter is introduced; `&`, `|`, and
  `,` remain reserved for their existing roles (intersection types, union
  types, and parameter separation respectively — see EDR-045 and EDR-086).

#### Why alias, not a composing trait

Orthon rejects trait inheritance
([EDR-019](../../../decision_records/architecture/EDR-019-traits.md),
Principle #6: "No inheritance — Traits do not extend other traits to form
hierarchies. Trait bounds express requirements... This is composition of
constraints, not inheritance."). A Swift-style `trait Comparable: Hash + Eq`
that other traits extend is therefore not available. Naming a bound set
*requires* the alias path — there is no "declare a new trait that composes
the others" route.

#### Negative bounds are a separate concern

The `+` operator is a **conjunction** ("must satisfy all of"), not
arithmetic. The presence of `+` does not imply a `-` (subtraction / negative
bound) operator. Negative bounds (`where T as !Hash`) are a separate,
deferred open question in
[`GENERICS.md`](../../../what/concepts/GENERICS.md) OQ4 (v0.2+) and are
orthogonal to bound conjunction aliases.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type System Policy | Alias introduces no new type identity; transparency |
| Documentation Policy | Alias is documentation intent only — no compiler-enforced boundary |

## Open Questions

1. **Alias in the signature tail** (resolved 2026-08-17): **top-level
   declaration is the primary form** — it solves a broader set of problems
   (reuse across many use sites, one alias per signature independent of
   position):
   ```orthon
   alias UserId = (x: Int): String   # top-level declaration — primary
   ```
   The inline tail position is **optional sugar only**, not a semantic
   alternative:
   ```orthon
   (x: Int): String alias UserId     # inline — sugar, optional
   ```
   Open sub-question: whether the inline form is worth keeping at all in v0.1,
   or deferred until the top-level form is proven.

2. **RHS syntax for function types**: is `(x: Int): String` the canonical RHS
   (named parameters preserved), and does it couple to
   `FUNCTION_RETURN_SYNTAX.md` / `FUNCTION_ARGUMENT_SYNTAX.md`?

3. **LLM generability** (resolved 2026-08-17): **top-level `alias` form — Pass**
   (`LLM_GENERABILITY_GATE`, EDR-014). It follows the familiar type-alias
   pattern (Python `X = Callable[...]`, Kotlin `typealias`, Rust `type`), adds
   no new grammar position, resolves like any name (full diagnostic coverage),
   and is strategy-independent. The inline tail form **Flags** two criteria
   (predictable generation, no hallucination surface) — the `alias` position
   at the end of a signature is non-standard and competes with `using` /
   `invariant` clauses for the tail. This favours deferring or rejecting the
   inline form in v0.1.

4. **Interaction with `using`/capture**: can a captured function type be aliased,
   or does the `using` clause live outside the alias (creation-time resolution)?

5. **Opaque type alias**: should Orthon consider an opaque form (Scala 3
   `opaque type`, OCaml private types) — a middle ground between `alias`
   (transparent) and `struct` (nominal)? (See `STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`
   § Opaque type aliases.)

6. **Bound conjunction alias — transitivity**: can a bound alias reference
   another bound alias on its RHS?
   ```orthon
   alias A = Hash + Eq
   alias B = A + Ord      # does this desugar to Hash + Eq + Ord?
   ```
   If yes, the compiler must flatten transitively. If no, the RHS of a
   bound alias may only name concrete traits, not other aliases.

7. **Bound conjunction alias in `where` clauses**: the inline `<>` form is
   the primary use site. Should `where T as Comparable` also desugar to
   `where T as Hash + Eq`? Expected: yes, for consistency — but verify
   against the `where`-clause grammar in
   [`EDR-086`](../../../decision_records/architecture/EDR-086-generics-syntax-revision.md).

8. **Schema Provider exposure**: when an LLM queries a generic function's
   contract, should the Schema Provider expose the **alias name**
   (`Comparable`) or the **expanded bound set** (`Hash + Eq`)? Working
   hypothesis: the expanded set is more useful for an LLM (it needs the
   concrete requirements, not an indirection to resolve). The alias name
   may be exposed as a secondary "display name" for human readability.

9. **Bound alias in `dyn` position**: can `dyn Comparable` be used as a
   trait object, given that `Comparable` is not a single trait but a
   conjunction? Coherence rules (orphan rule, EDR-019) may not extend
   cleanly to a multi-trait vtable. Working hypothesis: bound aliases are
   **bound-position only**; `dyn` requires a single concrete trait name.

10. **Single-trait RHS**: is `alias Key = Hash` (one-element conjunction)
    permitted, and if so, is it a no-op or does it carry documentation
    intent (naming a role: "the type used as a key")? Expected: permitted,
    treated as a trivial conjunction, primarily a documentation alias.

## Decision History

- **alias ≠ refinement** (2026-08-17): refinement types (`..(where ...)`)
  are a separate concept (`REFINEMENT_TYPES.md`); an invariant breaks
  interchangeability and violates one-concept-one-syntax.
- **`Fn[...]` positional form rejected for function types** (2026-08-17):
  ambiguous input/output; named form `(x: Int): String` is the base notation.
- **Effect markers do not apply to free-function aliases** (2026-08-17):
  `fun`/`proc`/`new` are the ownership axis on `self` only
  (`EFFECT_FOOTPRINT.md`); an alias RHS carries no effect marker.
- **Top-level alias is LLM-generable (Pass)** (2026-08-17): confirmed against
  `LLM_GENERABILITY_GATE` (EDR-014); inline tail form flags predictable-
  generation and hallucination-surface criteria — defer/reject inline in v0.1.
- **Bound conjunction alias proposed** (2026-08-21): extended the hypothesis
  to allow `alias Name = TraitA + TraitB + ...` on the RHS, naming a trait
  bound conjunction for use in `<>` and `where` clauses. Motivated by
  readability of 3+ trait bounds (EDR-086 inline syntax gets verbose:
  `<Hash + Eq + Ord + Clone as T>`). Trait inheritance is rejected
  ([EDR-019](../../../decision_records/architecture/EDR-019-traits.md),
  Principle #6: "No inheritance"), which closes the "declare a new
  composing trait" path and makes alias the only mechanism to name a bound
  set. Delimiter `+` is inherited from
  [EDR-086](../../../decision_records/architecture/EDR-086-generics-syntax-revision.md)
  #3 (conjunction operator); `&`, `|`, and `,` are all reserved for other
  roles (intersection types, union types, parameter separation). Negative
  bounds (`!Hash`) are a separate deferred OQ in
  [`GENERICS.md`](../../../what/concepts/GENERICS.md) OQ4 and are not
  implied by the `+` operator. Status: **hypothesis** — no EDR, no
  acceptance claim.
