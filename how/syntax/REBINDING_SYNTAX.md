# Rebinding Syntax

> **Status:** Phase 5 input — syntax hypothesis; not a concept under current review.
> Home: [`how/syntax/`](README.md) — established by EDR-087 (2026-08-15).
> Splits the former single "shadowing marker" question
> ([`SHADOWING_SYNTAX.md`](SHADOWING_SYNTAX.md), superseded) into two independent
> axes: same-scope rebinding and nested-scope capture.
>
> **Last updated:** 2026-08-19

## Issue (Why)

[`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)
(EDR-074) settles the semantics: the first assignment declares a variable; the
type is inferred from the initializer; variables are immutable by default;
re-assignment to an immutable name is a compile-time error; name shadowing is
permitted only with an explicit marker (Principle 5). The concrete keyword is
deferred to Phase 5 (Syntax).

`SHADOWING_SYNTAX.md` treated "the shadowing marker" as one question. It is
really two independent questions with different costs:

1. **Same-scope rebinding** — reuse a name for a new value in the same scope
   (the functional pipeline idiom: `data = parse(data)`).
2. **Nested-scope capture** — redeclare a name that is already visible from an
   outer scope (usually a bug source; should be explicit or forbidden).

## Locked Decisions (context, 2026-08-19)

- Immutable by default, keyword-free declaration: `x = 1` (EDR-074).
- Mutable marker: `var x = 1` — retained.
- `val` — rejected: naming confusion; universally `val` means immutable
  (Kotlin, Scala), which conflicts with LLM Generability.
- No implicit shadowing — any name reuse must be syntactically visible
  (EDR-074, Principle 5).
- `using` is unavailable as a capture marker — it is taken by dependency slots
  (`require`/`using`, EDR-081; sugar over context, EDR-085). See
  [`REQUIRE_USING_DEPENDENCY_SLOTS.md`](../../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md).

## Hypothesis

### Axis 1 — Same-scope rebinding

The pipeline idiom must be preserved. Two candidate mechanisms to verify
(LLM Generability, immutable-by-default, orthogonality):

| Candidate | Syntax | Semantics | Cost |
|-----------|--------|-----------|------|
| `var` reassignment | `var data = read(); data = parse(data)` | mutation of one cell | mutation where a fresh binding would do; weakens immutable-by-default |
| `let` shadowing | `let data = parse(data)` | fresh binding, same name | one extra keyword for rebinding |

Open: is a fresh binding (`let`) worth a keyword, or does `var` reassignment
cover the idiom at acceptable cost?

### Axis 2 — Nested-scope capture

Redeclaring an outer name inside a block must be explicit. `using` is taken.
The candidate marker name is open (not `let`, not `var`, not `using`).

## Rules

1. **No implicit shadowing** — a name already visible (same or outer scope)
   cannot be redeclared without an explicit marker (EDR-074, Principle 5).
2. **`using` is reserved** for dependency/context slots (EDR-081, EDR-085) —
   not usable as a capture marker.
3. **`var` is the mutability marker**, not a declaration keyword.
4. **`val` is rejected** as redundant and misleading.

## Open Questions

1. Axis 1: `var` reassignment vs `let` shadowing — which, or both?
2. Axis 2: candidate name for the explicit capture marker.
3. Should nested-scope capture exist at all, or must an inner block use a fresh name?
4. Coupling with closure capture (resolved via `using` slots, EDR-081 Amendment)
   and with type-annotation placement.

## Cross-References

- [`README.md`](README.md) — syntax hypothesis inbox (decision queue).
- [`SHADOWING_SYNTAX.md`](SHADOWING_SYNTAX.md) — superseded framing; kept as the
  `let`-based candidate for Axis 1.
- [`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md) —
  concept (EDR-074), Principle 5, Phase 5 boundary.
- [`TYPE_ANNOTATION_SYNTAX.md`](TYPE_ANNOTATION_SYNTAX.md) — coupled: marker/annotation placement.
- [`REQUIRE_USING_DEPENDENCY_SLOTS.md`](../../what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md) —
  why `using` is unavailable (EDR-081).
- [`CLOSURE_CAPTURE.md`](../concepts/research/essential/CLOSURE_CAPTURE.md) —
  capture via `using` slots (EDR-081 Amendment).
