# Hypothesis: Bounds in Angle Brackets — `where` Clause Elimination

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the FUNCTIONS concept review, trailing-clause discussion
> (2026-08-07). Syntax question — Phase 5 scope. Further revision of
> EDR-086 (bound-first `as` bounds).
>
> **See also:** `what/SYNTAX.md` (Phase 5 placeholder),
> `what/concepts/GENERICS.md` (EDR-024 / EDR-086),
> `what/concepts/TRAITS.md` (EDR-019), `what/GLOSSARY.md` § Trait Bound,
> `FUNCTION_RETURN_SYNTAX.md`, `FUNCTION_ARGUMENT_SYNTAX.md`,
> `FUNCTION_RETURNING_FUNCTION.md`, `CLOSURE_CAPTURE.md`,
> `how/DESIGN_PRINCIPLES.md` (Parsimony, DRY, Uniformity, Semantic Purity)

## Problem

Two related issues:

1. **The signature tail is crowded.** After the parameters the tail can hold
   `require ...` (EDR-081), `where ...` (GENERICS/EDR-086) and `-> Return`.
   No canonical order is defined anywhere — `SYNTAX.md` (Phase 5) and
   `CONFLICT_REGISTRY.md` (Phase 6) are placeholders. Two accepted examples
   already disagree: EDR-081 puts `require` before `->`, GENERICS puts
   `where` after the parameters.
2. **`where` duplicates bounds already expressible inline.** EDR-086
   established the bound-first inline form `<Iterator as T>`. A separate
   `where` clause is a second home for the same concept (a DRY violation —
   "no construct twice").

Proposal: extend the accepted bound-first inline form to conjunctions,
move **all** bounds into `<>`, and **eliminate the `where` clause** from the
language.

## Examples

```orthon
# Function — bounds inline, no where
fun process<Hash + Eq as T>(T value, T other)

# Generic type
type Pair<Hash as K, V>

# Class and method — uniform at every level
class UserService<Eq as T> require Database db
    fun find<Hash as K>(key: K) -> Option[User]

# Blanket impl — bound moves into <>
impl<Display as T> Printable for T
```

Reading mnemonic: `<>` says "process T, but note that T is (at least)
`Hash + Eq`". The bound is visible at the parameter itself, not at the end
of a long signature.

## Where `where` currently lives (inventory)

| Location | Current form | New home |
|----------|--------------|----------|
| Generic function signatures | `fun process<T>(...) where T as Hash + Eq` | `fun process<Hash + Eq as T>(...)` |
| Generic types | `type Pair<T> where T as Hash` | `type Pair<Hash as K, V>` |
| Blanket implementations | `impl<T> Printable for T where T as Display` | `impl<Display as T> Printable for T` |
| Negative bounds (open question in TRAITS) | `where T as !FixedSize` | Marker trait: `<NonFixedSize as T>` (opt-in — see caveat) |
| Associated-type bounds | `where T::Item: Display` | Named alias in module header, or deferred to v0.2 (see caveat) |

## Implications for Orthon

### Positive

- **One home for type parameters and their contracts.** `<>` becomes the
  single place for both; uniform across functions, types, classes, methods
  and impls (Uniformity principle).
- **The `,` ambiguity disappears.** EDR-086 flagged that in a `where` clause
  `,` had to distinguish "bounds on one parameter" from "constraints on
  different parameters". Inside `<>` the structure is fixed: `,` separates
  parameters, `+` conjoins bounds within one parameter — no ambiguity.
- **Parsimony / DRY.** One keyword fewer; the tail is freed for
  `require ... -> Return` (or prefix return), resolving the
  trailing-clause-order spec gap.
- **LLM generability.** The bound is read immediately at the parameter, so
  an LLM does not need to scan to the end of the signature.

### Negative / Risks

- **Bound-first order is the accepted direction.** `<Hash + Eq as T>` reads
  against the param-first intuition `<T as Hash + Eq>`; EDR-086 already chose
  bound-first, so this is consistent — but it must be re-affirmed.
- **Blanket-impl syntax changes.** `impl<Display as T> Printable for T`
  reads less naturally than `where`; needs an LLM generability check.
- **Negative bounds become opt-in marker traits only.** `<NonFixedSize as T>`
  requires a concrete marker trait that types explicitly implement. Automatic
  complement ("any T that is NOT `FixedSize`") is **not** expressible this
  way — that would need the compiler to compute type-level negation. v0.1 has
  no such task, so this is acceptable now, but it must be recorded as a
  boundary.
- **Associated-type bounds are the weakest point.** `<Eq as T::Item>` does
  not read well inside `<>`. Rust/C#/Swift all kept `where`-style clauses
  precisely for this case. The proposed mitigation — a named alias declared
  in the module header/imports, then used as a single bound — requires every
  relationship to be pre-named (a naming cost), and is close to the named
  bound-bundle idea already rejected for signature types. Alternative:
  defer associated-type bounds to v0.2.
- **Ripple.** Eliminating `where` revises the `where`-clause decision of
  EDR-086 (accepted 2026-08-07). Must update `GENERICS.md`, `TRAITS.md`,
  `GLOSSARY.md` (§ Trait Bound), `CORE_CONCEPTS.md`, and every generics
  example.

## Open Questions

1. Can blanket-impl bounds always move into `<>`, including multi-bound
   forms (`impl<Hash + Eq as T> Printable for T`)?
2. Negative bounds: is an opt-in marker trait (`<NonFixedSize as T>`)
   sufficient permanently, or will Orthon ever need automatic complement
   (requiring `where` or a new mechanism)?
3. Associated-type bounds: is a named alias in the module header a workable
   v0.1 mechanism, or should they be deferred to v0.2?
4. Interaction with the `comptime T: type` generics description in
   `GLOSSARY.md` — does the `<>` form survive?

## Next Step

Phase 5 syntax decision. If adopted, supersede the `where`-clause portion of
EDR-086. Coupled with `FUNCTION_RETURN_SYNTAX.md`, `FUNCTION_ARGUMENT_SYNTAX.md`,
`FUNCTION_RETURNING_FUNCTION.md`, `CLOSURE_CAPTURE.md`, and the lambda
syntax (`notes/code-block-semantics.md`).
