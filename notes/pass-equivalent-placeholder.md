# Pass Equivalent / Placeholder / Stub

> Discussion: what does Orthon use as a no-op / placeholder / stub, equivalent to Python's `pass`?

## Summary

Orthon has **no explicit `pass` keyword**. The most natural equivalent,
based on existing primitives, is the **empty block `{}`** — the `scope`
primitive explicitly states *"a scope with zero bindings is still a
scope"* (PRIMITIVE_BLOCKS.md § 3.2.5).

## Candidates

| Approach | Description | Alignment |
|----------|-------------|-----------|
| **Empty block `{}`** | Use `{ }` as body — `fun stub() {}` | High — reuses existing `scope` primitive, no new keyword |
| **Empty body (implicit)** | `fun stub()` with no body at all | Ambiguous — parser ambiguity risk |
| **`skip` keyword** | Explicit no-op keyword (Rust/Zig style) | Low — violates Minimal Core (new keyword for a trivial need) |
| **`todo()` / `unimplemented()`** | Panic/compile-error on reach | StdLib/macro — different use case (marker, not no-op) |

## Context

SYNTAX.md is a placeholder for Phase 5. This decision is not yet
formalised in the spec. If formalisation is desired before Phase 5,
it requires a gate entry in `how/gates/_language-design.md` and
potentially an EDR.

## Why not `pass`

Python's `pass` exists because of significant indentation — empty body
is syntactically ambiguous. Orthon uses explicit `{ }` blocks, so `{}`
is naturally a no-op without a dedicated keyword.

## See Also

- PRIMITIVE_BLOCKS.md § 3.2.5 `scope` — "A scope with zero bindings is still a scope"
- SYNTAX.md — Syntax reference (Phase 5 placeholder)
