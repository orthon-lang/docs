# Syntax Hypothesis Inbox

> Home for **open syntax hypotheses** — the syntax-lens mirror of
> `how/concepts/research/` (semantic hypotheses). Files here are
> Phase 5 (Syntax) **input material**: language comparisons and
> candidate spellings deferred from accepted concepts. They are not
> accepted Orthon specification.
>
> **Established by:** [EDR-087](../../how/decision_records/process/EDR-087-syntax-acceptance-process.md)
> (2026-08-15).
> **See also:** [`how/SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md) (the acceptance pipeline),
> [`what/syntax/`](../../what/syntax/) (accepted syntax records),
> [`what/SYNTAX.md`](../../what/SYNTAX.md) (hub — canonical reference),
> [`how/concepts/research/`](../concepts/research/) (semantic hypothesis inbox).

---

## Purpose

When an accepted concept defers its concrete syntax to Phase 5 (for
example, `EDR-027` deferred `: Type`, `EDR-074` deferred the annotation
syntax), the deferred syntax decision is recorded here as a **hypothesis
document**: one file per open syntax question, capturing the candidates,
the comparison, and the coupling with other syntax decisions. Phase 5
consumes this inbox, resolves each question through
[`SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md), and folds the result into
`what/syntax/` + the `what/SYNTAX.md` hub.

This inbox exists so no deferred syntax decision is lost when Phase 5
iterates concept-by-concept — it is the single inventory of "what still
needs a syntax decision".

## Conventions

- **Directory:** `how/syntax/` (How layer — design input, lowercase plural).
- **File naming:** `UPPER_SNAKE_SYNTAX.md` (content docs, UPPER_CASE per AGENTS.md §4.4).
- **Status header:** every file starts with a blockquote declaring
  *"Phase 5 input — syntax hypothesis; not a concept under current review."*
- **One question per file.** If a hypothesis spawns a distinct question, split it.
- **Coupling is explicit.** Each file lists the other syntax questions it
  couples with (same symbol, same position, same construct).
- **Indexed here.** Every hypothesis is registered in the Decision Queue
  below; do not add a file without a queue row.
- **Coupled-by-pointer.** Some open syntax questions live in concept
  research docs (for example `CLOSURE_CAPTURE.md`,
  `TRAIT_BLANKET_IMPLEMENTATION.md`, `notes/code-block-semantics.md`).
  They are **not** moved here — they are registered by pointer so the queue
  remains the single inventory.

## Lifecycle

```
how/syntax/{NAME}.md          ◄── SYNTAX HYPOTHESIS INBOX (this directory)
      │                            Open question, candidates, coupling.
      ▼
Decision Pipeline (10 Q)      ◄── PRE-FILTER (reuse how/process/DECISION_PIPELINE.md)
      │                            Language feature? Library? Sugar? Optimisation?
      ▼
Coupling & Overload Check     ◄── SYNTAX-SPECIFIC (SYNTAX_PIPELINE.md § Stage 3)
      │                            Symbol/keyword collision sweep across ALL concepts
      │                            (precedents: EDR-086 `:`, EDR-083 `:` in ranges).
      ▼
Syntax Principles (5)         ◄── ACCEPTANCE CRITERIA (what/SYNTAX.md § Syntax Principles)
      │                            One concept→one syntax; one symbol→one meaning;
      │                            no significant whitespace; named before symbolic;
      │                            derived from semantics.
      ▼
Syntax Acceptance Gates       ◄── VALIDATION (DECISION_VALIDATION.md § Gate Selection —
      │                            "Syntax change" row: LOGICAL_CONSISTENCY,
      │                            CONCEPTUAL_SIMPLICITY, ARCHITECTURAL_INTEGRITY,
      │                            LLM_GENERABILITY; + optional USER_VALUE /
      │                            IMPLEMENTATION_INDEPENDENCE / LONG_TERM_MAINTAINABILITY).
      ▼
EDR (Architecture category)   ◄── FORMAL RECORD (per syntax decision)
      │
      ▼
what/syntax/{NAME}.md         ◄── ACCEPTED SYNTAX RECORD (accepted hypothesis → What layer)
      │
      ▼
SYNTAX.md hub + "Resolved"    ◄── HUB UPDATE (what/SYNTAX.md gains a pointer;
      │                            hypothesis doc marked "Resolved — see SYNTAX.md § …",
      │                            per RANGE_STEP.md precedent).
      ▼
PARSER.md                     ◄── GRAMMAR (concrete grammar updated)
```

**Unified Phase 5 input flow:**

```
research semantic concepts → EDR + concepts → deferred syntax decisions → EDR + resolved syntax
        (how/concepts/research/)   (EDR + what/concepts/)      (how/syntax/)      (EDR + what/syntax/ + SYNTAX.md hub)
```

## Decision Queue

All syntax questions (open and resolved). Status legend: **Open** — awaiting
Phase 5 resolution; **Resolved** — accepted (record in `what/syntax/` or an
EDR); **Rejected** — a binding negative decision.

**Human review** column: the solo author's explicit sign-off required by
[`SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md) Stage 6b —
`Reviewed-by · Date · Verdict [LOCKED / changes]`. Open items are
`Pending`; no EDR may be filed before the sign-off is populated and
agents/GSD flows must not self-certify it.

### Open (in this inbox)

| Question | Hypothesis doc | Deferral source | Coupled with | Status | Human review |
|----------|----------------|-----------------|--------------|--------|--------------|
| Type annotation: `x: Type` vs `Type x` | [TYPE_ANNOTATION_SYNTAX.md](TYPE_ANNOTATION_SYNTAX.md) | EDR-027, EDR-074 | SHADOWING, FUNCTION_ARGUMENT, FUNCTION_RETURN, FUNCTION_RETURNING_FUNCTION | Open (Decision Pipeline run 2026-08-15) | Pending |
| Rebinding / capture: `var` vs `let`; explicit capture (not `using`) | [REBINDING_SYNTAX.md](REBINDING_SYNTAX.md) | EDR-074 (Principle 5) | SHADOWING (folded), TYPE_ANNOTATION, CLOSURE_CAPTURE | Open (2026-08-19) | Pending |
| Argument syntax: `(a: Int)` vs `(Int a)` | [FUNCTION_ARGUMENT_SYNTAX.md](FUNCTION_ARGUMENT_SYNTAX.md) | ITERATOR_PROTOCOL review (2026-08-05) | FUNCTION_RETURN, FUNCTION_RETURNING_FUNCTION, TYPE_ANNOTATION | Open | Pending |
| Return-type placement: suffix `->` vs prefix | [FUNCTION_RETURN_SYNTAX.md](FUNCTION_RETURN_SYNTAX.md) | ITERATOR_PROTOCOL review (2026-08-05) | FUNCTION_ARGUMENT, BOUNDS_IN_ANGLE_BRACKETS, lambda syntax | Open | Pending |
| Function-type notation / `->` overload | [FUNCTION_RETURNING_FUNCTION.md](FUNCTION_RETURNING_FUNCTION.md) | FUNCTIONS review (2026-08-07) | FUNCTION_RETURN, FUNCTION_ARGUMENT, CLOSURE_CAPTURE, lambda syntax | Open | Pending |
| Eliminate `where`: all bounds in `<>` | [BOUNDS_IN_ANGLE_BRACKETS.md](BOUNDS_IN_ANGLE_BRACKETS.md) | FUNCTIONS review (2026-08-07); revises EDR-086 | FUNCTION_RETURN, FUNCTION_ARGUMENT, FUNCTION_RETURNING_FUNCTION, CLOSURE_CAPTURE, TRAIT_BLANKET_IMPLEMENTATION | Open | Pending |

### Open — coupled by pointer (not relocated)

| Question | Home document | Status | Human review |
|----------|---------------|--------|--------------|
| Creation-time `using` literal syntax (was: capture keyword / lambda syntax) | `../concepts/research/essential/CLOSURE_CAPTURE.md` (semantics locked — EDR-081 Amendment 2026-08-16; syntax open), `../../notes/code-block-semantics.md` | Open (coupled — syntax only) | Pending |
| Blanket-impl syntax | `../concepts/research/important/TRAIT_BLANKET_IMPLEMENTATION.md` | Open (coupled) | Pending |

### Resolved (accepted — record in `what/syntax/`)

| Question | Record | Decision | Status | Human review |
|----------|--------|----------|--------|--------------|
| Invocation context operators `<-` / `|>` | [INVOCATION_SYNTAX.md](../../what/syntax/INVOCATION_SYNTAX.md) | EDR-085 | Resolved | EDR-085 (accepted) |
| Range literal `1..N`, `range(a,b)`, `.step(n)` | [RANGE_SYNTAX.md](../../what/syntax/RANGE_SYNTAX.md) | EDR-083 | Resolved | EDR-083 (accepted) |
| Generics: `<>` parameters, `as` bounds, `+` conjunction | [GENERICS_SYNTAX.md](../../what/syntax/GENERICS_SYNTAX.md) | EDR-086 | Resolved | EDR-086 (accepted) |
| Generator expression `gen(...)` | [GENERATOR_EXPRESSION_SYNTAX.md](../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md) | EDR-092 | Resolved | EDR-092 (accepted) |
| Collection literals `[1,2,3]`, `{"a":1}`, `{1,2,3}` | EDR-041 (concept) | EDR-041 | Resolved (record in `what/syntax/` pending Phase 5) | EDR-041 (accepted) |
| No significant whitespace | — | EDR-076 (rejected) | Rejected (binding negative) | EDR-076 (rejected) |
