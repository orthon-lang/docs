# Range Syntax

> **Accepted — EDR-083 (Range).** Explicit syntax record created by EDR-087
> (2026-08-15) so the accepted range syntax has a home in `what/syntax/`.
> The canonical semantic specification lives in `what/concepts/RANGE.md`;
> this file records the surface form.

## Canonical forms

```orthon
1..N                # inclusive-inclusive range literal: 1, 2, …, N (N elements)
range(a, b)         # named canonical form; range(a, b) ≡ a..b

(1..10).step(2)     # strided Range value (method, not syntax): 1, 3, 5, 7, 9
(1..5).step(-1)     # negative step iterates descending: 5, 4, 3, 2, 1

for i in 1..10      # Range implements IntoIterator<Int>; usable directly
```

## Rules

1. **`a..b` is inclusive-inclusive** — `1..N` produces N elements. This is
   the *only* range semantic; the `..=` spelling is eliminated (EDR-082 norm).
2. **`range(a, b)` ≡ `a..b`** — the literal is **Language** (compiler-
   recognized); the `Range` type and `range()` constructor are **StdLib**.
3. **`:` is never used in ranges** — `0..10:step(2)` is rejected; `:` is
   reserved for type annotations (conflict closed by EDR-083).
4. **`.step(n)` is a method** on `Range` returning a strided `Range` value
   (still a data value, still `IntoIterator`). `step(0)` is a compile-time error.
5. **Empty range is a value** — `end < start` (for example `1..0`) yields
   zero elements, not a syntax error.
6. **`0..<N` is FFI-boundary interop only** — never in application code.
7. **`enumerate(items)` ≡ `zip(1..len(items), items)`** — the `..=` spelling
   is superseded.

## Cross-References

- [EDR-083](../../how/decision_records/architecture/EDR-083-range.md) — deciding record.
- [`what/concepts/RANGE.md`](../concepts/RANGE.md) — full semantic specification.
- [`what/concepts/INDEXING.md`](../concepts/INDEXING.md) — 1-based indexing norm (EDR-082).
- [`what/SYNTAX.md`](../SYNTAX.md) — hub.
- [`how/syntax/README.md`](../../how/syntax/README.md) — decision queue (resolved).
