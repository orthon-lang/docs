# Accepted Syntax Records

> Canonical reference for **accepted** Orthon syntax, one file per
> construct. This is the "What" destination for syntax decisions — the
> mirror of `how/syntax/` (open hypotheses) and the counterpart of
> `what/concepts/` (accepted semantics).
>
> **Established by:** [EDR-087](../../how/decision_records/process/EDR-087-syntax-acceptance-process.md)
> (2026-08-15).
> **See also:** [`how/syntax/`](../../how/syntax/) (hypothesis inbox + decision queue),
> [`how/SYNTAX_PIPELINE.md`](../../how/SYNTAX_PIPELINE.md) (acceptance pipeline),
> [`what/SYNTAX.md`](../SYNTAX.md) (hub — top-level principles + pointers).

## Purpose

`what/SYNTAX.md` is deliberately kept as a **hub** — Syntax Principles plus
pointers — so it does not grow into an unmaintainable single file. Full
syntax definitions live here, one file per accepted construct.

## Conventions

- **Directory:** `what/syntax/` (What layer — accepted specification, lowercase plural).
- **File naming:** `UPPER_SNAKE_SYNTAX.md` (content docs, UPPER_CASE per AGENTS.md §4.4).
- **Header:** each file declares the deciding EDR and the concept(s) it renders.
- **Entry rule:** a record is created only after the syntax decision passes
  `how/SYNTAX_PIPELINE.md` and receives an Architecture-category EDR.
- **Indexed:** every record is registered in the hub (`what/SYNTAX.md`) and
  in the decision queue (`how/syntax/README.md` § Resolved).

## Records

- [INVOCATION_SYNTAX.md](INVOCATION_SYNTAX.md) — invocation context operators `<-` / `|>` (EDR-085).
- [RANGE_SYNTAX.md](RANGE_SYNTAX.md) — range literal `1..N`, `range(a,b)`, `.step(n)` (EDR-083).
- [GENERICS_SYNTAX.md](GENERICS_SYNTAX.md) — `<>` parameters, `as` bounds, `+` conjunction (EDR-086).
