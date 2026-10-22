# Measurement Protocol — LLM Generability of the Unified Contract Construct (EDR-098)

> **Status:** Protocol only. **The measurement has not been run**; no sampling was
> done when this protocol was written (2026-10-22).
> **Purpose:** gate resolution R13 of
> [EDR-098](../how/decision_records/architecture/EDR-098-trait-contract-unification.md)
> (Proposed): the evidence for the criterion "Predictable generation" of the
> `LLM_GENERABILITY_GATE` (at least 90 percent correct attempts,
> [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md)). The method is
> [`EMPIRICAL_ANALYSIS_METHOD.md`](../how/gates/methods/EMPIRICAL_ANALYSIS_METHOD.md);
> the gate itself is described in [`llm-generability-gate.md`](llm-generability-gate.md).
> **Related:** open question 7 of
> [`CONFORMANCE_SYNTAX.md`](../how/syntax/CONFORMANCE_SYNTAX.md) asks for the same
> measurement for `obeys` against `impl ... for`.
> **Date:** 2026-10-22

## When it can be run

The spelling under test is provisional. The run needs a frozen text of the rules
and a frozen set of diagnostics, so it waits for, or is repeated per candidate of:

1. The position of the conformance statement (P1, P2, P3) and the receiver
   spelling in type contracts (R14, syntax pipeline).
2. The words (A1 `contract` + `obeys`, A2 `trait` + `obeys`).
3. The diagnostics of Decision item 13 of EDR-098 (R15).

Until then the protocol can be run as a **comparison of candidates**: one variant
per combination that is still open. That result is an input to the syntax pipeline,
not a gate verdict.

## What the model receives

Only the canonical schema, as the gate's fail condition requires: the grammar
fragment of EDR-098 for the variant, the rules of Decision items 1 to 13 in one
page, and the examples catalog of the hypothesis for that variant. No other
Orthon text, no hints about the expected errors.

## Task set (12 tasks)

| ID | Form | Task | Counts as correct when |
|----|------|------|------------------------|
| T1 | short form | Write a contract for a square-root function from an English description | Parentheses present, no `self`, named result, `requires` and `ensures` use the named result correctly |
| T2 | function conformance | Write a function that obeys the contract of T1 | `obeys` in the position of the variant; signature matches; no stronger `requires` |
| T3 | member list | Write a type-level contract with three members | Members matched by name and signature; receiver spelling of the variant |
| T4 | type conformance | Make a given type obey the contract of T3 | Every member provided with the same kind and signature |
| T5 | mutating member | Write a stack contract with a mutating `push` and conditions | Member kind is the mutating kind; no `old` |
| T6 | generic | Write a generic function that obeys `Sort<T>` | Type parameters linked by name; bounds not stronger than the contract's |
| T7 | marker | Declare a marker contract and make a type obey it | No members; conformance statement correct |
| T8 | several contracts | Make one function obey two contracts | Both named in the form of the variant; conditions of both respected |
| T9 | self-correction | Given the diagnostic and the failing code of one of the R15 errors, fix the code | The fix removes the error without changing the meaning |
| T10 | migration | Translate a given `trait` / `impl ... for` example of `TRAITS.md` into the variant | Same members and conformance; no leftover `impl` |
| T11 | confusable words | From two requirements, one about a dependency and one about a precondition, write the signature | `require` used for the dependency and `requires` for the precondition, not swapped |
| T12 | rejection | Given a request that needs a conditions-only contract, answer in the way the rules require | The model writes a named predicate or a parameter list, not the conditions-only header |

## Sampling

- At least two model families, at least one small and one large.
- 10 samples per task per model per variant, default sampling settings, recorded.
- A control run of the same tasks in the current `trait` / `impl ... for` form, to
  separate a regression from a habit that already exists.
- The schema text, the task text, the model identifiers and the settings are stored
  with the results so the run can be repeated.

## Grading

- Each sample is graded pass or fail against the right-hand column of the task
  table; a sample that is syntactically valid but violates a rule of the
  Decision items counts as fail.
- No parser exists yet, so grading is by a second model with the rubric, and the
  author or a delegate checks a random 20 percent by hand. Disagreements are
  recorded and decided by the human.
- Each failure is assigned one code from the taxonomy below (the first matching
  code).

## Error taxonomy

| Code | Failure |
|------|---------|
| E1 | Habit from the old form: `trait` or `impl ... for` where the variant uses other words |
| E2 | `require` and `requires` swapped |
| E3 | Receiver spelling wrong for the variant |
| E4 | Short form written without parentheses, or with `self` |
| E5 | `obeys` in a position the variant does not allow, or written twice (P2) |
| E6 | A function made to obey a contract whose member is not `call`, or that has several members |
| E7 | `old` written in a postcondition |
| E8 | Member kind mismatch between contract and implementation |
| E9 | Result type not convertible, or a stronger bound, or a stronger `requires` |
| E10 | Mixing the short and the long form of one contract |
| E11 | Other, described in words |

## Result rule

| Result per variant | Reading |
|--------------------|---------|
| 90 percent or more of all samples pass, and no task below 70 percent | "Predictable generation" Pass for that variant |
| 90 percent or more overall but a task below 70 percent | Flag; the task and its dominant error code are the root cause |
| Below 90 percent, one dominant error code | Flag; a fix (wording, diagnostic, syntax) is proposed and the variant is re-run |
| Below 90 percent, several error codes | Fail for that variant |

T9 is also read on its own: at least 90 percent of the fixes must succeed after one
diagnostic, which is the evidence for "Self-correctable".

## Output

A table of pass rates per task, model and variant, the error codes with counts, the
control run, and the stored inputs. It is recorded as a measurement entry in
`how/gates/DECISION_LOG.md` under the EDR-098 entry, and the criteria
"Predictable generation" and "Self-correctable" of the gate are re-scored.

## Not covered

The protocol measures generation from the schema. It does not measure
generation inside a large existing codebase, nor tool-assisted generation with a
language server.
