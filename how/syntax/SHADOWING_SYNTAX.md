# Shadowing Syntax

> **Status:** Phase 5 input — syntax hypothesis; not a concept under current review.
> Home: [`how/syntax/`](README.md) — established by EDR-087 (2026-08-15).
> Records the explicit-shadowing-keyword hypothesis deferred to Phase 5 (Syntax)
> by [`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md)
> (EDR-074, Principle 5). The semantic core — shadowing requires an explicit
> marker — is settled in the concept; this document holds the syntax hypothesis
> for the marker and its interaction with mutability.
>
> **Last updated:** 2026-08-15

## Issue (Why)

How should Orthon express variable shadowing, and how should the shadowing marker compose with the mutability marker?

[`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md) Principle 5 settles the semantics: shadowing (re-declaring a name in an inner scope) is permitted but must be syntactically visible. The concept defers the concrete keyword and its interaction with mutability to Phase 5 (Syntax). This document records the proposed hypothesis.

## Hypothesis: two orthogonal axes

Shadowing and mutability are two independent axes. Each keyword performs exactly one job (one concept per syntax).

| | Immutable | Mutable |
|---|---|---|
| **Fresh name** | `x = 1` | `var x = 1` |
| **Shadowing** | `let x = 1` | `let var x = 1` |

- `x = 1` — first assignment declares a new, immutable variable. Default path: no keyword.
- `var x = 1` — declares a new mutable variable. Valid only when the name does not already exist in scope.
- `let x = 1` — declares a new binding that shadows an existing name. Immutable.
- `let var x = 1` — declares a new mutable binding that shadows an existing name.

## Rules

1. **`let` = new binding over an existing name** — it signals "I know I am creating a new variable, hiding an outer one".
2. **`var` = mutable** — it is a mutability marker, not a declaration keyword.
3. **`var` on an existing name is a compile error** — mutable shadowing must be written `let var x = ...`. This keeps `var` unambiguous and keeps shadowing explicit (Principle 5).
4. **No `val`** — the default path is keyword-free immutable, so a dedicated immutable-declaration keyword would be redundant.

## Rationale

- **Orthogonality** — binding kind and mutability are independent; every combination is expressible and no keyword is overloaded.
- **LLM Generability** — `let` and `var` are corpus-familiar (JavaScript, TypeScript, Kotlin, Swift, Rust). `let var` mirrors Rust's `let mut`, which LLMs already know.
- **Explicit shadowing** — accidental shadowing is impossible because the marker is mandatory.

## Comparison

| Language | Binding | Mutable binding | Shadowing |
|---|---|---|---|
| Rust | `let x` | `let mut x` | `let` (always) |
| Kotlin | `val x` | `var x` | fresh scope name |
| TypeScript / JavaScript | `const x` / `let x` | `let x` | `let` / `const` (inner scope) |
| Orthon (hypothesis) | `x = 1` | `var x = 1` | `let x = 1` / `let var x = 1` |

## Open Questions (Phase 5)

1. Keyword order: `let var x` vs `var let x` — the hypothesis recommends `let var`, mirroring Rust's `let mut`.
2. Should bare `let x` on a fresh name be a compile error (there is nothing to shadow) or an allowed-but-redundant declaration?
3. Keyword naming: `let` vs `shadow` vs another marker.

## Cross-References

- [`README.md`](README.md) — syntax hypothesis inbox (decision queue).
- [`DECLARATION_BY_ASSIGNMENT.md`](../../what/concepts/DECLARATION_BY_ASSIGNMENT.md) — concept (EDR-074), Principle 5, Phase 5 boundary.
- [`TYPE_ANNOTATION_SYNTAX.md`](TYPE_ANNOTATION_SYNTAX.md) — companion Phase 5 syntax hypothesis.
