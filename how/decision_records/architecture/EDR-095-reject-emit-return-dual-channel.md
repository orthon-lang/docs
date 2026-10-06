# EDR-095: Reject `emit`+`return` Dual-Channel — Split Stream from Terminal Result

**Status:** Accepted

**Date:** 2026-10-06

**Category:** Architecture

**Scope:** Subsystem

**Human Sign-off:** Reviewed-by: mniedre · Date: 2026-10-06 · Verdict: LOCKED

**Supersedes:** [EDR-052](./EDR-052-emit-as-intermediate-result.md)

---

### Context

EDR-052 (2026-07-27) specified a dual-channel model: a single function may use
`emit` to produce a stream of intermediate values AND a differently-typed
`return` to produce a final value, with the final value multiplexed through
the same iterator object and retrieved via a `.final()` accessor. A 2026-10-06
concept-review conversation found this model has a readability defect: `let x
= f()` is universally expected to bind `f`'s `return` value, not an
`Iterator<T>`, and `.final()` hides a second result channel behind an accessor
that does not read as returning *the* result — a reader cannot tell, from the
call site alone, that `f()`'s true terminal value is one method call away
rather than already bound to `x`.

This surfaced during that conversation, which also drafted a replacement
alternative — an outer plain `fun` owning the terminal result via ordinary
`return`, and a nested `fun` (closing over the outer function's `mut` locals)
owning the `emit` stream and returning `Iterator<T>` — and added it to
`what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`'s "### Nested `emit` Function"
subsection as "disputed, open, not a settled recommendation" (quick task
quick-261006-chn, commit `e5924cc`). The project owner (mniedre) has now
confirmed the reversal below as FINAL: this EDR promotes that alternative from
disputed to ACCEPTED and formally supersedes EDR-052's dual-channel reading.

This decision must stay consistent with already-settled facts: `emit` is
pull-based, consumer-driven production (EDR-021), reaffirmed emit-only and
one-way by EDR-091, which withdrew the bidirectional `yield` form entirely —
there is exactly one production keyword in the language. `gen(...)` is
reserved exclusively for generator-expression comprehension syntax
(EDR-092/EDR-093, both LOCKED) — it is not a general-purpose
function-declaration keyword and is unrelated to this decision's "no new
declaration keyword" point. EDR-085 closed the `execution_context`
constructor set at four members (`defer`/`delegate`/`spawn`/`fork`) and
established that invocation's "what" and "how" are orthogonal axes — the
production-arity question this EDR resolves (does a function emit a variable
count of items or return exactly one value) is a "what" question, not a "how
to execute" question, and must not be answered by adding a fifth constructor.

---

### Decision

1. **Reject the `emit`+`return` dual-channel form.** A single function must
   not use `emit` to produce a stream AND a differently-typed `return` to
   produce a final value multiplexed through the same return path. This is
   the core reversal of EDR-052's Decision section.

2. **Reject the `.final()` accessor.** `.final()` is removed from the
   specification entirely — no iterator exposes a `.final()` method.

3. **Accept the replacement model: split across two functions.** An outer
   plain `fun` owns the terminal result via ordinary `return`; a nested `fun`
   (closing over the outer function's `mut` locals) owns the `emit` stream and
   returns `Iterator<T>`. The following example and its explanatory
   paragraphs are reproduced verbatim from
   `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`'s "### Nested `emit`
   Function — Splitting Stream from Terminal Result" subsection:

   ```orthon
   type Progress(step: Int, current: Int)

   fun show_progress(info: Progress)
       log("Step {info.step}: current result = {info.current}")

   fun process_data(batches: Array<Array<Int>>) -> Int
       let mut result = 0
       let mut step = 0
       let mut spikes = 0

       fun track_batch(batch: Array<Int>) -> Iterator<Progress>
           for value in batch:
               result += value
               step += 1
               if value > 25:
                   spikes += 1
               emit Progress(step: step, current: result)

       for batch in batches:
           for info in track_batch(batch):
               show_progress(info)

       log("spikes: {spikes}")
       return result

   result = process_data([[10, 20], [30], [40, 5, 60]])
   print("Total:", result)
   ```

   `track_batch` is nested inside `process_data` specifically because each
   call must update three closed-over mutable locals of the outer function at
   once — `result`, `step`, and `spikes`. A free-standing top-level function
   doing the same per-batch work would have to return a composite value and
   have every call site in the `for batch in batches:` loop manually unpack
   that composite and fold it back into three separate accumulators on every
   iteration; nesting lets it write directly into the outer bindings through
   closure capture (per `PRIMITIVE_BLOCKS.md`'s mutable-capture rule — `mut`
   on the captured binding, here `result`/`step`/`spikes`, is what makes this
   legal), eliminating that manual fold-back entirely.

   `track_batch` uses `emit` rather than `return` because one call can
   genuinely produce a variable number of `Progress` values — one per element
   of `batch`, so 0..N depending on the batch's length. The general rule: use
   `emit` only when a single invocation can produce a variable count of items
   (including zero); if a function always emits exactly once per call, plain
   `return` is correct instead — wrapping one guaranteed value in an
   `Iterator<T>` only to unwrap it immediately is unjustified overhead.

   This example deliberately does not use `.final()` or combine `emit` with a
   differently-typed `return` in the same function. The terminal result
   (`result`) is owned entirely by the outer `fun process_data`, returned with
   an ordinary `return`; the intermediate stream is owned entirely by the
   nested `fun track_batch`, which returns `Iterator<Progress>`. The two
   result channels are two different functions, not one function multiplexing
   two kinds of output through one return path.

4. **No new function-declaration keyword for emit-producing functions.**
   (a) `gen(...)` is reserved exclusively for generator-expression
   comprehension (EDR-092/EDR-093, LOCKED) and does not apply here. (b)
   `GENERATORS.md` Principle 1 states "`emit` is the sole production keyword
   — there is no second keyword." (c) `fun`/`proc`/`new` (per
   `what/SEMANTIC_MODEL.md` § Mutation) is a closed, three-member
   Mutation-Policy-relative-to-`self` triad, orthogonal to the stream-vs-
   single-value axis this EDR addresses — a function's production arity does
   not change which of the three mutation-declaration keywords it uses.

5. **No new `execution_context` constructor/type for emit-producing
   functions.** Sequential `emit` generators need no execution context at
   all — fully specified by EDR-021/EDR-050 (lazy, pull-based, state machine,
   suspend/resume at `next()`). If intermediate results ever genuinely
   originate from parallel work, EDR-085's existing `spawn()`/`fork()`
   constructors (decision point 6: submissions "join the generator stream,"
   `next()`/`stop()`, `grab`/`gather`) already cover it — no new primitive
   either way.

6. **Emit-applicability criterion.** Use `emit` only when a single invocation
   can genuinely produce a variable count of items (0, 1, or many — not
   statically fixed at exactly one on every path). If every path through a
   function body contains exactly one `emit`, that is a signal to use plain
   `return` instead.

7. **Compiler-diagnostic recommendation (elevates point 6).** Static
   reachability analysis over a function's CFG can determine whether every
   path contains exactly one `emit` (no path has zero or multiple); when so,
   the compiler SHOULD suggest rewriting to plain `return`. This is a Layer 6
   (Exhaustiveness & Completeness) class check per
   `COMPILER_AS_STATIC_ANALYZER.md`'s layered-verification model — the same
   layer that already covers "all function return paths return a value." The
   diagnostic uses that document's established machine-readable shape
   (`code`/`severity`/`location`/`repair_hint`), not a new vocabulary.

8. **Retract EDR-052's Principle 2 as a direct consequence of point 1.**
   EDR-052's `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` Principle 2
   ("Intermediate ≠ lazy — the computation drives itself rather than waiting
   for `next()` calls") is retracted — not as an independently-justified
   decision, but because it has no referent once the dual-channel form it
   described (point 1) is rejected. All `emit` in the language is PULL-based
   (consumer-driven, lazy) per EDR-021/EDR-050/EDR-091; no push-based `emit`
   mode exists anywhere in the language.
   `what/concepts/LAZY_SEQUENCE_GENERATORS.md` already specifies this: "The
   `emit` keyword desugars to storing the current value and control flow
   state, then returning control to the consumer. On the next `next()` call,
   execution resumes at the point after the last `emit`."

---

### Consequences

**Positive:**
- Removes the `let x = f()`/`Iterator<T>` ambiguity and the hidden-second-
  channel problem — a binding's type is always what it visibly looks like.
- Two result channels → two functions → two owners is simpler than one
  function multiplexing two channels through one return path plus an
  accessor.
- Removes `.final()` and its per-iterator final-value storage slot from every
  `emit`-using function's required state-machine surface.
- Keeps `emit` strictly within its existing pull-based contract — no second
  execution discipline, no contradiction between a "computation drives
  itself" principle and the actually-specified consumer-driven protocol.
- Gives programmers, LLM code generators, and (per point 7) the compiler
  itself a crisp, mechanically checkable rule for choosing `emit` vs.
  `return`.
- Zero new syntax, zero new keyword, zero new `execution_context`
  constructor.

**Negative:**
- Computations whose variable-count stream and one-shot final result are
  tightly coupled now need two function declarations (one nested) where
  EDR-052 allowed one — a minor ergonomics cost, mitigated by closure capture
  doing the data-sharing job `.final()`'s stored iterator state used to do.
- The compiler-diagnostic recommendation (point 7) is a SHOULD-level
  recommendation, not yet a fully specified compiler pass — its concrete
  activation/severity is deferred to future Phase 7 / implementation-repo
  work; this EDR commits to the criterion, not to shipping code.
- Other project prose referencing `.final()`/dual-channel beyond the three
  files this change's companion tasks touch
  (`what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`,
  `how/decision_records/INDEX.md`, `what/GLOSSARY.md`) may still exist and
  need a future sweep — this EDR's Compliance only guarantees those specific
  files.

---

### Compliance

1. No accepted (`what/`) document recommends a function using `emit` together
   with a differently-typed `return` retrieved via `.final()`; where such
   content remains for historical reference, it is explicitly marked
   WITHDRAWN.
2. `.final()` does not appear as an active `Iterator<T>` method in any
   accepted specification text — only in withdrawal/historical notes.
3. A computation needing both a variable-count stream and a one-shot terminal
   result is specified as two functions: an outer plain `fun` returning the
   terminal result via `return`, and a nested `fun` (closing over the outer
   function's `mut` locals as needed) returning `Iterator<T>` via `emit`.
4. `fun`/`proc`/`new` remains a closed, three-member declaration triad.
5. `execution_context`'s constructor set (`defer`/`delegate`/`spawn`/`fork`)
   remains closed per EDR-085.
6. All `emit` remains pull-based/consumer-driven per
   EDR-021/EDR-050/EDR-091; no push-based `emit` mode is specified anywhere
   in the language.
7. The emit-applicability criterion and the compiler-diagnostic
   recommendation are documented in
   `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep EDR-052's dual-channel `.final()` model | Rejected for the readability defect described in Context — `let x = f()` does not visibly bind an `Iterator<T>`, and `.final()` hides a second result channel. |
| A new function-declaration keyword (e.g. `gen`) for emit-producing functions | Rejected per point 4 — category error conflating production-arity with the closed Mutation-Policy triad (`fun`/`proc`/`new`); `gen(...)` is already reserved for a different purpose (generator-expression comprehension, EDR-092/EDR-093). |
| A new `execution_context` constructor/type for emit-producing functions | Rejected per point 5 — category error conflating "how to execute" with "how many values to return"; already covered by `spawn()`/`fork()` if parallel origin is ever needed, per EDR-085. |
| A push-based `emit` mode alongside the existing pull-based mode | Rejected per point 8 — would add a second execution discipline to one keyword, with no evidence of need; contradicts EDR-021/EDR-050/EDR-091's settled pull-based model. |
| No change — keep the dual-channel form permanently | Rejected because the project owner found the ambiguity severe enough to warrant reversal before it propagates into implementation or LLM-generated code. |

---

### Gate Validation

Gate selection per `DECISION_VALIDATION.md` § Gate Selection: this is a
semantic refinement that narrows a prior semantic refinement — required gates
`LOGICAL_CONSISTENCY_GATE` and `ARCHITECTURAL_INTEGRITY_GATE`; both optional
gates are also applied: `CONCEPTUAL_SIMPLICITY_GATE` (the model is narrowed —
`.final()` and the dual-channel multiplexing are removed) and
`LLM_GENERABILITY_GATE` (point 7 is specifically a diagnostic/tooling point
bearing on LLM self-correction). `USER_VALUE_GATE` is also applied — the
readability problem is stated in concrete programmer terms, the same way
EDR-052 itself ran its own `USER_VALUE_GATE`.

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | [Working Backwards](../../gates/methods/WORKING_BACKWARDS_METHOD.md) | Pass | The problem is stated in programmer terms: "I expect `let x = f()` to bind `f`'s returned value, not a stream wrapper with a hidden second result behind `.final()`" (thesis 1/2). |
| `LOGICAL_CONSISTENCY_GATE` | [Socratic Method](../../gates/methods/SOCRATIC_METHOD.md) | Pass | Theses 4/5/8 each resolve a term-consistency question by citing an already-settled fact (closed `fun`/`proc`/`new` triad, closed `execution_context` constructor set, pull-based-only `emit`) rather than introducing a new one; no self-referential paradox. |
| `CONCEPTUAL_SIMPLICITY_GATE` | [Scientific Method](../../gates/methods/SCIENTIFIC_METHOD.md) | Pass | Thesis 2 removes `.final()` and its per-iterator state slot; thesis 1 removes the dual-channel multiplexing path. The model shrinks — no new concept is added to replace what is removed (thesis 3 reuses plain `fun` and closures, both already primitive). |
| `ARCHITECTURAL_INTEGRITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass | Thesis 8's retraction operates entirely within EDR-021's existing pull-based model — no layer violation; theses 4/5 explicitly preserve the closed Mutation-Policy triad and the closed `execution_context` constructor set, so no new primitive is introduced at any layer. |
| `LLM_GENERABILITY_GATE` | [Logical Analysis](../../gates/methods/LOGICAL_ANALYSIS_METHOD.md) | Pass (Flag) | Thesis 7's compiler-diagnostic recommendation gives an LLM a mechanically checkable rule (CFG reachability: every path emits exactly once → suggest `return`) for self-correcting generated code; flagged because the diagnostic is a recommendation, not yet a fully specified compiler pass (see Consequences, Negative). |

**Gates not applied:** `IMPLEMENTATION_INDEPENDENCE_GATE` — the decision
operates entirely at the specification level (which functions may combine
`emit` and `return`, and how); it does not depend on or constrain any
Implementation Strategy. `LONG_TERM_MAINTAINABILITY_GATE` — the change
reduces the surface area the specification must maintain (removing
`.final()` and the dual-channel path) rather than adding to it; no new
maintenance burden is introduced.
