# Type Alias (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Home: `how/concepts/research/important/` — semantic hypothesis (important tier).
> Registered in the `how/concepts/research/README.md` tier table (2026-08-17).
>
> **Last updated:** 2026-08-17
>
> **Related:** `../essential/FUNCTIONS.md`,
> `STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`,
> `../essential/CLOSURE_CAPTURE.md` (EDR-081),
> `../../../../what/PRIMITIVE_BLOCKS.md`,
> `../../../../what/SEMANTIC_MODEL.md`,
> `../../../../notes/code-block-semantics.md`,
> `../../../syntax/FUNCTION_RETURNING_FUNCTION.md` (function-type notation),
> `../deferrable/USING_DIRECTIVES.md` (import aliasing — distinct concept),
> `../deferrable/REFINEMENT_TYPES.md` (invariant-bearing types — distinct concept)

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
