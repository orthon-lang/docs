# Generics Syntax

> **Accepted — EDR-086 (Generics Syntax Revision).** Explicit syntax record
> created by EDR-087 (2026-08-15) so the accepted generics syntax has a home
> in `what/syntax/`. The semantic model (bounds, dispatch, variance) lives in
> `what/concepts/GENERICS.md` (EDR-024); this file records the surface form.

## Canonical forms

```orthon
type Pair<T, U>               # type parameters in angle brackets
fun identity<T>(T value) -> T

fun first<Iterator as T>(T items)      # bound-first inline shorthand
type Pair<Hash as K, V>                # bound-first on a type parameter

fun process<Hash + Eq as T>(T value, T other)   # `+` conjoins bounds on one parameter
where T as Hash, U as Ord               # `,` separates constraints on different parameters
```

## Rules

1. **Type parameters use `<>`** — `[ ]` is freed for indexing, array types,
   and slices (EDR-086 problems 1–2).
2. **Bounds are written bound-first with `as`** — `<Iterator as T>` reads
   "T is (at least) an Iterator".
3. **`+` combines bounds on one parameter** (conjunction); **`,` separates
   constraints on different parameters** — the two roles must not be conflated.
4. **No `where T: Trait` and no `[T: Trait]`** — the `:` bound form is
   retired; `:` remains the type-annotation symbol.
5. **No method-level shadowing** — a method must not re-declare a type
   parameter of its enclosing type; a duplicate name is a compile error.
6. **Variance** is computed by position-based inference from trait method
   signatures; examples use Orthon's real subtyping sources (unions, literal
   types, widening) — never a class hierarchy.
7. **`<>` requires parser disambiguation** from comparison operators `<`/`>`
   (well-understood problem; Rust, Swift).
8. **Explicit variance annotations** remain deferred to v0.2+.

## Open revision candidate

`BOUNDS_IN_ANGLE_BRACKETS.md` (in `how/syntax/`) proposes eliminating the
`where` clause entirely and moving **all** bounds into `<>`. If adopted in
Phase 5, it supersedes the `where`-clause portion of EDR-086.

## Cross-References

- [EDR-086](../../how/decision_records/architecture/EDR-086-generics-syntax-revision.md) — deciding record.
- [EDR-024](../../how/decision_records/architecture/EDR-024-generics.md) — generics semantics (superseded in syntax by EDR-086).
- [`what/concepts/GENERICS.md`](../concepts/GENERICS.md) — full semantic specification.
- [`what/concepts/TRAITS.md`](../concepts/TRAITS.md) — trait bounds (EDR-019).
- [`what/SYNTAX.md`](../SYNTAX.md) — hub.
- [`how/syntax/BOUNDS_IN_ANGLE_BRACKETS.md`](../../how/syntax/BOUNDS_IN_ANGLE_BRACKETS.md) — open revision candidate.
