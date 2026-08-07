# Function Type Notation — Functions Returning Functions

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the FUNCTIONS concept review (2026-08-07).
> Syntax question — Phase 5 scope. Coupled with return-type placement
> (`FUNCTION_RETURN_SYNTAX.md`) and argument syntax
> (`FUNCTION_ARGUMENT_SYNTAX.md`).
>
> **See also:** `what/SYNTAX.md` (Phase 5 placeholder), `FUNCTIONS.md`,
> `CLOSURE_CAPTURE.md`, `how/DESIGN_PRINCIPLES.md` (Semantic Purity, DRY),
> `notes/code-block-semantics.md`

## Problem

Two related notation questions:

1. **Function type in annotations.** The current notation writes a function
   type with an arrow: `pred: (Int) -> Bool`. Reviewer feedback (2026-08-07):
   "inside `()` we only define input data — there should be no `->` there."
   The arrow reads as noise inside a parameter annotation.
2. **Function returning a function.** The current notation nests arrows:
   `fun make_adder(n: Int) -> (Int) -> Int`. The double arrow is non-obvious
   and hard for an LLM to generate correctly.

Root cause: `->` is overloaded (function type, return annotation, lambda
body — plus the `=>` match-arm conflict). A reader cannot always tell which
role `->` plays without context.

## Is a function-returning-function public?

Yes — not exotic; it is the basis of:
- **Currying / partial application** — `make_adder(n)` returns `(x) -> n + x`;
- **Factories** — `create_parser(fmt)` returns a parser;
- **Decorators** — a decorator is literally `(F) -> F`;
- **Composition** — `compose(f, g)` returns `f ∘ g`;
- **Middleware / configuration** — a configured handler.

First-class functions (FUNCTIONS.md) require it. Only the *notation* is open.

## Examples (other languages)

| Language | Function-returning-function notation |
|----------|--------------------------------------|
| Swift | `func makeAdder(_ n: Int) -> (Int) -> Int` (double arrow) |
| Kotlin | `fun makeAdder(n: Int): (Int) -> Int` (colon + arrow) |
| Rust | `fn make_adder(n: i32) -> impl Fn(i32) -> i32` (arrow + `Fn` trait) |
| Haskell | `makeAdder :: Int -> (Int -> Int)` (curried arrow) |
| TypeScript | `function makeAdder(n: number): (x: number) => number` (double arrow) |

All first-class-function languages solve it; none makes it read trivially.

## Implications for Orthon

- **Option A — keep `(T) -> U`** (current convention; Rust/Swift/Kotlin style).
  Familiar, but `->` stays overloaded and the double arrow remains.
- **Option B — nominal function type** `Fn[Int, Bool]` (no arrow in
  annotations). Directly answers "no `->` inside `()`": `pred: Fn[Int, Bool]`.
  Returned functions: `fun make_adder(n: Int) -> Fn[Int, Int]` — no nesting.
  Cost: more verbose; deviates from convention.
- **Option C — "no arrow" model** (the reviewer's instinct): prefix return
  type (`fun Int make_adder(n: Int)`), nominal function types
  (`Fn[Int, Bool]`), lambda bodies via explicit `return`
  (`(x) return x > 0`). Then `->` is removed from the language entirely —
  one less overloaded symbol, no double arrow. Cost: largest deviation;
  function types verbosity.

The reviewer's broader "type-first" model (`Type name = expr` declarations,
`(Int a, Int b)` arguments, prefix return) is consistent with Option C. Note:
type-first declarations would reverse the annotation position of the accepted
`DECLARATION_BY_ASSIGNMENT.md` (EDR-074, currently `count: Int = 42`).

## Open Questions

1. Is a first-class function *type* syntax needed at all, or can function
   types be nominal (`Fn[...]`, trait-based per EDR-019)?
2. If `->` is kept, should it be reserved for exactly one role — which one?
3. Which notation is most LLM-generable for a function returning a function?
4. Interaction with the capture keyword (`capture`, `CLOSURE_CAPTURE.md`) in
   the chosen lambda/function form.

## Next Step

Phase 5 syntax decision; coupled with `FUNCTION_RETURN_SYNTAX.md`,
`FUNCTION_ARGUMENT_SYNTAX.md`, `CLOSURE_CAPTURE.md`, and lambda syntax
(`notes/code-block-semantics.md`).
