# EDR-091: Withdraw Bidirectional Yield — Emit-Only Generators

**Status:** Accepted

**Date:** 2026-09-06

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: Dan Niedre · Date: 2026-09-06 · Verdict: LOCKED

**Partially supersedes:** [EDR-050](./EDR-050-generators.md) (decision items 1 and 5)

---

### Context

EDR-050 (2026-07-27) extended the `emit` model (EDR-021) with three additions:
bidirectional `yield` (`send`-style receiver semantics), generator expressions,
and `yield from` delegation. A review of the GENERATORS concept found that the
bidirectional form is a **category error** inside the generator model: a pure
producer is a semicoroutine (generator/iterator) — control and data flow in one
direction; a construct that exchanges a value both out and in at every
suspension is a **coroutine**. Generators are the one-way special case of
coroutines, not vice versa — a producer does not consume.

The async `defer` coroutine (EDR-085) cannot host the pattern: its suspension
is **scheduler-driven** (`await(ctx)` on external results), and there is no
**caller-injected resume** — OQ4 removed `yield` from that concept ("runtime
yields on `await` automatically") and OQ6 forbids self-delegation /
resumption-with-a-value. Synchronous interactive patterns (the motivating use
cases of EDR-050) are expressible by composition — an explicit stateful object
with one method per round (`type` + `fun` + `match` on an explicit stage
field) — without new language surface and without a hidden compiler-built
state machine.

### Decision

1. **Withdraw** decision items 1 and 5 of EDR-050: bidirectional `yield` and
   the `BidirectionalGenerator[T, U]` trait.
2. **Withdraw the `yield` and `yield from` keywords entirely (S2).** The
   one-way `yield` ≡ `emit` alias is removed — there is no second production
   keyword. The `yield(value)` free-function alias in LAZY_SEQUENCE_GENERATORS
   (EDR-021) is removed.
3. **Retain** generator expressions (`(expr for x in src if cond)`) and their
   emit-based desugaring (EDR-050 items 2–3). The parenthesised form is the
   **single** syntax; a `gen(...)` call form is rejected as a redundant
   synonym.
4. **Delegation is composition**, not a keyword: `for v in sub: emit v`.
5. **Demote** the withdrawn bidirectional form to the research hypothesis
   [`COROUTINE_ON_YIELD.md`](../../concepts/research/deferrable/COROUTINE_ON_YIELD.md),
   which records the synchronous-yield-coroutine vs. async-`defer`-coroutine
   comparison.
6. Generators are **emit-only** — one-way production per EDR-021, unchanged.

### Consequences

**Positive:**
- One production keyword (`emit`); generator = producer with no consuming
  role — the category error is removed.
- Restores EDR-021's original `emit`-over-`yield` decision and its rationale
  (avoids Python-style `yield` confusion).
- Smaller, more orthogonal core; the EDR-050-flagged negative of "two
  keywords for closely related concepts" is eliminated.

**Negative:**
- An interactive producer-consumer dialogue loses its straight-line,
  compiler-managed resumable-body form; the pattern must be written as
  explicit stateful rounds (composition).

### Compliance

1. `what/concepts/GENERATORS.md` amended: emit-only model with an amendment
   banner and updated Decision History.
2. `yield`, `yield from`, and `BidirectionalGenerator` no longer appear as
   language constructs in accepted (`what/`) documents — only as amendment
   notes, historical rationale, or cross-language references.
3. The hypothesis `how/concepts/research/deferrable/COROUTINE_ON_YIELD.md`
   records the withdrawn form and its comparison against `defer`.
4. Delegation is documented as composition (`for v in sub: emit v`).
5. Registers updated: `what/CORE_CONCEPTS.md`, `what/GLOSSARY.md`,
   `what/LIBRARY_BOUNDARY.md`, `what/concepts/PUSH_STREAMS.md`,
   `what/concepts/LAZY_SEQUENCE_GENERATORS.md`,
   `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`.

### Alternatives Considered

| Alternative | Rationale for Rejection |
|---|---|
| Keep bidirectional `yield` in generators | Category error — producers do not consume (see Context). |
| Re-home the bidirectional form into `defer` (EDR-085) | Not expressible: `defer` is scheduler-driven; no caller-injected resume (OQ4/OQ6). |
| Keep `yield` as a one-way alias for `emit` | Two keywords for one action; contradicts minimal core and EDR-021. |

### Gate Validation

Resolved through the concept-review discussion (2026-09-06): category
correction (S2 decision) verified against `defer` semantics (EDR-085
OQ4/OQ6); generator-expression syntax locked to the single parenthesised
form; comparison and rationale recorded in the demoted hypothesis
`COROUTINE_ON_YIELD.md`.
