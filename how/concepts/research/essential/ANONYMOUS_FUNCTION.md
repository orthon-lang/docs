# Anonymous Functions (Hypothesis)

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the FUNCTIONS concept review and the Phase 5 syntax inventory.
> Home: `how/concepts/research/essential/` — semantic hypothesis (essential tier).
> Registered in the `how/syntax/` decision queue as coupled-by-pointer (EDR-087).
>
> **Last updated:** 2026-08-16
>
> **Related:** `FUNCTIONS.md`, `CLOSURE_CAPTURE.md`,
> `EXCLUSIVE_DECLARATIONS.md`, `../deferrable/ASYNC_LAMBDA.md`,
> `../../../DESIGN_PRINCIPLES.md` (Orthogonality, Explicitness),
> `../../../../what/SEMANTIC_MODEL.md` § Mutation,
> `../../../../notes/code-block-semantics.md` (lambda as expression),
> `../../../syntax/FUNCTION_RETURNING_FUNCTION.md` (cascade / `->` overload)

## Problem

`FUNCTIONS.md` (essential) establishes functions as **first-class values** with
two declaration forms — named functions and anonymous closures — and the
"Named before anonymous" principle. The anonymous form, however, carries
several open questions that cut across semantics, terminology, and syntax:

1. How does an *anonymous function* relate to the terms *lambda* and
   *closure*? Is the distinction terminological, syntactic, or semantic?
2. Do the three effect declaration kinds — `fun` / `proc` / `new` from
   `EXCLUSIVE_DECLARATIONS.md` and `SEMANTIC_MODEL.md` § Mutation — apply to
   anonymous (and free) functions, or only to methods with `self`?
3. Can an anonymous function return another function (a cascade / currying),
   and is nesting depth limited or arbitrary?
4. Can an anonymous function be assigned to a variable and subsequently
   aliased?

This document records the hypothesis and the open questions; it makes no
acceptance claim. Syntax decisions belong to Phase 5 (`SYNTAX_PIPELINE.md`,
EDR-087).

## Examples from Other Languages

| Language | Anonymous function (lambda) | Notes |
|----------|------------------------------|-------|
| **Python** | `lambda x: x + 1` | Single expression only; assignable (`f = lambda ...`); no statements |
| **JavaScript** | `x => x + 1`, `(x, y) => { ... }` | First-class; assignable; closures returned by functions |
| **Rust** | `\|x\| x + 1` | Closures; capture by reference / move (`Fn`/`FnMut`/`FnOnce`) |
| **Kotlin** | `{ x -> x + 1 }` | Trailing-lambda; assignable to `val` |
| **Java** | `x -> x + 1` | Lambda is a single-abstract-method instance; type is nominal |
| **Swift** | `{ (x: Int) -> Int in x + 1 }` | Closure; explicit capture lists (`[weak self]`) |
| **Haskell** | `\x -> x + 1` | Curried by default — functions returning functions are the norm |

Common pattern: every first-class-function language lets an anonymous function
be assigned to a name, and most let a function return another function. The
differences lie in capture semantics, type treatment, and notation — all open
for Orthon.

## Implications for Orthon

- **First-class, assignable.** `FUNCTIONS.md` already shows an anonymous
  closure assigned to a variable (`multiply = fn (a: Int, b: Int) -> Int
  a * b`). Assignment is the mechanism that gives an anonymous function a
  name without a declaration.
- **Terminology.** `notes/code-block-semantics.md` treats a *lambda
  expression* (`|x| expr`) as an *anonymous function* in expression form.
  Orthon needs a settled stance for `GLOSSARY.md`: is *lambda* a syntactic
  form of *anonymous function*, a synonym, or distinct? Is a *closure* an
  anonymous (or named) function that captures?
- **Effect markers.** `EXCLUSIVE_DECLARATIONS.md` presents `fun` / `proc` /
  `new` as method declaration kinds operating on `self`. Whether these extend
  to free and anonymous functions — and whether the marker is explicit or
  inferred from the body (the category inference `ASYNC_LAMBDA.md` proposes
  for async lambdas) — is open.
- **Cascade / nesting.** `how/syntax/FUNCTION_RETURNING_FUNCTION.md`
  documents the notation problem: `->` is overloaded (function type, return
  annotation, lambda body), and function-returning-function notation nests
  arrows (`(Int) -> (Int) -> Int`). First-class semantics imply arbitrary
  nesting; the open question is notation and LLM generability, not
  semantics.
- **Aliasing.** Function values are values. Whether two names can alias one
  function value, and whether a closure's captured environment is shared
  across aliases, interacts with the Ownership model (value vs reference
  semantics of function values) and `DELEGATE.md`'s callable-value model.

## Canonical Forms (hypothesis sketch)

Syntax is **not** settled — these are candidate forms for discussion only.
The canonical-form decision belongs to Phase 5.

```orthon
let inc = |x| x + 1                    # lambda form (expression)
let inc = fn (x: Int) -> Int x + 1     # fn form (FUNCTIONS.md precedent)
let make = |n| |x| n + x               # cascade — anonymous returning anonymous
add = fn [capture: count] (y: Int) -> Int count + y  # explicit capture (capture keyword, CLOSURE_CAPTURE.md)
```

## Open Questions

1. **Can an anonymous function return another function? Is this a cascade,
   and is nesting limited to 1+1 or arbitrary depth?**
   Grounding: first-class functions (`FUNCTIONS.md`) and
   `FUNCTION_RETURNING_FUNCTION.md` (currying, factories, decorators,
   composition). Orthogonality (no special cases) suggests no semantic depth
   limit. The open part is notation — the double arrow and the overloaded
   `->`. Question: is arbitrary (curried) nesting allowed, or is depth
   deliberately capped (1+1) to preserve LLM generability and readability?

2. **Do the effect markers (`new` / `fun` / `proc`) apply to all functions or
   only to methods with `self`? If free functions carry no marker, how is
   their effect expressed?**
   `EXCLUSIVE_DECLARATIONS.md` / `SEMANTIC_MODEL.md` define the three kinds
   for methods (receiver `self`). Free and anonymous functions have no
   receiver to mutate; is their category omitted, inferred from the body
   (category inference per `ASYNC_LAMBDA.md`), or must it be declared?

3. **How does an anonymous function differ from a lambda?**
   `notes/code-block-semantics.md` treats "lambda expression" as an anonymous
   function in expression form. Terminology question for `GLOSSARY.md`: are
   they synonyms, or is a lambda one (expression / sugar) form of the
   anonymous-function concept, with closures being anonymous functions that
   capture?

4. **Can an anonymous function be assigned to a variable?**
   Yes under first-class semantics (`FUNCTIONS.md`: `multiply = fn ...`).
   Follow-up: what identity / equality semantics do function values get, and
   does assignment change the value's status (bound-by-name vs anonymous)?

5. **Can aliases be made on an anonymous function?**
   With function values as values, can two names alias one function value?
   For closures, is the captured environment shared across aliases? This
   interacts with the Ownership model (value vs reference semantics of
   function values) and `DELEGATE.md`'s callable-value model.
