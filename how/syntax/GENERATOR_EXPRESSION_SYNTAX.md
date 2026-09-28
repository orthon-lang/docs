# Generator Expression Syntax

> **✅ Resolved via [EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md)
> (2026-09-28).** The accepted specification lives in
> [`what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md)
> and is indexed from the [`what/SYNTAX.md`](../../what/SYNTAX.md) hub.
> This document is the **reasoning trail** — the Syntax Pipeline
> (Stages 1–6b) run that backs the decision, kept as provenance, not a
> binding verdict. The decision is settled; nothing here re-decides it.
>
> **See also:** [`README.md`](README.md) (decision queue),
> [`what/concepts/GENERATORS.md`](../../what/concepts/GENERATORS.md) (semantic spec),
> [`how/SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md) (the pipeline this run follows),
> [`RANGE_STEP.md`](../concepts/research/important/RANGE_STEP.md) (the "resolved reasoning trail" precedent).
>
> **Last updated:** 2026-09-28

## Issue (Why)

What concrete surface form should Orthon use for a generator expression?

The semantics are settled and are **not** reopened here: `emit` is the
sole one-way production keyword (EDR-021), generator expressions are sugar
over it (EDR-050), and iterator combinators supply the desugaring targets
(EDR-032). Only the **surface form** is chosen.

The bare parenthesised form `(expr for x in src if cond)` — locked by
EDR-050 and reaffirmed by EDR-091 — carries **no explicit production
marker**: at its opening delimiter it is indistinguishable from an ordinary
parenthesised expression or a tuple, so a reader (human or LLM) must infer
"this is a generator expression" purely from the interior `for` clause (per
EDR-092 Context). An explicit reserved `gen(` marker makes the construct
self-announcing at its first token, improving learnability and LLM
generability **without adding any semantics** — the construct remains pure
sugar over already-accepted constructs.

## Decision Pipeline Run

> **Pre-filter result (2026-09-28).** Generator expressions already passed
> the Decision Pipeline via EDR-050; this run tests the *surface-form
> hypothesis* only, per [`DECISION_PIPELINE.md`](../process/DECISION_PIPELINE.md)
> and the "Syntax change" row of `DECISION_VALIDATION.md` § Gate Selection.
> This is provenance, not a new binding verdict.

| Q# | Question | Answer |
|----|----------|--------|
| Q1 | What problem are we solving? | Choosing the surface form for a generator expression — an explicit `gen(...)` marker vs. the marker-less bare parenthesised form. The semantics are settled (EDR-021 `emit`, EDR-050 generator expressions); only the surface form is open. |
| Q2 | Is this a language problem or a library problem? | **Language** — surface grammar of the language. `gen` is a reserved production the parser recognises directly, not a stdlib identifier and not a macro (EDR-092 items 2–3). |
| Q3 | Can it be solved with existing primitives? | N/A — generator-expression semantics already exist (EDR-050); this only picks the spelling. No primitive is added or changed. |
| Q4 | Does it violate any Design Principle? | No. It strengthens *one symbol → one meaning* (a dedicated production marker), *named before symbolic* (`gen` is a named keyword marker), and *explicitness* (the intent is visible at the opening token). The minimal-core principle is respected — no new core primitive. |
| Q5 | Adds new semantics (vs sugar)? | **No new semantics — surface form only.** `gen(...)` is pure sugar over the emit-only model (EDR-021 / EDR-091); it desugars to stdlib combinators or an `emit`-based `fun` (EDR-092 item 5). |
| Q6 | Expressible through composition? | Yes — the desugaring is composition over `emit` / `.filter` / `.map` / `.flat_map`; delegation is ordinary iteration + `emit` (equivalently a nested `gen(...)` clause). |
| Q7 | Syntactic sugar over primitives? | **Yes** — sugar over `emit` and stdlib combinators. This is precisely why it is a Syntax Pipeline decision, not a new semantic-concept decision. |
| Q8 | Optimisation, not semantics? | **No** — it is a surface-syntax choice, not an optimisation. Laziness (EDR-050) is unchanged. |
| Q9 | Backward compatibility? | N/A — pre-v1.0. The bare parenthesised form is withdrawn; the `gen(...)` spelling replaces it (anti-memory retained in EDR-092 Decision History). |
| Q10 | Worth adding at all? | Feature: **yes**, already accepted (EDR-050). Surface form: **yes** — the explicit marker resolves the marker-less-form ambiguity flagged in the GENERATORS concept review. |

**Verdict:** Pipeline **PASS** (sugar over settled constructs). No blocker;
this is a Syntax Pipeline decision recorded in EDR-092, not a new semantic
decision. Acceptance path: [`SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md).

## Coupling & Overload Check

Stage 3 requires a symbol/keyword collision sweep for `gen` across all
accepted concepts (**one symbol → one meaning**). The sweep was **run during
this trail's authoring (2026-09-28)**; the actual findings are recorded
below rather than asserted.

**(a) The `gen(` production form** appears only as the one generator-expression
construct, and only in the generator documentation:

- [`what/concepts/GENERATORS.md`](../../what/concepts/GENERATORS.md) — the canonical `gen(...)` production and its examples.
- [`what/concepts/LAZY_SEQUENCE_GENERATORS.md`](../../what/concepts/LAZY_SEQUENCE_GENERATORS.md) — cites `gen(...)` when resolving Open Question 2.
- [`what/CORE_CONCEPTS.md`](../../what/CORE_CONCEPTS.md) — the GENERATORS summary shows `gen(x * x for x in 1..10)`.

All three are the **same** EDR-092 construct. (The `gen(...)` occurrences in
EDR-050 / EDR-091 / EDR-092 and `INDEX.md` are the decision records *about*
this same production, not a second meaning.)

**(b) The prose word `gen`** also occurs, but never as the reserved production,
and only in **non-accepted** (research / notes) layers:

- [`REFLECTION_ALTERNATIVES.md`](../concepts/research/deferrable/REFLECTION_ALTERNATIVES.md) — the informal phrase "Code gen" in a comparison table (deferrable research tier).
- [`notes/primitive-blocks-discussion.md`](../../notes/primitive-blocks-discussion.md) — an `id + gen` generation-counter in a descriptor discussion (notes layer).
- `how/concepts/research/essential/LAZY_SEQUENCE_GENERATORS.md` — a Python code example `def gen(): …` (a foreign-language identifier, not Orthon).

**Verdict: CLEAN.** No `gen(` production ever carried another meaning — every
production-form occurrence is the single EDR-092 generator expression. The
prose-`gen` occurrences are informal words or a foreign-language identifier in
non-accepted layers, not the reserved production. **One symbol → one meaning**
holds for `gen(`. There is no cross-hypothesis coupling: `gen` shares no
symbol/position with the known contention points (`,`, `:`, `<>`, `->`, the
signature tail).

## Syntax Principles

One-line verdict against each of the five principles from
[`what/SYNTAX.md`](../../what/SYNTAX.md) § Syntax Principles:

1. **One concept → one syntax** — **PASS.** `gen(...)` is the single surface form; the bare parenthesised form is withdrawn.
2. **One symbol → one meaning** — **PASS.** The Coupling & Overload sweep confirms `gen(` opens a comprehension and nothing else.
3. **No significant whitespace** — **PASS.** The comprehension clauses are delimited by keywords (`for` / `in` / `if`) and parentheses; indentation is cosmetic.
4. **Named before symbolic** — **PASS.** `gen` is itself a *named* keyword marker rather than a symbolic sigil, which is exactly why the marked form is preferred over the marker-less parenthesised form.
5. **Syntax derived from semantics** — **PASS.** The surface form renders the already-accepted emit-only generator semantics (EDR-021 / EDR-050); it adds no semantics.

## Syntax Acceptance Gates

The four **required** gates from `DECISION_VALIDATION.md` § Gate Selection —
"Syntax change" row — with a one-line PASS verdict each, plus the applicable
optional gate:

- **`LOGICAL_CONSISTENCY_GATE`** — **PASS.** `gen(...)` is consistent with the emit-only model (EDR-021 / EDR-091); it introduces no contradiction with any accepted construct.
- **`CONCEPTUAL_SIMPLICITY_GATE`** — **PASS.** A single reserved marker replaces a marker-less form; no new concept, no new primitive.
- **`ARCHITECTURAL_INTEGRITY_GATE`** — **PASS.** `gen` is grammar, not a stdlib function and not a macro — the language/library boundary stays crisp (EDR-092 items 2–3).
- **`LLM_GENERABILITY_GATE`** — **PASS.** The opening `gen(` token names the intent, so the construct is unambiguous to parse and highly generable.
- **`USER_VALUE_GATE`** (optional — applicable, purely syntactic sugar) — **PASS.** The explicit marker improves readability and generation reliability at no semantic cost.

Per EDR-092 Gate Validation, a full seven-gate table is **not** reproduced for
a locked sugar decision over settled constructs.

## Language Design Gate (relevant items)

One-line verdict on the syntax-relevant items from
[`_language-design.md`](../gates/_language-design.md):

- **Named equivalence** — **PASS.** The block spelling `fun … emit` is the named equivalent of the `gen(...)` expression form; the two are interchangeable (EDR-092 item 6).
- **All canonical forms documented** — **PASS.** The single, filtered, single-delegate, and nested/flattening forms are all shown in the accepted record.
- **Explicitness** — **PASS.** The generator intent is syntactically visible at the opening `gen(` token.
- **Orthogonality** — **PASS.** `gen(` has one context-independent meaning and composes freely (nested `gen(...)` clauses, combinator chains).
- **LLM generability** — **PASS.** No ambiguity an LLM can misparse; the marker removes the tuple/parenthesised-expression confusion of the old form.

## Human Sign-off (Stage 6b)

The solo author's sign-off for this decision is the **pre-existing LOCKED
sign-off recorded in EDR-092**, quoted verbatim:

`Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED`

This sign-off is pre-existing and is **NOT** re-created here. Stage 6b is a
non-automatable gate; agents / GSD flows must not self-certify it. This trail
merely references the sign-off already given.

## Cross-References

- [`README.md`](README.md) — syntax hypothesis inbox and decision queue (Resolved row).
- [`what/syntax/GENERATOR_EXPRESSION_SYNTAX.md`](../../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md) — accepted syntax record (the canonical surface form).
- [`what/SYNTAX.md`](../../what/SYNTAX.md) — hub.
- [`what/concepts/GENERATORS.md`](../../what/concepts/GENERATORS.md) — full semantic specification.
- [EDR-092](../../how/decision_records/architecture/EDR-092-generator-expression-syntax.md) — the deciding record.
- [`how/SYNTAX_PIPELINE.md`](../SYNTAX_PIPELINE.md) — the acceptance pipeline this run follows.
- [`how/process/DECISION_PIPELINE.md`](../process/DECISION_PIPELINE.md) — the 10-question pre-filter.
- [`how/gates/DECISION_VALIDATION.md`](../gates/DECISION_VALIDATION.md) — § Gate Selection ("Syntax change" row).
