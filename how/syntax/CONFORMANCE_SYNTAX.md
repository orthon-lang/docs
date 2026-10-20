# Hypothesis: Conformance Syntax — `impl ... for` vs `obeys`

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the 2026-10-18/19 review of the CONTRACTS concept. Phase 5
> input — syntax hypothesis; not a concept under current review. Home:
> [`how/syntax/`](README.md) — established by
> [EDR-087](../decision_records/process/EDR-087-syntax-acceptance-process.md).
>
> **Author's decision (2026-10-19):** `obeys` is accepted **as a candidate**
> to replace `impl`. It is not accepted syntax: it must pass the syntax
> pipeline, and it is **blocked on the semantic hypothesis**
> ([`TRAIT_CONTRACT_UNIFICATION.md`](../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md)),
> because a syntax decision cannot add or change semantics
> ([`SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md)).
>
> **Last updated:** 2026-10-20
>
> **Scope (Review decision 15 of the semantic hypothesis):** v0.1 covers
> types that obey contracts and named functions that obey contracts at their
> declaration. `obeys` on lambdas, conformance of function values used
> through generics or `dyn`, and the coupling to the `->` overload are
> **deferred to v0.2**.
>
> **See also:** [`../../what/SYNTAX.md`](../../what/SYNTAX.md) (Syntax Principles),
> [`../../what/concepts/TRAITS.md`](../../what/concepts/TRAITS.md) (EDR-019),
> [`../../what/concepts/CONTRACTS.md`](../../what/concepts/CONTRACTS.md) (EDR-056),
> [`FUNCTION_RETURNING_FUNCTION.md`](FUNCTION_RETURNING_FUNCTION.md) (`->` overload),
> [`FUNCTION_RETURN_SYNTAX.md`](FUNCTION_RETURN_SYNTAX.md),
> [`FUNCTION_ARGUMENT_SYNTAX.md`](FUNCTION_ARGUMENT_SYNTAX.md),
> [`BOUNDS_IN_ANGLE_BRACKETS.md`](BOUNDS_IN_ANGLE_BRACKETS.md) (signature tail),
> [`../concepts/research/important/TRAIT_BLANKET_IMPLEMENTATION.md`](../concepts/research/important/TRAIT_BLANKET_IMPLEMENTATION.md),
> [`../DESIGN_PRINCIPLES.md`](../DESIGN_PRINCIPLES.md)

---

## Problem

One question: **which word, and in which position, declares that a type or a function conforms to a contract?**

1. Today a type declares conformance with `impl Trait for Type` (Rust lineage). The author dislikes `impl`: it names an implementation block rather than a promise to satisfy a specification.
2. The contract-unification hypothesis proposes that functions declare conformance to contracts too (`fun f(...) obeys Sqrt`). `impl ... for` does not read for a function.
3. [Syntax Principle 1](../../what/SYNTAX.md) ("one concept → one syntax") means that if functions get `obeys` while types keep `impl ... for`, one concept has two spellings. A new word is justified only if it **replaces** `impl`, not if it is added next to it.

This file decides the surface form only. The meaning (one construct, kind decided by the application site) is the subject of the semantic hypothesis.

## Inventory of current forms (as of 2026-10-19)

| Form in the specification | Where |
|---|---|
| `impl Trait for Type` | TRAITS and many concept examples (35 lines in 17 files under `what/` and `how/`) |
| `impl<T> Printable for T where T as Display` | blanket implementation (TRAITS); bound-first alternative `impl<Display as T> Printable for T` in `TRAIT_BLANKET_IMPLEMENTATION.md` |
| `impl Callable(Int) -> Age for Age` | contract with parameters (CONSTRAINED_TYPES desugaring) |
| `impl Show for Point:` and braced `impl ... { ... }` | older examples with block openers; not consistent with each other |
| conformance outside the type's own declaration (`impl ForeignTrait for User`) | orphan-rule examples |
| `@derive(...)` generating `impl` blocks | EDR-029 |

## Candidates

| Word | Reads for a type | Reads for a function | Collision in the specification | Note |
|---|---|---|---|---|
| `obeys` | `User obeys Printable` | `fun f(x) -> T obeys Sqrt` | none as keyword or identifier; one prose use ("the general law it obeys", `SEMANTIC_MODEL.md`) | short; direction "bound by a rule"; no known prior in mainstream languages (not verified) |
| `impl ... for` | yes | awkward | is the status quo | disliked by the author |
| `implements` | yes | awkward | prose only (7 files) | Java prior |
| `satisfies` | yes | yes | prose only (5 files) | longer; more likely as a user identifier |
| `follows` | yes | yes | prose only (11 files) | ambiguous (sequence) |
| `is` | yes | yes | taken: identity equality (EQUALITY) | violates "one symbol → one meaning" |
| `as` | — | — | taken: bounds and labels | violates the same principle |
| `with` | — | — | taken: copy-with-modify | violates the same principle |

Languages in other ecosystems use `implements`, `impl ... for`, a colon after the type name, `instance ... where`, or implicit (structural) satisfaction. These are cited from general knowledge and have not been verified against primary sources.

## Examples by usage context (candidate `obeys`; spellings provisional)

```orthon
// 1. Type: declaration followed by members
User obeys Printable
    fun format(self) -> String
        return "User({self.name})"

// 2. Function: signature tail
fun sqrt_newton(x: Float) -> Float obeys Sqrt
    ...

// 3. Method
proc push(item: T) obeys Pushable
    ...

// 4. Several contracts
fun sqrt_safe(x: Float) -> Float obeys Sqrt, Logged

// 5. Contract with parameters (replaces: impl Callable(Int) -> Age for Age)
Age obeys Callable(Int) -> Age

// 6. Bound on a generic: unchanged, a different role (requirement, not promise)
fun print_all<Printable as T>([T] items)

// 7. Conformance declared outside the type's own declaration (orphan-rule case)
ForeignType obeys LocalContract
```

Problem cases:

```orthon
// 8. [v0.2, deferred] Inline lambda: where does the tail go?
fn(x) -> newton(x) obeys Sqrt          // does `obeys` belong to the lambda or to the body expression?

// 9. [v0.2, deferred] Block lambda: unambiguous
fn (x: Float) -> Float obeys Sqrt
    newton(x)

// 10. Conditional (blanket) conformance: form undefined
List<Printable as T> obeys Printable   // candidate; interacts with TRAIT_BLANKET_IMPLEMENTATION

// 11. Reserved word
let obeys = 3                          // ERROR if reserved; allowed if the word is contextual
```

## Coupling and overload check (Syntax Pipeline, Stage 3)

| Coupled question | Why |
|---|---|
| Signature tail order ([`BOUNDS_IN_ANGLE_BRACKETS.md`](BOUNDS_IN_ANGLE_BRACKETS.md), `FUNCTION_RETURN_SYNTAX.md`) | the tail already holds `require`, `where`, `-> Return` with no canonical order; `obeys` joins it |
| `->` overload ([`FUNCTION_RETURNING_FUNCTION.md`](FUNCTION_RETURNING_FUNCTION.md)) | **Deferred to v0.2 with lambdas.** In the inline lambda `fn(x) -> expr` the arrow separates the body, so there is no slot for a tail clause before the body |
| Comma | `obeys A, B` uses the comma, which is already an overloaded symbol in the queue |
| Blanket conformance ([`TRAIT_BLANKET_IMPLEMENTATION.md`](../concepts/research/important/TRAIT_BLANKET_IMPLEMENTATION.md)) | the conditional form depends on where bounds live |
| Block openers and braces | older `impl` examples disagree (`:` versus `{ }`) |
| Name of the construct and of the multi-instance kind | `contract` versus `trait`, and the word for multi-instance contracts, are tracked in the semantic hypothesis |

## Syntax Principles (preview — not gate verdicts)

| Principle | Reading |
|---|---|
| 1. One concept → one syntax | satisfied only if `obeys` replaces `impl`; two forms for one concept otherwise |
| 2. One symbol → one meaning | `obeys` has one meaning for types and functions |
| 3. No significant whitespace | independent of this choice |
| 4. Named before symbolic | a word, not a symbol |
| 5. Syntax derived from semantics | blocked until the semantic hypothesis is decided |

## Implications for Orthon

### Positive

- One declaration word for types and functions; the verb reads as a promise to a rule.
- Removes `impl ... for`, whose `for` also names loops.
- No symbol is added.

### Negative / Risks

- A new reserved word; the full reserved-word inventory does not exist yet, so the cost cannot be counted.
- No prior in mainstream languages (not verified): LLM generability must be measured against `impl`.
- Cascade: 35 `impl ... for` lines in 17 files, plus `@derive` output and every accepted concept that shows `impl`.
- The signature tail order is unsolved; the inline-lambda slot is deferred to v0.2 together with lambdas.

## Open Questions

1. Reserved word or contextual (soft) keyword that is reserved only in the tail and declaration positions?
2. Position per construct: signature tail for functions, statement for types, or also inline in a type header (`type User obeys Printable`).
3. **Deferred to v0.2 (Review decision 15).** Spelling for lambdas, given the `->` overload.
4. Conditional (blanket) conformance and the placement of its bounds.
5. Contracts with parameters (`obeys Callable(Int) -> Age`).
6. Keep a `for`-style target for conformance declared outside the type (orphan-rule case), or is the statement form enough?
7. Measurement protocol for the LLM Generability Gate: `obeys` against `impl ... for` on type-level and function-level declarations.

## Next Step

1. Wait for (or decide together with) the semantic hypothesis [`TRAIT_CONTRACT_UNIFICATION.md`](../concepts/research/essential/TRAIT_CONTRACT_UNIFICATION.md): the construct and the application sites must be fixed first.
2. Run the Decision Pipeline pre-filter and the Coupling & Overload Check against the signature-tail hypothesis (the `->` overload matters here only for lambdas, which are deferred to v0.2).
3. Apply the five Syntax Principles and the required gates (LOGICAL_CONSISTENCY, CONCEPTUAL_SIMPLICITY, ARCHITECTURAL_INTEGRITY, LLM_GENERABILITY).
4. Human sign-off, then an EDR (Architecture) and a record in `what/syntax/`.

> **Human sign-off:** `Reviewed-by: — · Date: — · Verdict: pending`
> Agents and GSD flows must not self-certify this field.
