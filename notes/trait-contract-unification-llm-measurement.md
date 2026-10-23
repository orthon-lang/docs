# Measurement Protocol — LLM Generability of the Unified Contract Construct (EDR-098)

> **Status:** Protocol, revised 2026-10-23 after a small pilot of 2026-10-22. **The
> full measurement has not been run.** The pilot is not a gate measurement (see
> "Pilot of 2026-10-22"); it found defects in this protocol, which are fixed here.
> The verdicts of the gates are unchanged.
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
> **Date:** 2026-10-22; revised 2026-10-23

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
page, and the examples of the hypothesis for that variant. No other Orthon text.

The schema must state, because the pilot showed that models cannot guess them:

- **How a generic contract is applied.** The schema shows `obeys Sort<T>` with the
  type argument (pilot defect D1). For P1 this cannot be shown; see T6.
- **How fields are read in a member.** `self.name`, not `name` (D3).
- **Line comments start with `//`** (D4).
- **The spelling of arguments is the schema's own** (`x: Float`); the old example of
  T10 is written in it (D6).

### Two schema levels

| Level | Content | Used for |
|-------|---------|----------|
| S-full | The rules plus the list of compile errors and the statement that `old` does not exist | All tasks |
| S-blind | The same rules **without** the list of compile errors, without the statement about `old`, and without the sentence that explains the difference between `require` and `requires` | T5, T11, T12 (and T1 to T4 as a check) |

S-full tests whether a model can follow the rules. S-blind tests whether the forms
are discoverable, which the gate's "No hallucination surface" and "Self-correctable"
criteria are about (D5). The pilot ran only S-full.

## Task set (12 tasks)

Revised tasks are marked (revised).

| ID | Form | Task | Counts as correct when |
|----|------|------|------------------------|
| T1 | short form | Write a contract for a square-root function from an English description | Parentheses present, no `self`, named result, `requires` and `ensures` use the named result correctly |
| T2 | function conformance | Write a function that obeys the contract of T1 | `obeys` in the position of the variant; signature matches; no stronger `requires` |
| T3 | member list | Write a type-level contract with three members | Members matched by name and signature; receiver spelling of the variant |
| T4 | type conformance | Make a given type obey the contract of T3 | Every member provided with the same kind and signature; fields read as `self.name` (revised) |
| T5 | mutating member (revised) | Write a counter contract with a reading and a mutating member and the requirement "one more than before" | Member kind is the mutating kind; no `old`; the answer either states an absolute condition or the workaround (a `new` method returning the changed value) or says the condition cannot be written. Silent omission is recorded as E14 |
| T6 | generic (revised) | Write a generic contract and a generic function that obeys it | Under P3: type parameters linked by name, type argument given (`obeys Sort<T>`), bounds not stronger. Under P1: not scored until the syntax pipeline decides how a separate statement binds type parameters (open question 4 of `CONFORMANCE_SYNTAX.md`); the bare name is recorded as an observation |
| T7 | marker | Declare a marker contract and make a type obey it | No members; conformance statement correct |
| T8 | several contracts (revised) | Part (a): make a function obey `Sqrt` and `Logged`, where `Logged(x: Float) -> result: Float`. Part (b): a function must obey two contracts whose results differ (`Sqrt` returns Float, `Flag(x: Float) -> result: Bool`) | (a) Both named in the form of the variant. (b) The model reports that one function cannot match both signatures, instead of writing the statement. Both parts must be right |
| T9 | self-correction | Given the diagnostic and the failing code of one of the R15 errors, fix the code | The fix removes the error without changing the meaning |
| T10 | migration (revised) | Translate a given `trait` / `impl ... for` example of `TRAITS.md` into the variant. The source is written with the schema's argument spelling (`items: [T]`) | Same members and conformance; no leftover `impl`; `print_all` unchanged |
| T11 | confusable words | From two requirements, one about a dependency and one about a precondition, write the signature | `require` used for the dependency and `requires` for the precondition, not swapped |
| T12 | rejection | Given a request that needs a conditions-only contract, answer in the way the rules require | The model writes a named predicate or a parameter list, not the conditions-only header. Under S-blind the model is not told that the header is an error |

## Sampling

- At least two model families, at least one small and one large.
- 10 samples per task per model per variant, default sampling settings, recorded.
- **One task per call** (a fresh context for each task) in the final run, so that
  an earlier answer cannot be copied into a later one (D7). The pilot answered all
  twelve tasks in one response.
- A control run of the same tasks in the current `trait` / `impl ... for` form, to
  separate a regression from a habit that already exists.
- The schema text, the task text, the model identifiers and the settings are stored
  with the results so the run can be repeated.

## Grading

- Each sample is graded pass or fail against the right-hand column of the task
  table; a sample that is syntactically valid but violates a rule of the
  Decision items counts as fail.
- **The grader is not the agent that wrote the protocol and the schema** (D8). The
  pilot was graded by the drafting agent and is not independent. No parser exists
  yet, so grading is by a second model with the rubric, and the author or a delegate
  checks a random 20 percent by hand. Disagreements are recorded and decided by the
  human.
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
| E9 | Result type not convertible, or a stronger bound, or a stronger `requires`, or the type argument of a generic contract missing |
| E10 | Mixing the short and the long form of one contract |
| E11 | Other, described in words |
| E12 | Comment syntax of another language (for example `#`) |
| E13 | Field read without `self` inside a member |
| E14 | Silent omission: a condition that cannot be written is left out without saying so or offering the workaround (recorded, not counted as a failure) |

## Result rule

| Result per variant | Reading |
|--------------------|---------|
| 90 percent or more of all samples pass, and no task below 70 percent | "Predictable generation" Pass for that variant |
| 90 percent or more overall but a task below 70 percent | Flag; the task and its dominant error code are the root cause |
| Below 90 percent, one dominant error code | Flag; a fix (wording, diagnostic, syntax) is proposed and the variant is re-run |
| Below 90 percent, several error codes | Fail for that variant |

T9 is also read on its own: at least 90 percent of the fixes must succeed after one
diagnostic, which is the evidence for "Self-correctable". The S-full and S-blind
results are reported separately; a Pass at S-full only is not enough for "No
hallucination surface".

## Output

A table of pass rates per task, model, variant and schema level, the error codes
with counts, the control run, and the stored inputs. It is recorded as a
measurement entry in `how/gates/DECISION_LOG.md` under the EDR-098 entry, and the
criteria "Predictable generation" and "Self-correctable" of the gate are re-scored.

## Not covered

The protocol measures generation from the schema. It does not measure
generation inside a large existing codebase, nor tool-assisted generation with a
language server.

## Pilot of 2026-10-22

**Status of the pilot.** A small trial of this protocol, made to find its defects.
It is **not** the measurement of the gate and changes no verdict. The inputs and
the condensed outputs were kept outside the repository.

### Design

| Variant | Models | Runs | Tasks per run |
|---------|--------|------|---------------|
| P3: `obeys` in the tail of a function, a statement with a block for a type | small ×2, large ×2 | 4 | 12 |
| P1: always a separate statement | small ×2, large ×2 | 4 | 12 |
| Control: the current `trait` / `impl ... for` form | small ×1, large ×1 | 2 | 7 (those expressible in the old form) |

Words A1 (`contract`, `obeys`). Schema level S-full only. Each run was a fresh
model call that read one file with the rules and the tasks and answered all tasks
in one response. Every run reported that it used only the read tool (two also named the hand-back tool of the harness).

### Results

| Task | Contract variants (n = 8) | Control (n = 2) |
|------|---------------------------|-----------------|
| T1 short form | 8/8 | 2/2 |
| T2 function obeys | 8/8 | not applicable |
| T3 member list | 8/8 | 2/2 |
| T4 type obeys | 8/8 | 2/2 |
| T5 mutating member, no `old` | 8/8 | 2/2 |
| **T6 generic** | **0/8** | 2/2 |
| T7 marker | 8/8 | 2/2 |
| T8 two contracts | 8/8 by form (task defect) | not applicable |
| T9 self-correction | 8/8 | not applicable |
| T10 migration | 8/8 by the protocol of the time | not applicable |
| T11 `require` and `requires` | 8/8 | 2/2 |
| T12 rejection | 8/8 | not applicable |

Totals: contract variants 88/96 (91.7 percent); P3 44/48 and P1 44/48; small model
44/48 and large model 44/48; control 14/14. On the seven tasks that the control
could answer (T1, T3, T4, T5, T6, T7, T11) the contract variants scored 48/56
(85.7 percent) against 14/14. By the result rule this reads as a Flag whose single
root cause is T6; the sample is far too small for any statement about the 90
percent threshold.

### Observations

1. **T6.** All eight wrote `obeys Sort` without the type argument. The schema did
   not show `obeys Sort<T>`, so the failure is a defect of the schema (D1). Under P1
   the gap is deeper: a separate statement has no place to bind `T`.
2. **No habit errors.** No sample used `trait` or `impl`, swapped `require` and
   `requires`, wrote `old`, or wrote the header without parentheses. The schema
   listed these errors, so T9, T11 and T12 were weak tests (D5).
3. **T5.** All ten gave up the postcondition silently; none offered the workaround
   of EDR-097, because the schema did not state it (D2).
4. **T8.** The task was defective: `Logged(x: Float)` has no result, so by the
   matching rule a function cannot obey both contracts. Five of eight noticed it,
   three did not (D2b).
5. **T10.** All eight rewrote `print_all<Printable as T>([T] items)` as
   `(items: [T])`. The documents themselves mix the two argument spellings
   (`FUNCTION_ARGUMENT_SYNTAX.md` is open), so the old snippet conflicted with the
   schema (D6). This is a risk for any migration, independent of the contracts.
6. **Fields and comments.** Three of ten runs, all by the small model, wrote `r`
   instead of `self.r`; one run used `#` for comments (D3, D4).
7. **P3 against P1.** No difference in the share of correct answers. P1 needed one
   more statement per subject and cannot bind a type parameter.
8. **Small against large model.** The same share of correct answers.

### Limitations

- Two samples per cell; no statistical power.
- All tasks of a run were answered in one response; later answers may copy earlier
  ones (D7).
- The grader was the drafting agent, the same model family that wrote the protocol
  and the schema; no independent grader and no hand check of a sample (D8).
- The schema was a one-page version and listed the expected errors (D5).
- No parser; grading by a rubric.

### What the pilot does not show

It does not show that the construct is generable at the 90 percent level, nor that
it is not. It shows that the protocol, with the changes below, can be run, and it
shows two syntax questions that are independent of the contracts: how a separate
conformance statement binds a type parameter (P1) and which argument spelling the
documents use.

## Defects found and the changes made

| ID | Defect found | Change in this revision |
|----|--------------|--------------------------|
| D1 | The schema did not show the type argument of a generic contract; T6 failed 0/8 | The schema shows `obeys Sort<T>`; T6 criterion requires the type argument; under P1 T6 is not scored until the syntax is decided |
| D2 | T5 had no stated workaround; all gave up silently | The S-full schema states the workaround; silent omission is coded E14 |
| D2b | T8 was defective: the second contract had no result | T8 is two-part: (a) with `Logged` returning a result, (b) incompatible results, which must be reported |
| D3 | Field access was not specified; three runs wrote `r` | The schema states `self.name`; E13 added |
| D4 | Comment syntax was not specified; one run used `#` | The schema states `//`; E12 added |
| D5 | The schema listed the expected errors, so T9, T11 and T12 were leading | Two schema levels: S-full and S-blind; T5, T11, T12 are also run at S-blind |
| D6 | The T10 source used `[T] items`, the schema `items: [T]`; all rewrote it | The T10 source is written with the schema's spelling; the argument syntax is noted as a separate open question |
| D7 | Twelve tasks in one response allowed carry-over | One task per call in the final run |
| D8 | The grader was not independent | The grader is a different agent and a hand check of 20 percent is required |
