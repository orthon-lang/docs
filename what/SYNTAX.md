# Syntax Reference

> **⚠️ DRAFT — Placeholder for Phase 5.**
> This document will contain the complete syntax reference for all
> Orthon language concepts. Syntax is derived from semantics, not
> the reverse.
>
> **Status:** Hub — Syntax Principles plus pointers to accepted syntax
> records in `what/syntax/`. Established by EDR-087 (2026-08-15).
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 5,
> [`SEMANTIC_MODEL.md`](SEMANTIC_MODEL.md),
> [`PRIMITIVE_BLOCKS.md`](PRIMITIVE_BLOCKS.md),
> [`ARCHITECTURE.md`](../how/architecture/ARCHITECTURE.md),
> [`PARSER.md`](../how/architecture/PARSER.md)

---

## Syntax Principles

1. **One concept → one syntax.** Each language concept has exactly one
   canonical syntactic form.
2. **One symbol → one meaning.** Every symbol, operator, and keyword
   has exactly one context-independent meaning.
3. **No significant whitespace.** Indentation is cosmetic; structure is
   explicit.
4. **Named before symbolic.** Named forms preferred when ambiguity risk
   outweighs brevity benefit.
5. **Syntax derived from semantics.** Syntax is the external interface
   of the semantic model, not an independent design exercise.

## Accepted Syntax (hub)

`what/SYNTAX.md` is the **hub**: it holds the Syntax Principles and points
to the full accepted syntax records in [`what/syntax/`](syntax/). One record
per construct — see [`what/syntax/README.md`](syntax/README.md) for the index.

| Construct | Record | Decision |
|-----------|--------|----------|
| Invocation context operators `<-` / `\|>` | [`what/syntax/INVOCATION_SYNTAX.md`](syntax/INVOCATION_SYNTAX.md) | EDR-085 |
| Range literal `1..N`, `range(a,b)`, `.step(n)` | [`what/syntax/RANGE_SYNTAX.md`](syntax/RANGE_SYNTAX.md) | EDR-083 |
| Generics: `<>` parameters, `as` bounds | [`what/syntax/GENERICS_SYNTAX.md`](syntax/GENERICS_SYNTAX.md) | EDR-086 |
| Generator expression `gen(...)` | [`what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](syntax/GENERATOR_EXPRESSION_SYNTAX.md) | EDR-092 |

**Open syntax questions** (Phase 5 input) live in the hypothesis inbox:
[`how/syntax/`](../how/syntax/) with its decision queue
([`how/syntax/README.md`](../how/syntax/README.md)). The acceptance process
is defined in [`how/SYNTAX_PIPELINE.md`](../how/SYNTAX_PIPELINE.md) (EDR-087).
