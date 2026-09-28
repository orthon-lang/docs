---
quick_id: 260928-mkv
type: execute
mode: quick
autonomous: true
depends_on: []
files_modified:
  - how/decision_records/architecture/EDR-093-generator-expression-single-clause.md
  - how/decision_records/architecture/EDR-092-generator-expression-syntax.md
  - how/decision_records/INDEX.md
  - what/concepts/GENERATORS.md
  - what/syntax/GENERATOR_EXPRESSION_SYNTAX.md
  - how/syntax/GENERATOR_EXPRESSION_SYNTAX.md
  - how/architecture/PARSER.md
commit_shape:
  - "docs: record EDR-093 (generator-expression single-clause amendment)"
  - "docs: narrow generator expressions to single-clause per EDR-093"
must_haves:
  truths:
    - "A reader of the accepted spec (what/concepts + what/syntax) sees generator expressions as one `for` clause plus zero or more `if` filters, with multi-clause explicitly deferred to v1.x."
    - "EDR-093 exists as an Architecture EDR that amends EDR-092 item 4, carries the verbatim LOCKED sign-off, and is discoverable from INDEX.md and from an EDR-092 back-pointer."
    - "The `gen(x for x in gen(...))` nested-SOURCE form remains presented as allowed; the multi-`for` `gen(...)` form no longer appears as a canonical/accepted example anywhere in what/ or in the PARSER grammar."
    - "Flatten/cartesian are routed to iterator combinators (pure flatten `.flatten()`, dependent/cartesian `.flat_map()`) with no combinator renamed."
  artifacts:
    - how/decision_records/architecture/EDR-093-generator-expression-single-clause.md
  key_links:
    - "EDR-093 -> EDR-092 (amends) and EDR-093 -> EDR-022 (combinators) resolve at correct depth."
    - "EDR-092 -> EDR-093 back-pointer banner resolves."
    - "INDEX.md lists EDR-093 in BOTH the All-Records and By-Category > Architecture tables."
---

<objective>
Apply the author-approved re-decision that a `gen(...)` generator expression is SINGLE-CLAUSE only — exactly one `for` clause plus zero or more `if` filters. Multi-clause (nested/flatten/cartesian) `gen(...)` is withdrawn from v0.1 and deferred to v1.x. This narrows decision item 4 of the already-accepted EDR-092; it does not reopen the emit-only model (EDR-021/EDR-091) or the `gen(` reserved-marker surface form (EDR-092).

Purpose: Keep the Orthon v0.1 spec self-consistent after a LOCKED re-decision, recorded as a new EDR that partially amends a prior one — mirroring the EDR-091 precedent (a new EDR that withdraws part of a prior EDR).

Output: EDR-093 (new), an EDR-092 back-pointer, INDEX.md registration, and single-clause propagation across the concept doc, both syntax docs, and the PARSER grammar — delivered as two atomic `docs:` commits.

This is a HOW plan for a settled decision. The decision itself is not in scope to revisit.
</objective>

<locked_decision>
Ground truth (do not revisit — Human Sign-off verbatim: `Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED`):

- A `gen(...)` generator expression has EXACTLY ONE `for` clause plus zero or more `if` filters: `gen(expr for x in src if cond ...)`.
- Multiple `for` clauses in one `gen` (nested/flatten/cartesian) are WITHDRAWN from v0.1.
- A `gen` whose SOURCE is itself a `gen` stays allowed: `gen(x for x in gen(...))` = single-clause with a generator source.
- Flatten/cartesian route to iterator combinators (EDR-022): pure flatten = `.flatten()`; map+flatten (dependent inner / cartesian) = `.flat_map(...)`. KEEP the name `flat_map` — NO combinator rename.
- Forward-compatible: single -> multi-clause widening in v1.x is purely additive and breaks no v0.1 code. Record as "Future extension (v1.x): multi-clause".
</locked_decision>

<planner_notes>
Read before executing — three facts established during planning that change how the tasks are carried out:

1. `.flatten()` DOES NOT currently exist as a documented iterator combinator anywhere under `what/`. A repo sweep found only `.flat_map(fn)` (what/concepts/ITERATOR_PROTOCOL.md line 121, and its research copy). The required-reading note assumed ITERATOR_PROTOCOL "has .flatten / .flat_map"; it has only `.flat_map`. The locked decision nonetheless routes pure flatten to `.flatten()`. Resolution: honor the decision — write `.flatten()` in GENERATORS.md routing guidance as decided, and DO NOT modify ITERATOR_PROTOCOL.md (out of scope; "reference, do NOT rename"). `.flatten()` / `.flat_map()` appear only in prose and fenced code examples, never as markdown links, so they create no dangling cross-reference under AGENTS §10.8. Optionally add one clarifying sentence in GENERATORS.md that `.flat_map(|x| x)` is the identity-flatten equivalent, but the decision's spelling `.flatten()` is what to write. Do NOT rename `flat_map`.

2. The EDR-092 rows in INDEX.md are textually IDENTICAL in the All-Records table (around line 97) and the By-Category > Architecture table (around line 201). A bare `Edit` on the EDR-092 row text will not be unique. Each insertion must anchor on the EDR-092 row PLUS its distinct following line: in All-Records the row is followed (after a blank line) by `> **Note:** EDR-008, EDR-009, ...`; in By-Category the row is followed (after a blank line) by `### Process`. Use those trailing lines as the disambiguating context (AGENTS §10.12).

3. INDEX.md Status Summary currently reads `| Accepted | 77 |`. Increment to 78 (this adds one Accepted EDR).
</planner_notes>

<context>
@/home/user/docs/.claude/CLAUDE.md
@/home/user/docs/AGENTS.md
@/home/user/docs/how/decision_records/architecture/EDR-091-withdraw-bidirectional-yield.md
@/home/user/docs/how/decision_records/architecture/EDR-092-generator-expression-syntax.md
@/home/user/docs/how/templates/_edr-architecture.md
@/home/user/docs/how/decision_records/INDEX.md
@/home/user/docs/what/concepts/GENERATORS.md
@/home/user/docs/what/syntax/GENERATOR_EXPRESSION_SYNTAX.md
@/home/user/docs/how/syntax/GENERATOR_EXPRESSION_SYNTAX.md
@/home/user/docs/what/concepts/ITERATOR_PROTOCOL.md
@/home/user/docs/how/architecture/PARSER.md
</context>

<execution_notes>
- Documentation-only. All written content MUST be English (AGENTS §10.9).
- Two atomic `docs:` commits (AGENTS §10.5, §10.10). Commit 1 = Tasks 1-3 (file the record + register + back-pointer — one intention: "the EDR-093 amendment is recorded and cross-linked"). Commit 2 = Tasks 4-7 (one intention: "apply single-clause narrowing across the spec").
- End EVERY commit message with the session attribution trailer:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
  `Claude-Session: https://claude.ai/code/session_01Vsb5z7HkxDmGupvL8p6cMx`
- Ordering: Task 1 (EDR-093 must exist) before Tasks 2 and 3 (they point at it). Single executor, sequential — run in listed order.
- Do NOT re-run the fn->fun or [T]-><T> migrations. Do NOT create a new Human Sign-off — reuse the LOCKED line verbatim. Do NOT rename `flat_map`. Do NOT touch ITERATOR_PROTOCOL.md, `.claude/`, or `.planning/` contents.
- Author examples in current conventions (`fun`, `<T>`).
- After every edit that adds or modifies a link, confirm it resolves at the correct depth from the SOURCE file (AGENTS §10.8; use `concepts/` and `syntax/` prefixes where required).
</execution_notes>

<tasks>

<!-- ===================== COMMIT 1: record the amendment ===================== -->

<task type="auto">
  <name>Task 1: Create EDR-093 (generator-expression single-clause amendment)</name>
  <files>how/decision_records/architecture/EDR-093-generator-expression-single-clause.md</files>
  <action>
Create the new Architecture EDR by mirroring the shape of EDR-091 (a new EDR that withdraws part of a prior EDR) and the base fields of _edr-architecture.md. Structure and content:

Header block (same field order as EDR-091/EDR-092):
- Title `# EDR-093: Generator Expression — Single-Clause Only`.
- `**Status:** Accepted`, `**Date:** 2026-09-28`, `**Category:** Architecture`, `**Scope:** Subsystem`.
- `**Human Sign-off:**` line embedding the provided LOCKED sign-off VERBATIM: `Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED`. Do NOT invent a new sign-off.
- A partial-amendment pointer line `**Amends:** [EDR-092](./EDR-092-generator-expression-syntax.md) (decision item 4 — the nested / multi-clause scope).` Same-directory link (`./`).
- Then `---`.

`### Context`: EDR-092 adopted `gen(...)` as a reserved comprehension production and its item 4 scoped `gen` to the comprehension form "including nested / multi-clause forms" such as the flattening comprehension. A re-review narrows that scope: a multi-clause comprehension bundles two orthogonal operations (iteration and flattening) into one surface form, whereas single-clause + named iterator combinators keeps each operation explicit and independently composable. State plainly that the emit-only production model (EDR-021 / [EDR-091](./EDR-091-withdraw-bidirectional-yield.md)) and the reserved `gen(` surface marker ([EDR-092](./EDR-092-generator-expression-syntax.md)) are unchanged — only the clause count is narrowed.

`### Decision` (numbered, mirroring EDR-091's numbered decision list):
1. Single-clause only: a `gen(...)` generator expression has exactly one `for` clause plus zero or more `if` filters — `gen(expr for x in src if cond ...)`.
2. Multiple `for` clauses in one `gen` (nested / flatten / cartesian) are WITHDRAWN from v0.1.
3. A `gen` whose SOURCE is itself a `gen` stays allowed: `gen(x for x in gen(...))` — single-clause with a generator source.
4. Flatten / cartesian route to iterator combinators ([EDR-022](./EDR-022-iterator-protocol.md)): pure flatten uses `.flatten()`; map-then-flatten (dependent inner / cartesian) uses `.flat_map(...)`. No combinator is renamed — `flat_map` keeps its name.
5. `gen(...)` remains pure sugar over `emit` and iterator combinators; the emit-only model (EDR-021 / EDR-091) and the reserved `gen(` marker (EDR-092) are unchanged. No new core primitive.
6. Future extension (v1.x): multi-clause. Widening single -> multi-clause in v1.x is purely additive and breaks no v0.1 code.

Include one short `orthon` example block showing the allowed single-clause and nested-source forms (using `fun` / `<T>` conventions), e.g. `gen(x for x in 1..100 if x % 2 == 0)` and `gen(x for x in gen(...))`, plus a one-line "flatten via `.flatten()`; cartesian / dependent via `.flat_map(...)`" note.

`### Consequences`:
- Positive: one `gen` = one iteration + optional filters (orthogonality restored); flatten / cartesian is explicit through named combinators; smaller, unambiguous grammar; higher LLM generability; forward-compatible with a purely additive v1.x widening.
- Negative: nested flattening loses its comprehension spelling in v0.1 and must be written with `.flatten()` / `.flat_map()` or a nested-source `gen`.

`### Compliance` (numbered, name the touched docs): GENERATORS.md single-clause + nested-source form + flatten/flat_map routing + Decision-History bullet; what/syntax record narrowed with a v1.x note; how/syntax reasoning-trail amendment note; PARSER grammar tightened; INDEX registration; EDR-092 back-pointer banner.

`### Alternatives Considered` (table, mirroring EDR-091's format). At least:
- "Keep the multi-clause comprehension in v0.1 (EDR-092 item 4)" | "Deferred to v1.x, NOT rejected: it couples iteration with flattening, iterator combinators (`.flatten()` / `.flat_map()`, EDR-022) already express it, and widening single -> multi-clause later is purely additive. Recorded as Future extension (v1.x)."
- "Add / rename a comprehension keyword to spell flatten" | "Unnecessary surface; the iterator-combinator family (EDR-022) already covers flatten and cartesian. `flat_map` is not renamed."

`### Gate Validation`: prose only (per the EDR-091 / EDR-092 precedent — a locked amendment over settled sugar does not reproduce the full seven-gate table). State that single-clause is a strict narrowing of an already gate-passed sugar: `gen(` still opens exactly one comprehension (the EDR-092 Coupling & Overload "one symbol -> one meaning" result is unaffected), it remains pure sugar (no new primitive), and the emit-only model is untouched.

Cross-reference depth from this file (how/decision_records/architecture/): same-dir EDRs use `./EDR-0NN-...md`; what/concepts and what/syntax use `../../../what/...`; how/syntax uses `../../syntax/...`; how/architecture uses `../../architecture/...`. Verify EDR-021/022/050/091/092 target filenames exist before linking.
  </action>
  <verify>
    <automated>test -f how/decision_records/architecture/EDR-093-generator-expression-single-clause.md && grep -Fq "Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED" how/decision_records/architecture/EDR-093-generator-expression-single-clause.md && grep -Fq "Future extension (v1.x)" how/decision_records/architecture/EDR-093-generator-expression-single-clause.md && grep -Fq "EDR-092-generator-expression-syntax.md" how/decision_records/architecture/EDR-093-generator-expression-single-clause.md && grep -Fq "EDR-022-iterator-protocol.md" how/decision_records/architecture/EDR-093-generator-expression-single-clause.md</automated>
  </verify>
  <done>EDR-093 exists as an Architecture EDR mirroring EDR-091's shape, embeds the verbatim LOCKED sign-off, records the single-clause decision with the v1.x forward-compatible note, defers (not rejects) multi-clause in Alternatives, and links EDR-092 (amends) and EDR-022 (combinators) at correct depth.</done>
</task>

<task type="auto">
  <name>Task 2: Add an "Amended by EDR-093" back-pointer to EDR-092's header</name>
  <files>how/decision_records/architecture/EDR-092-generator-expression-syntax.md</files>
  <action>
Add ONLY a pointer line to EDR-092's header. Anchor on the existing `**Partially supersedes:**` line (currently the last header line before the `---`) and insert a new line immediately after it:
`**Amended by:** [EDR-093](./EDR-093-generator-expression-single-clause.md) (decision item 4 — the nested / multi-clause scope is withdrawn from v0.1; generator expressions are single-clause only).`
Keep a blank line between the new line and the `---` divider (match the surrounding spacing). Do NOT alter EDR-092's Status, Date, Human Sign-off, `**Partially supersedes:**` text, the Decision section (item 4's nested/multi-clause text stays as anti-memory), Consequences, or any other body content. This is a pointer addition only.
  </action>
  <verify>
    <automated>grep -Eq "\*\*Amended by:\*\*.*EDR-093" how/decision_records/architecture/EDR-092-generator-expression-syntax.md && grep -Fq "Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED" how/decision_records/architecture/EDR-092-generator-expression-syntax.md && grep -Fq "including nested / multi-clause forms such as the flattening comprehension" how/decision_records/architecture/EDR-092-generator-expression-syntax.md</automated>
  </verify>
  <done>EDR-092 header carries an "Amended by: EDR-093" pointer; its sign-off line and its decision item-4 text are byte-for-byte unchanged (both still present).</done>
</task>

<task type="auto">
  <name>Task 3: Register EDR-093 in INDEX.md (both tables) and update counts/footer</name>
  <files>how/decision_records/INDEX.md</files>
  <action>
Four edits (see planner_notes 2 and 3 for the identical-row disambiguation and the count value):

1. All-Records table: insert an EDR-093 row immediately after the EDR-092 row. Disambiguate by anchoring on the EDR-092 row together with the blank line and the following `> **Note:** EDR-008, EDR-009, ...` line so the match is unique to the All-Records table. New row:
`| EDR-093 | Architecture | [Generator Expression — Single-Clause Only](architecture/EDR-093-generator-expression-single-clause.md) | Accepted | 2026-09-28 | Amends EDR-092 |`

2. By-Category > Architecture table: insert the same-shaped EDR-093 row immediately after the EDR-092 row there. Disambiguate by anchoring on the EDR-092 row together with the blank line and the following `### Process` heading so the match is unique to the By-Category table. Use the same 6-column row format the neighbouring EDR-092/EDR-091 rows use in that table.

3. Status Summary: change `| Accepted | 77 |` to `| Accepted | 78 |`.

4. Footer: keep the date line `*Last updated: 2026-09-28*` (same day) and PREPEND a one-line note to the running parenthetical describing this change, e.g. "(Generator Expression Single-Clause Amendment added — EDR-093, Architecture; amends EDR-092 item 4, withdrawing multi-clause `gen(...)` from v0.1 and deferring it to v1.x. Prior: ...)". Keep the existing note chain after it.

Both new rows link with the `architecture/EDR-093-...md` relative path (INDEX.md lives in how/decision_records/, so no `../`). Confirm the link resolves.
  </action>
  <verify>
    <automated>test "$(grep -c 'architecture/EDR-093-generator-expression-single-clause.md' how/decision_records/INDEX.md)" -ge 2 && grep -Fq "| Accepted | 78 |" how/decision_records/INDEX.md && ! grep -Fq "| Accepted | 77 |" how/decision_records/INDEX.md && grep -Eq "Last updated: 2026-09-28.*EDR-093|EDR-093.*Last updated: 2026-09-28" how/decision_records/INDEX.md; test $? -eq 0 || grep -q "EDR-093" how/decision_records/INDEX.md</automated>
  </verify>
  <done>EDR-093 appears in BOTH the All-Records and By-Category > Architecture tables with distinct anchors, the Accepted count reads 78, and the footer carries a prepended EDR-093 note.</done>
</task>

<!-- Commit checkpoint: `docs: record EDR-093 (generator-expression single-clause amendment)` covering Tasks 1-3, with the session attribution trailer. -->

<!-- ===================== COMMIT 2: apply single-clause across the spec ===================== -->

<task type="auto">
  <name>Task 4: Narrow GENERATORS.md to single-clause; route flatten/cartesian to combinators</name>
  <files>what/concepts/GENERATORS.md</files>
  <action>
Apply the single-clause narrowing to the accepted concept doc, staying emit-only.

Model section (Generator Expressions): replace the "two canonical shapes — the single form and the nested / flattening form" framing and REMOVE the multi-clause example line (the `gen` with two `for ... in` clauses used as the flattening form). <!-- planner-discipline-allow: gen(v for s in subs for v in s) --> Keep the single-clause examples (`gen(x for x in src if cond)`, squares, evens, names, doubled) and ADD/keep the nested-SOURCE form `gen(x for x in gen(...))`, described as single-clause with a generator source (allowed). Reframe the surrounding prose to "one `for` clause plus zero or more `if` filters".

Delegation / flatten guidance: keep single-source delegation as `for v in sub: emit v`, equivalently the single-clause `gen(v for v in sub)`. Rewrite the flattening guidance so that a multi-clause `gen` is NO LONGER offered: route PURE flatten (a sequence of sequences) to `.flatten()` and DEPENDENT-inner / cartesian to `.flat_map(...)` — both are iterator combinators (EDR-022). Fix the two "equivalently, a nested `gen(...)` clause" mentions (in the "why there is no `yield from`" paragraph and in the `emit` relationship table) so they no longer imply a multi-`for` `gen`; delegation of a single sub-generator stays valid, flattening of many routes to combinators. Per planner_notes 1: write `.flatten()` as decided, do NOT modify ITERATOR_PROTOCOL.md, do NOT rename `flat_map`; optionally note `.flat_map(|x| x)` as the identity-flatten equivalent.

Header banner: add an EDR-093 amendment note mirroring the existing EDR-092 amendment-note style (single-clause re-decision; multi-clause deferred to v1.x; emit-only and the `gen(` marker unchanged; embed the LOCKED sign-off line). Add EDR-093 to the top "ACCEPTED — ..." line and to the "Governing records" list at the bottom, linking `../../how/decision_records/architecture/EDR-093-generator-expression-single-clause.md`.

Decision History: ADD a new bullet (do not rewrite the existing EDR-092 anti-memory bullet): "2026-09-28 — Amended by EDR-093: generator expressions narrowed to single-clause (one `for` + optional `if`s); multi-clause `gen(...)` withdrawn from v0.1 and deferred to v1.x; flatten/cartesian route to `.flatten()` / `.flat_map()`; nested-source `gen(x for x in gen(...))` stays allowed. Generators remain emit-only." Link EDR-093.

Verify every added/edited link resolves from what/concepts/ at correct depth (`../../how/...`, and `concepts/`-prefixed peers where applicable per AGENTS §10.8).
  </action>
  <verify>
    <automated>! grep -Eq 'gen\([^)]*for .+ in .+ for .+ in' what/concepts/GENERATORS.md && grep -Fq "gen(x for x in gen(" what/concepts/GENERATORS.md && grep -Fq ".flatten()" what/concepts/GENERATORS.md && grep -Fq ".flat_map" what/concepts/GENERATORS.md && grep -q "EDR-093" what/concepts/GENERATORS.md</automated>
  </verify>
  <done>GENERATORS.md presents generator expressions as single-clause only, keeps the nested-SOURCE `gen(x for x in gen(...))` form, routes pure flatten to `.flatten()` and dependent/cartesian to `.flat_map()` without renaming any combinator, carries an EDR-093 amendment banner + Decision-History bullet, stays emit-only, and contains no multi-`for` `gen(...)` example.</done>
</task>

<task type="auto">
  <name>Task 5: Narrow what/syntax/GENERATOR_EXPRESSION_SYNTAX.md (canonical forms + Rules)</name>
  <files>what/syntax/GENERATOR_EXPRESSION_SYNTAX.md</files>
  <action>
Update the accepted syntax record to single-clause.

Canonical forms block: remove the nested / flattening multi-`for` example line (the `gen` with two `for ... in`). <!-- planner-discipline-allow: gen(v for s in subs for v in s) --> Keep the single, filtered, and single-delegate forms; optionally add the nested-SOURCE form `gen(x for x in gen(...))`. Update the "All four are the one construct" sentence to the correct remaining count (e.g. "All of these are the one construct ...").

Rules: rewrite Rule 3 (currently "Scope is the comprehension form only, including nested / multi-clause forms") to state the comprehension is ONE `for` clause plus zero or more `if` filters, and ADD a short note "Multi-clause (multiple `for` clauses in one `gen`) is deferred to v1.x — see EDR-093." Point Rule 3 (and the closing "This record implements ..." line / Cross-References) to EDR-093. Update Rules 4/5 wording only where they reference the multi-clause scope; DO NOT weaken the emit-only or pure-sugar rules.

Leave the existing Rejected-forms entry (Rule 6: `gen(sub)`, source-first, infix) intact — do not touch it.

Header banner: add an EDR-093 pointer alongside the EDR-092 acceptance note (single-clause amendment; multi-clause deferred to v1.x). Add EDR-093 to Cross-References linking `../../how/decision_records/architecture/EDR-093-generator-expression-single-clause.md`. Verify link depth from what/syntax/ (`../../how/...`, `../concepts/...`).
  </action>
  <verify>
    <automated>! grep -Eq 'gen\([^)]*for .+ in .+ for .+ in' what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -Fiq "v1.x" what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -q "EDR-093" what/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -Fq "gen(sub)" what/syntax/GENERATOR_EXPRESSION_SYNTAX.md</automated>
  </verify>
  <done>The accepted syntax record shows single-clause canonical forms and Rules (one `for` + optional `if`s), carries a "multi-clause deferred to v1.x" note pointing to EDR-093, keeps the existing Rejected-forms entries intact, and has no multi-`for` example.</done>
</task>

<task type="auto">
  <name>Task 6: Append a single-clause amendment note to how/syntax/GENERATOR_EXPRESSION_SYNTAX.md (reasoning trail)</name>
  <files>how/syntax/GENERATOR_EXPRESSION_SYNTAX.md</files>
  <action>
This file is the reasoning-trail provenance ("not a binding verdict"). APPEND an amendment note (a new short section, e.g. `## Amendment (EDR-093, 2026-09-28)`, placed after the header block or at the end before Cross-References) recording: the surface-form decision is narrowed to single-clause (one `for` clause plus zero or more `if` filters); multi-clause `gen(...)` is withdrawn from v0.1 and deferred to v1.x; and — stated explicitly — the pipeline verdicts are UNAFFECTED: `gen(...)` is still pure sugar over the emit-only model, `gen(` still opens a (now single-clause) comprehension, so the Coupling & Overload "one symbol -> one meaning" result and the Syntax-Principles / Syntax-Acceptance-Gate PASS verdicts all stand. Reference EDR-093 with a link at `../../how/decision_records/architecture/EDR-093-generator-expression-single-clause.md`.

Also correct the single now-stale present-tense claim: the "All canonical forms documented" gate line currently enumerates "the single, filtered, single-delegate, and nested/flattening forms are all shown in the accepted record" — update it so it no longer asserts the nested/flattening form is in the accepted record (e.g. note it as deferred to v1.x per EDR-093). Do not otherwise rewrite the historical pipeline run; it is provenance of what was considered.

Add EDR-093 to Cross-References. Verify link depth from how/syntax/ (mirror the file's own `../../how/decision_records/architecture/...` convention).
  </action>
  <verify>
    <automated>grep -q "EDR-093" how/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -Fiq "single-clause" how/syntax/GENERATOR_EXPRESSION_SYNTAX.md && grep -Fiq "v1.x" how/syntax/GENERATOR_EXPRESSION_SYNTAX.md</automated>
  </verify>
  <done>The reasoning trail carries an appended EDR-093 amendment note stating the scope is narrowed to single-clause with the pipeline verdicts (coupling result + gate PASSes) explicitly unaffected, the one stale "nested/flattening forms are shown in the accepted record" claim is corrected, and EDR-093 is linked in Cross-References.</done>
</task>

<task type="auto">
  <name>Task 7: Tighten the GeneratorExpr grammar in PARSER.md to single-clause</name>
  <files>how/architecture/PARSER.md</files>
  <action>
In the "Generator expression" grammar subsection, tighten the EBNF to single-clause. Replace the production line so it reads exactly:
`GeneratorExpr ::= "gen" "(" Expr ForClause IfClause* ")"`
Keep the `ForClause ::= "for" Pattern "in" Expr` and `IfClause ::= "if" Expr` lines unchanged. Remove the repeated `( ForClause | IfClause )*` tail entirely (one ForClause, zero-or-more IfClause). <!-- planner-discipline-allow: ( ForClause | IfClause )* -->

Update the prose under the grammar: it currently says the grammar "accepts the filtered form ... and the nested / flattening form" with a multi-`for` example — rewrite so it states one `for` clause plus zero or more `if` filters, keeping only the filtered single-clause example. Add a one-line note "Multi-clause (multiple `for` clauses in one `gen`): a future v1.x extension — see EDR-093." Change the "Per EDR-092" attribution to also cite the EDR-093 amendment, linking `../decision_records/architecture/EDR-093-generator-expression-single-clause.md`. Bump the file's `> **Last updated:**` line to 2026-09-28 (the grammar section is being changed). Verify link depth from how/architecture/ (`../decision_records/architecture/...`).
  </action>
  <verify>
    <automated>grep -Fq 'GeneratorExpr ::= "gen" "(" Expr ForClause IfClause* ")"' how/architecture/PARSER.md && ! grep -Fq '( ForClause | IfClause )*' how/architecture/PARSER.md && grep -Fiq "v1.x" how/architecture/PARSER.md && grep -q "EDR-093" how/architecture/PARSER.md</automated>
  </verify>
  <done>PARSER.md's GeneratorExpr production is single-clause (one ForClause, zero+ IfClause), the repeated clause tail is gone, the prose describes one `for` + optional `if`s with a v1.x multi-clause note citing EDR-093, and no multi-`for` example remains.</done>
</task>

<!-- Commit checkpoint: `docs: narrow generator expressions to single-clause per EDR-093` covering Tasks 4-7, with the session attribution trailer. -->

</tasks>

<threat_model>
Documentation-only change: no executable surface, no external/untrusted input, no data flow, and no package-manager installs. There are no trust boundaries to cross and no STRIDE-applicable components; the package-legitimacy gate (T-{phase}-SC) is N/A because no dependency is added. The only integrity concern is internal consistency of the specification, which the per-task verification and the cross-reference checks below cover.
</threat_model>

<verification>
Run from /home/user/docs after all tasks:

1. EDR-093 exists and is well-formed: `test -f how/decision_records/architecture/EDR-093-generator-expression-single-clause.md` and it embeds the verbatim LOCKED sign-off and the "Future extension (v1.x)" note.
2. Back-pointer: EDR-092 has an "Amended by: EDR-093" line while its sign-off and item-4 decision text are unchanged.
3. INDEX: `grep -c 'architecture/EDR-093-generator-expression-single-clause.md' how/decision_records/INDEX.md` >= 2; Accepted count reads 78; footer carries the EDR-093 note.
4. No multi-clause `gen(...)` example survives in any accepted doc or the grammar: `grep -REn 'gen\([^)]*for .+ in .+ for .+ in' what/concepts/GENERATORS.md what/syntax/GENERATOR_EXPRESSION_SYNTAX.md how/architecture/PARSER.md` returns nothing.
5. Nested-SOURCE form present: `grep -Fq "gen(x for x in gen(" what/concepts/GENERATORS.md`.
6. Flatten routing present without rename: GENERATORS.md contains `.flatten()` and `.flat_map`; ITERATOR_PROTOCOL.md is UNMODIFIED (`git diff --name-only` must NOT list what/concepts/ITERATOR_PROTOCOL.md).
7. Grammar tightened: PARSER.md contains `GeneratorExpr ::= "gen" "(" Expr ForClause IfClause* ")"` and no longer contains `( ForClause | IfClause )*`.
8. Cross-references resolve: every new EDR-093 link (from EDR-092, INDEX, GENERATORS, both syntax docs, PARSER) points to an existing file at the correct relative depth (AGENTS §10.8); `gen(sub)` Rejected-forms entry still present in what/syntax.
9. All written content is English (AGENTS §10.9).
10. `git log --oneline -2` shows exactly the two `docs:` commits, each ending with the session attribution trailer.
</verification>

<success_criteria>
- EDR-093 recorded (Architecture, Accepted, verbatim LOCKED sign-off), amending EDR-092 item 4, with multi-clause deferred (not rejected) to v1.x and flatten/cartesian routed to `.flatten()` / `.flat_map()`.
- EDR-092 back-pointer added without touching its decision/sign-off/supersession text.
- INDEX.md registers EDR-093 in both tables; Accepted = 78; footer noted.
- GENERATORS.md, what/syntax, how/syntax, and PARSER.md all present generator expressions as single-clause (one `for` + optional `if`s), keep the nested-SOURCE form, and defer multi-clause to v1.x; no multi-`for` `gen(...)` example remains anywhere.
- `flat_map` is not renamed; ITERATOR_PROTOCOL.md, `.claude/`, and `.planning/` are untouched.
- Exactly two atomic `docs:` commits, each with the attribution trailer.
</success_criteria>

<output>
Two commits on the current branch:
1. `docs: record EDR-093 (generator-expression single-clause amendment)` — Tasks 1-3.
2. `docs: narrow generator expressions to single-clause per EDR-093` — Tasks 4-7.
Update STATE.md's Quick Tasks Completed table per the /gsd-quick workflow (do not edit other .planning/ contents in the doc commits).
</output>