---
quick_id: 260928-mkv
type: execute
mode: quick
status: complete
subsystem: language-spec / generators
tags: [generators, generator-expression, iterator-combinators, EDR-093, single-clause]
completed: 2026-09-28
commits: 2
plan_head_before: 11885cbdf938c205f7007c8540c598016e3f786d
plan_head_after: 70c6941ddff65e89da138a81b408e59a5d0b3a2a
key-files:
  created:
    - how/decision_records/architecture/EDR-093-generator-expression-single-clause.md
  modified:
    - how/decision_records/architecture/EDR-092-generator-expression-syntax.md
    - how/decision_records/architecture/EDR-022-iterator-protocol.md
    - how/decision_records/INDEX.md
    - what/concepts/GENERATORS.md
    - what/concepts/ITERATOR_PROTOCOL.md
    - what/syntax/GENERATOR_EXPRESSION_SYNTAX.md
    - how/syntax/GENERATOR_EXPRESSION_SYNTAX.md
    - how/architecture/PARSER.md
---

# Quick Task 260928-mkv: Amend Generator Expressions to Single-Clause Summary

Recorded EDR-093 (Architecture, Accepted, LOCKED) narrowing `gen(...)` generator
expressions to single-clause — one `for` clause plus zero or more `if` filters —
withdrawing multi-clause `gen(...)` from v0.1 and deferring it to v1.x, then
propagated the narrowing across the concept doc, both syntax docs, and the PARSER
grammar. Per the author-approved addendum (option A), added `.flatten()` as a
first-class StdLib iterator combinator so pure-flatten routing is consistent.

## Commits

1. `40c547b` — `docs: record EDR-093 (generator-expression single-clause amendment)` (Tasks 1-3 + addendum)
2. `70c6941` — `docs: narrow generator expressions to single-clause per EDR-093` (Tasks 4-7)

Both commits end with the required session attribution trailer
(`Co-Authored-By: Claude Opus 4.8` / `Claude-Session: …session_01Vsb5z7HkxDmGupvL8p6cMx`).

## What Was Done

### Commit 1 — record the amendment (+ addendum)
- **EDR-093 created** mirroring EDR-091's shape: header with verbatim LOCKED
  sign-off (`Reviewed-by: mniedre · Date: 2026-09-28 · Verdict: LOCKED`),
  `**Amends:** EDR-092` pointer, numbered Decision (single-clause; multi-clause
  withdrawn; nested-source stays allowed; flatten→`.flatten()` / cartesian→
  `.flat_map()`; pure sugar; Future extension v1.x), Consequences, Compliance,
  Alternatives (multi-clause **deferred, not rejected**), and prose Gate
  Validation. Decision item 4 explicitly states EDR-093 introduces the
  `.flatten()` StdLib combinator and does not rename `flat_map`.
- **EDR-092 back-pointer** added (`**Amended by:** EDR-093`) with its Status,
  sign-off, and item-4 text left byte-for-byte unchanged.
- **INDEX.md**: EDR-093 row added to BOTH the All-Records and By-Category →
  Architecture tables (disambiguated via the `> **Note:**` and `### Process`
  trailing anchors), Accepted count 77 → 78, footer note prepended.
- **Addendum — `.flatten()`**: added to `what/concepts/ITERATOR_PROTOCOL.md`
  Standard Combinators table (next to `.flat_map`), with a free-function
  equivalent `flatten(collection)` and a one-line EDR-093 introduction note
  (`.flatten()` ≡ `.flat_map(|x| x)`).
- **Addendum — EDR-022 pointer**: light "Amended by EDR-093: adds `.flatten()`"
  note added to EDR-022's Amendments section (pointer only; decision not
  rewritten).

### Commit 2 — apply single-clause across the spec
- **GENERATORS.md**: Model section reframed to single-clause; removed the
  multi-`for` example; added the nested-source `gen(x for x in gen(...))` form;
  flatten/cartesian routed to `.flatten()` / `.flat_map(...)`; fixed the
  "no `yield from`" delegation prose; EDR-093 added to the ACCEPTED line, a new
  amendment banner, the Governing-records list, and a new Decision-History bullet.
- **what/syntax/GENERATOR_EXPRESSION_SYNTAX.md**: canonical forms narrowed
  (multi-clause example replaced by nested-source), "All four" → "All of these";
  Rule 3 rewritten to single-clause with a v1.x deferral note; Rule 5 delegation
  wording corrected; EDR-093 added to the banner and Cross-References;
  Rejected-forms entry (`gen(sub)`) left intact.
- **how/syntax/GENERATOR_EXPRESSION_SYNTAX.md**: appended
  `## Amendment (EDR-093, 2026-09-28)` stating the scope is narrowed to
  single-clause and that the pipeline verdicts (Coupling result + gate PASSes)
  are UNAFFECTED; corrected the stale "nested/flattening forms … shown in the
  accepted record" gate line; EDR-093 added to Cross-References.
- **PARSER.md**: `GeneratorExpr` production tightened to
  `"gen" "(" Expr ForClause IfClause* ")"` (repeated clause tail removed); prose
  rewritten to one `for` + zero-or-more `if` with a v1.x multi-clause note;
  attribution now cites EDR-093; `Last updated` bumped to 2026-09-28.

## `.flatten()` Confirmation

`.flatten()` was added to `what/concepts/ITERATOR_PROTOCOL.md` — as a row in the
Standard Combinators (StdLib) table matching the existing column format
(`| Combinator | Signature | Description |`), signature
`Iterator<T>` where `self: Iterator<Iterator<T>>`, one-level flatten semantics —
plus a `flatten(collection)` free-function equivalent and an EDR-093
introduction note. `.flat_map` was not renamed.

## Deviations from Plan

- **Addendum applied (author decision A), overriding the plan's "do not modify
  ITERATOR_PROTOCOL.md" note.** The plan's planner_notes 1 and verification #6
  said to leave ITERATOR_PROTOCOL.md untouched; the author-approved addendum
  explicitly directs adding `.flatten()` there. The addendum takes precedence,
  so ITERATOR_PROTOCOL.md and EDR-022 were modified in Commit 1 as instructed.
- **[Rule 3 - blocking] Corrected relative-link depth for the new
  ITERATOR_PROTOCOL → EDR-093 link.** Initially followed the file's own
  pre-existing `../how/…` pattern (which is a pre-existing broken depth in that
  file); fixed my inserted link to the correct `../../how/…` before committing.
  The pre-existing broken link on line 3 of ITERATOR_PROTOCOL.md was left
  untouched (out of scope — not caused by this task).

## Known Notes (not stubs)

- The multi-clause form `gen(v for s in subs for v in s)` still appears once in
  `what/concepts/GENERATORS.md` (Decision History bullet, line ~217). This is
  the pre-existing **EDR-092 anti-memory** bullet the plan explicitly instructed
  to keep verbatim ("do not rewrite the existing EDR-092 anti-memory bullet").
  It records a now-withdrawn form as history and is immediately followed by the
  new EDR-093 Decision-History bullet documenting the withdrawal. It is NOT a
  canonical/accepted example, so the must_have truth ("multi-`for` `gen(...)`
  form no longer appears as a canonical/accepted example") holds. The plan's
  naive `! grep` regex matches this anti-memory line by design; this is expected
  and correct.

## Verification

All plan verification checks pass:
1. EDR-093 exists, embeds verbatim LOCKED sign-off, carries "Future extension (v1.x)". PASS
2. EDR-092 back-pointer present; sign-off and item-4 text unchanged. PASS
3. INDEX lists EDR-093 in both tables; Accepted = 78; footer note present. PASS
4. No multi-`for` `gen(...)` canonical example in accepted docs / grammar (only the preserved EDR-092 anti-memory bullet remains). PASS
5. Nested-source `gen(x for x in gen(` present in GENERATORS.md. PASS
6. Flatten routed without rename: GENERATORS.md has `.flatten()` and `.flat_map`; ITERATOR_PROTOCOL.md carries `.flatten()` per addendum. PASS
7. PARSER grammar single-clause (`… Expr ForClause IfClause* …`); old `( ForClause | IfClause )*` tail removed. PASS
8. All new EDR-093 cross-references resolve at correct depth; `gen(sub)` rejected-form entry still present. PASS
9. All written content is English. PASS
10. `git log` shows exactly the two `docs:` commits, each with the attribution trailer. PASS

## Metrics

- Commits: 2 (measured via `git rev-list --count ${plan_head_before}..HEAD`)
- Files: 9 changed (1 created, 8 modified), 0 deletions
- No `.planning/` or `.claude/` content touched; ROADMAP.md not touched

## Self-Check: PASSED
- `how/decision_records/architecture/EDR-093-generator-expression-single-clause.md` — FOUND
- Commit `40c547b` — FOUND
- Commit `70c6941` — FOUND
