---
phase: quick-261006-chn
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
autonomous: true
requirements: [EDR-052]

estimate:
  tokens: 9000
  raw_tokens: 9000
  tasks: 1
  confidence: low

must_haves:
  truths:
    - "what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md documents a second model: a nested `emit` function (owns the intermediate stream) paired with an outer plain `fun` (owns the terminal result), as an alternative to the `.final()` dual-channel pattern."
    - "The existing `.final()` / dual-channel emit+differently-typed-return example and its Behavioural Specification remain in the document, unmodified in substance."
    - "A note in the document flags the dual-channel `.final()` pattern as disputed/open per a concept-review discussion, and points to the nested-function subsection as the preferred alternative where applicable."
    - "Open Questions gains one new numbered item capturing the unresolved choice between the same-function dual-channel pattern and the split outer/nested-function pattern."
  artifacts:
    - what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md
  key_links:
    - "New subsection sits structurally between '### Equivalent Forms' and '### Behavioural Specification'."
    - "EDR-052 badge/status line and Decision History at top/bottom of file are unchanged — no new EDR created, accepted status preserved."
---

<objective>
Document the resolved nested-`emit`-function pattern (outer `fun` owns the terminal result,
nested `fun` owns the `emit` stream by closing over multiple `mut` locals of the outer function)
as a new model inside the ACCEPTED `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` (EDR-052), and
flag the existing `.final()` dual-channel pattern (`emit` + a differently-typed `return` in the
SAME function) as disputed/open per a concept-review discussion — without creating a new EDR,
without changing the accepted status, and without removing the dual-channel model it still
documents as the ACCEPTED spec.

Purpose: a concept-review conversation found the dual-channel pattern has unresolved readability
problems (`let x = f()` looks like it binds `f`'s `return` value, not an `Iterator<T>`; `.final()`
hides a second result channel) and converged on splitting responsibilities across two functions
as a cleaner alternative. This is documentation enrichment of an already-accepted concept, not a
new architectural decision — the dispute is recorded as open, not resolved.

Output: `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` gains one new "### Nested `emit` Function"
subsection (with a disputed-pattern note and a verified Orthon code example), and one new line
under "## Open Questions". No other file changes.
</objective>

<execution_context>
@.claude/gsd-core/workflows/execute-plan.md
@.claude/gsd-core/templates/summary.md
</execution_context>

<context>
# File being edited — read in full before editing, preserve everything not explicitly changed:
@what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md

# Syntax precedents this task's new example must match (do not invent syntax not attested here):
@what/concepts/DATACLASSES.md
@what/concepts/ALGEBRAIC_DATA_TYPES.md
@what/PRIMITIVE_BLOCKS.md
@what/concepts/ITERATION_LOOP.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add nested-emit-function model, disputed-pattern note, and Open Questions entry</name>
  <files>what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md</files>
  <action>
Edit ONLY `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md`. Do not touch the file's accepted-status
badge/EDR-052 reference line, the Issue/Principles/Policy Footprint sections, the existing
"### Intermediate Result Pattern" / "### Semantic Model" / "### Equivalent Forms" subsections and
their code blocks, "### Behavioural Specification", "## Default Strategy", "## Alternative
Strategies", or "## Decision History" — all stay exactly as they are. Do NOT create a new EDR and
do NOT touch any file other than this one.

Insertion point: directly after the closing fence of the "### Equivalent Forms" code block and
directly before the "### Behavioural Specification" heading, insert a new subsection with heading
text exactly: `### Nested \`emit\` Function — Splitting Stream from Terminal Result`

Immediately under that heading, add this disputed-pattern note as a blockquote (use `>` on every
line), wording close to: "**Disputed — open, not a settled recommendation.** The dual-channel
pattern shown above (`emit` plus a differently-typed `return` in the same function, retrieved via
`.final()`) was flagged in a concept-review discussion: readers expect `let x = f()` to bind `f`'s
`return` value, not an `Iterator<T>`, and `.final()` hides a second result channel behind an
accessor that does not read as returning *the* result. The review converged on the pattern below —
splitting the stream and the terminal result across two functions — as the preferred shape where
the choice is available. The dual-channel form above remains the ACCEPTED EDR-052 specification;
this subsection documents an additional model alongside it, not a replacement. See Open Questions
below." Keep the two bolded fragments ("Disputed — open, not a settled recommendation." and "ACCEPTED EDR-052") since they carry the disputed/still-accepted signal the two other edits in this task cross-reference.

After the blockquote, add one Orthon-fenced (```orthon) code example, reproducing exactly this
line sequence (4-space indent per nesting level, matching the file's existing examples — no
trailing colon after `fun`/`type` signature lines, `:` only on `for`/`if` control-flow lines):
`type Progress(step: Int, current: Int)` ; blank line ; `fun show_progress(info: Progress)` ;
(indent 1) `log("Step {info.step}: current result = {info.current}")` ; blank line ;
`fun process_data(batches: Array<Array<Int>>) -> Int` ; (indent 1) `let mut result = 0` ;
(indent 1) `let mut step = 0` ; (indent 1) `let mut spikes = 0` ; blank line ;
(indent 1) `fun track_batch(batch: Array<Int>) -> Iterator<Progress>` ;
(indent 2) `for value in batch:` ; (indent 3) `result += value` ; (indent 3) `step += 1` ;
(indent 3) `if value > 25:` ; (indent 4) `spikes += 1` ;
(indent 3) `emit Progress(step: step, current: result)` ; blank line ;
(indent 1) `for batch in batches:` ; (indent 2) `for info in track_batch(batch):` ;
(indent 3) `show_progress(info)` ; blank line ; (indent 1) `log("spikes: {spikes}")` ;
(indent 1) `return result` ; blank line ;
`result = process_data([[10, 20], [30], [40, 5, 60]])` ; `print("Total:", result)`.
Use the record-construction syntax `type Progress(step: Int, current: Int)` (EDR-039 product-type
form, as in DATACLASSES.md's `type Point(x: Float, y: Float)`), NOT a `type Progress:` block form —
the block-colon form is not the ADT/record spec's accepted syntax. Keep every string literal in
English (project rule: all code snippets must be English) — do not use the Russian strings from
any earlier draft of this example.

After the code block, add exactly three explanatory paragraphs (prose, not inside the code
fence), in this order, each making one point:
(1) `track_batch` is nested inside `process_data` specifically because each call must update
THREE closed-over mutable locals of the outer function at once — `result`, `step`, and `spikes`.
A free-standing top-level function doing the same per-batch work would have to return a composite
value and have every call site in the `for batch in batches:` loop manually unpack that composite
and fold it back into three separate accumulators on every iteration; nesting lets it write
directly into the outer bindings through closure capture (per PRIMITIVE_BLOCKS.md's mutable-
capture rule — `mut` on the captured binding, here `result`/`step`/`spikes`, is what makes this
legal), eliminating that manual fold-back entirely.
(2) `track_batch` uses `emit` rather than `return` because one call can genuinely produce a
variable number of `Progress` values — one per element of `batch`, so 0..N depending on the
batch's length. State this as the general rule: use `emit` only when a single invocation can
produce a variable count of items (including zero); if a function always emits exactly once per
call, plain `return` is correct instead — wrapping one guaranteed value in an `Iterator<T>` only
to unwrap it immediately is unjustified overhead.
(3) This example deliberately does not use `.final()` or combine `emit` with a differently-typed
`return` in the same function. The terminal result (`result`) is owned entirely by the outer `fun
process_data`, returned with an ordinary `return`; the intermediate stream is owned entirely by
the nested `fun track_batch`, which returns `Iterator<Progress>`. The two result channels are two
different functions, not one function multiplexing two kinds of output through one return path.

Finally, open `## Open Questions` near the end of the file (currently 3 numbered items, directly
before `## Decision History`) and append a 4th numbered item, wording close to: "4. Is combining
`emit` with a differently-typed `return` in the same function (the `.final()` pattern, see Model
above) sound, or should it be replaced by the split pattern — an outer terminal-owning `fun` plus
a nested stream-owning `emit` `fun` (see 'Nested `emit` Function — Splitting Stream from Terminal
Result' above)? Raised but not resolved by the concept-review discussion that surfaced it." Do not
renumber or edit items 1-3, and do not add anything to "## Decision History" — this is an open
question, not a decision.
  </action>
  <verify>
    <automated>test -f what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -q 'Nested `emit` Function' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -q 'track_batch' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -qi 'disputed' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -qi 'alternative' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -q '\.final()' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -q 'Behavioural Specification' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && grep -q 'ACCEPTED.*EDR-052\|EDR-052.*ACCEPTED\|✅ ACCEPTED' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md && [ "$(sed -n '/^## Open Questions/,/^## Decision History/p' what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md | grep -c '^[0-9]\+\.')" = "4" ]</automated>
  </verify>
  <done>
`what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` has a new "### Nested `emit` Function — Splitting
Stream from Terminal Result" subsection (between Equivalent Forms and Behavioural Specification)
containing the disputed-pattern blockquote, the `track_batch`/`process_data`/`Progress` Orthon
example with English string literals, and the three explanatory paragraphs; the pre-existing
`.final()` dual-channel example and Behavioural Specification are untouched; Open Questions now
has exactly 4 numbered items; no EDR file was created and the file's accepted-status badge is
unchanged.
  </done>
</task>

</tasks>

<threat_model>
Not applicable. This is a documentation-only change in a specification repository (per CLAUDE.md:
no compiler, runtime, or implementation code present). No code is added, no runtime executes, no
package-manager install occurs, and no trust boundary is crossed — the change edits one Markdown
file. STRIDE register omitted for lack of any applicable component.
</threat_model>

<verification>
- The edited file is valid Markdown, English-only (including the new code example's string
  literals), following the project's heading-level and relative-link conventions.
- The new subsection sits between "### Equivalent Forms" and "### Behavioural Specification"; the
  pre-existing dual-channel example, its Behavioural Specification, and the file's accepted-status
  badge are byte-for-byte unchanged apart from the new subsection and the new Open Questions line.
- Open Questions has exactly one new numbered item (4), appended after the existing 3; Decision
  History is untouched; no new EDR file exists under `how/decision_records/`.
- The new code example uses only syntax attested elsewhere in `what/concepts/` (record
  construction `Type(field: value, ...)`, `let mut`, `{expr}` string interpolation, `for`/`if`
  with trailing `:`, `fun` signatures without trailing `:`) — no invented syntax.
</verification>

<success_criteria>
- `what/concepts/EMIT_AS_INTERMEDIATE_RESULT.md` documents the nested-`emit`-function pattern as
  an additional accepted-doc model, with the three justifications (multi-variable mutable closure
  capture; `emit` only for a genuinely variable per-call item count; two channels via two
  functions, not `.final()` on one) present as prose, not only inside the code block.
- The dual-channel `.final()` pattern is flagged disputed/open and still fully documented as the
  ACCEPTED EDR-052 spec — nothing is deleted, no new EDR is created, no status badge changes.
- Open Questions records the unresolved same-function-vs-split-functions question.
</success_criteria>

<output>
Create `.planning/quick/261006-chn-document-the-resolved-nested-emit-functi/261006-chn-SUMMARY.md`
when done.
</output>
