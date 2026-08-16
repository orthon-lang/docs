# Closure Capture

> **✅ RESOLVED — open design hypothesis, resolved during review (2026-08-16).**
> Raised during the FUNCTIONS concept review (2026-08-07).
> **Resolution:** closure capture is a creation-time resolution of a `using`
> slot (unification accepted). See EDR-081 Amendment (2026-08-16). The
> provisional `capture` keyword is superseded.
>
> **Related:** `FUNCTIONS.md`, `DEPENDENCY_REQUIRE_USING.md` (EDR-081),
> `../what/DESIGN_PRINCIPLES.md` (Semantic Purity, POLA, Parsimony)

## Problem

Closure capture needs an explicit marker (FUNCTIONS.md principle 3 —
Explicitness; no silent capture of the enclosing scope). The keyword must
be chosen without multiplying entities (Parsimony) and without overloading
existing keywords (Semantic Purity / POLA).

Two candidates:
1. **`capture`** — a dedicated keyword: clean, unambiguous meaning; a new word.
2. **`using`** — reuse the EDR-081 resolution keyword, unifying closure
   capture with environment injection into one mechanism.

## Examples (other languages)

| Language | Capture mechanism | Style |
|----------|-------------------|-------|
| JavaScript / Python | Implicit, by reference | Silent — the `i=i` loop-variable problem |
| Swift | `[weak self]`, `[capture list]` | Explicit capture list in brackets |
| Rust | `move` modifier | Explicit, opt-in transfer |
| Scala 3 | `given`/`using` | Keyword-based environment injection |

Swift's bracketed capture list is the closest analogue to the rejected
`fn [...] (...)` syntax from FUNCTIONS.md; Rust's `move` is the closest
analogue to a dedicated capture keyword.

## Implications for Orthon

- **`capture` (dedicated keyword):** preserves Semantic Purity (one meaning
  per symbol) and reads clearly — `fun () capture count -> Int`. Cost: one
  new keyword.
- **`using` (reuse):** gives `using` a third syntactic role/position (EDR-081
  already has call-site provision and constructor provision). EDR-081 was
  created precisely to remove `using` overloading; reuse contradicts that
  rationale unless capture is formally unified as "environment injection"
  with two resolutions (creation-time capture vs call-time provision).
- **Parsimony:** reuse avoids a new word but adds a meaning to an existing
  symbol; `capture` adds one word but removes the need for a capture-list
  delimiter syntax (`[...]`).

## Provisional Decision (superseded 2026-08-16)

`capture` (2026-08-07). Rationale: Semantic Purity / POLA outweigh parsimony;
`using` unification is a larger architectural change best decided
independently.

**Superseded:** the unification was accepted (2026-08-16) — closure capture is
a creation-time resolution of a `using` slot. See EDR-081 Amendment (2026-08-16).

## Open Questions

1. ~~Can `using` be redefined as ONE meaning — "environment injection" — with
   creation-time (capture) and call-time (provision) resolutions, without
   violating Semantic Purity? If yes, `capture` becomes redundant.~~
   **Resolved (2026-08-16):** yes — EDR-081 Amendment; `capture` superseded.
2. Does the unified model improve or hurt LLM generability? *(open — empirical)*
3. Mutable vs by-value capture. *(deferred — separate decision)*

## Syntax (Phase 5 — deferred)

Semantics are locked (EDR-081 Amendment, 2026-08-16). The remaining open
question is purely syntactic — the exact surface form of creation-time
`using` in function literals. Registered in the Phase 5 syntax inbox
(`how/syntax/README.md` — "Open, coupled by pointer"). Phase 5 input; not
under current concept review.

**Open syntax questions:**
1. Canonical literal form: is `using n` part of the literal signature
   (`fn (x) using n -> x * n`) or a separate capture clause?
2. Multiple slots: `fn (x) using a, b -> ...` vs a single `using` clause
   with comma-separated names?
3. Fallback edge case: is an unresolvable slot at creation a compile error,
   or does it fall to `given` per the unified priority (EDR-081 Amendment §2)?
4. Interaction with the signature tail — `using n` + `-> Return` — a known
   contention point (see `FUNCTION_RETURN_SYNTAX.md`,
   `FUNCTION_RETURNING_FUNCTION.md`).

**Coupled with:** `how/syntax/FUNCTION_RETURN_SYNTAX.md`,
`how/syntax/FUNCTION_ARGUMENT_SYNTAX.md`,
`how/syntax/FUNCTION_RETURNING_FUNCTION.md`, lambda syntax
(`notes/code-block-semantics.md`).
