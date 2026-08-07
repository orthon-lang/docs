# Closure Capture

> **⚠️ HYPOTHESIS — open design hypothesis, not an accepted concept.**
> Raised during the FUNCTIONS concept review (2026-08-07).
> **Provisional decision:** `capture` is the closure-capture keyword.
> The unification question below is deliberately deferred.
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

## Provisional Decision

`capture` (2026-08-07). Rationale: Semantic Purity / POLA outweigh parsimony;
`using` unification is a larger architectural change best decided
independently.

## Open Questions

1. Can `using` be redefined as ONE meaning — "environment injection" — with
   creation-time (capture) and call-time (provision) resolutions, without
   violating Semantic Purity? If yes, `capture` becomes redundant.
2. Does the unified model improve or hurt LLM generability?
3. What is the exact syntax for mutable vs by-value capture under `capture`?
