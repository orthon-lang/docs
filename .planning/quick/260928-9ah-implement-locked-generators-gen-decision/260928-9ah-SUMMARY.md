---
phase: quick-260928-9ah
plan: 01
subsystem: language-spec / decision-records
status: complete
tags: [generators, gen-expression, EDR-092, EDR-086, notation-migration, fn-to-fun]
requires:
  - EDR-050 (parenthesised generator-expression syntax lock)
  - EDR-091 (emit-only generators; item 3 gen(...) rejection)
  - EDR-086 (angle-bracket generics)
  - SEMANTIC_MODEL.md Declaration Kinds (fun / proc / new)
provides:
  - EDR-092 (gen(...) reserved comprehension production, LOCKED)
  - what/concepts/GENERATORS.md gen(...) single surface form + Variant-B delegation
  - LAZY_SEQUENCE_GENERATORS Open Question 2 resolved negatively
  - repo-wide fn->fun typo correction (orthon declaration keyword)
  - repo-wide [T]-><T> generics-notation migration (EDR-086)
affects:
  - how/decision_records/INDEX.md
  - what/CORE_CONCEPTS.md
  - ~90 spec / research / decision-record files (notation migrations)
key-files:
  created:
    - how/decision_records/architecture/EDR-092-generator-expression-syntax.md
  modified:
    - how/decision_records/INDEX.md
    - what/concepts/GENERATORS.md
    - what/concepts/LAZY_SEQUENCE_GENERATORS.md
    - what/CORE_CONCEPTS.md
decisions:
  - "gen(...) is the single generator-expression surface form — a reserved comprehension production (not a stdlib function, not a macro), pure sugar over emit/stdlib combinators with no new core primitive (EDR-092, LOCKED)."
  - "emit does not appear inside lambdas/closures — LAZY_SEQUENCE_GENERATORS OQ2 closed negatively."
  - "fn is not an Orthon Declaration Kind; every orthon-block declaration keyword fn migrated to fun."
  - "Orthon generic type-parameter positions migrated [T]->\<T\> per EDR-086; value/index/array/other-language/retired-syntax brackets preserved."
metrics:
  commits: 3
  files_changed: 93
  tasks: 6
  completed: 2026-09-28
actuals:
  tasks: 6
  commits: 3
plan_head_before: 156f45f419554825659bbb20605d4a4950af85d3
plan_head_after: 78697edd2fb26960bc588b85549e48506e99fdec
---

# Quick Task 260928-9ah: Implement Locked Generators gen(...) Decision Summary

Applied the LOCKED GENERATORS review decisions (Human Sign-off: `Reviewed-by:
mniedre · Date: 2026-09-28 · Verdict: LOCKED`) plus two repo-wide notation
migrations, delivered as exactly three atomic commits.

## Commits

| Commit | Hash | Subject |
|--------|------|---------|
| A (feat) | `2305845` | feat: adopt gen(...) generator-expression production (EDR-092) — TASKS 1+2+5+6 |
| B (fix)  | `84b87ea` | fix: correct fn -> fun declaration keyword in Orthon code blocks — TASK 3 |
| C (refactor) | `78697ed` | refactor: migrate generic type parameters [T] -> <T> (EDR-086) — TASK 4 |

Each commit ends with the session attribution trailer. `.claude/` and
`.planning/` were not touched by any commit.

## What was done

- **TASK 1 (Commit A):** Created `EDR-092-generator-expression-syntax.md`
  (Architecture, Accepted, Scope: Subsystem) with the verbatim Human Sign-off,
  the `gen(sub)` rejection, the citation of LAZY_SEQUENCE_GENERATORS Open
  Question 2 (closed negatively), a `Partially-supersedes` referencing EDR-050 +
  EDR-091, and all examples authored in `fun` + `<T>` (needs no later
  migration). Indexed in INDEX.md's All Records and By-Category Architecture
  tables (exactly two rows); Accepted count 76 -> 77; footer dated 2026-09-28.
- **TASK 2 (Commit A):** Rewrote `GENERATORS.md` to the `gen(...)` single surface
  form (bare parenthesised form removed), added the single and nested/flattening
  shapes (`gen(v for v in sub)`, `gen(v for s in subs for v in s)`), rewrote
  delegation per Variant B (re-emission IS delegation; block `fun...emit` and
  expression `gen(...)` are the same thing; lazy pull-through; `.take` short-
  circuit; no `yield from` because emit-only), recorded `gen(sub)` rejected, and
  pointed the header / Decision History / Affected Documents at EDR-092 via the
  `../../how/` depth. Left `fun` + `[T]` in place for TASK 4 (migrated in C).
- **TASK 5 (Commit A):** Annotated LAZY_SEQUENCE_GENERATORS Open Question 2
  "RESOLVED (negatively) — 2026-09-28, EDR-092" while keeping the original
  question text as anti-memory; OQ 1/3/4/5 not renumbered.
- **TASK 6 (Commit A):** Repointed the CORE_CONCEPTS GENERATORS summary to
  `gen(x * x for x in 1..10)` and added an EDR-092 reference.
- **TASK 3 (Commit B):** Migrated `fn` -> `fun` repo-wide inside ```orthon blocks
  (fence-aware, per-occurrence). 121 declaration-keyword replacements across 33
  files (+ INDEX editorial note). Only declaration-form `fn <ident>(` / `<ident><`
  / `<ident>[` and operator method `fn ==(` were changed.
- **TASK 4 (Commit C):** Migrated Orthon generic `[T]` -> `<T>` repo-wide,
  per-occurrence, fence- and language-aware. 462 replacements across 68 files
  (+ INDEX editorial note). Three pure-Orthon files hard-pass: zero stdlib-generic
  brackets, `Iterator<T>` present, no `<digit>` / `a<i>` over-migration.

## Verification

All six per-task `<verify>` gates were run. Task 1, 5, 6 gates: PASS. Task 3 and
Task 4 residual sweeps were reviewed occurrence-by-occurrence (see below). Task 4
hard gates and all preservation anchors (`a[i]`, `arr[1..3]`, `[1, 2, 3]`,
`data2[0]`, `mut[1, 2, 3]`, `[T: Trait]`, `[Layered Architecture](`) pass. A
line-by-line diff check confirmed every Commit C change (outside the INDEX
editorial note) is a pure `[]`->`<>` swap with no other content altered.

## Residual-sweep notes (verified preserved cases)

**fn->fun residual (TASK 3):** all remaining `fn` inside orthon blocks are verified
non-declaration uses — anonymous/lambda forms (`fn (x) ->`, `fn() ->`, `fn x ->`),
function-type annotations (`fn(T)`, `Option<fn>`), invocation placeholders
(`fn(args)`), and DIALECTS.md's dialect-alias examples (`fn` as a Rust-profile
alias for `def`, with brace-body illustrative syntax).

**[T]->\<T\> residual (TASK 4):** 33 remaining `TypeName[...]` occurrences, each a
verified preserved case:
- Other-language fenced code: Scala (`CONTEXT_PARAMETERS.md`), TypeScript
  (`TYPE_LEVEL_COMPUTATION.md` `T[K]`, `Partial`), Akka/Scala
  (`actor-implementation-taxonomy.md`).
- Other-language prose attributions: PydanticAI/Python (`TODO.md` `RunContext[Deps]`),
  Scala/Haskell (`REFINEMENT_TYPES.md:53`), Python (`LITERAL_TYPES.md`,
  `UNION_INTERSECTION_TYPES.md`), Rust (`VOLDEMORT_TYPES.md:256`).
- Cross-language comparison tables: `notes/idioms-overview.md`,
  `notes/idioms-deep-dive.md` (Scala `Option[T]`/`Try[T]`/`Either[L,R]` columns
  alongside Orthon's `Option<T>`/`Result<T,E>`).
- Mermaid diagram nodes: `how/PHILOSOPHY.md` (`V[Vision]`, `P[Core Principles]`, ...).
- Array-type numerics: `DYNAMIC_COLLECTIONS.md` (`Array[Int, 3]`, `Array[String, 5]`).
- Deliberate retired-syntax anti-memory: `EDR-086` (`List[Cat]` / `List[Animal]`
  variance examples), `GENERICS_SYNTAX.md` (`[T: Trait]`).
- Rejected nominal function-type proposal: `Fn[Int, Bool]` /
  `Fn[Int, String]` in `FUNCTION_RETURNING_FUNCTION.md` and `TYPE_ALIAS.md`
  (positional `Fn[...]` was explicitly rejected 2026-08-17 — its brackets are
  intentional).

## Deviations from Plan

**1. [Plan gate vs. action inconsistency — resolved in favour of the machine gate + must_haves] EDR-092 not named literally in the INDEX.md footer.**
- The Task 1 action text said to prepend a footer note beginning "EDR-092 added — ...",
  but the Task 1 `<verify>` gate asserts `grep -c 'EDR-092' == 2` (exactly the two
  discoverable table rows). Including the literal token "EDR-092" in the footer
  would make the count 3 and fail the gate.
- The must_haves truth requires only that EDR-092 be discoverable in both tables
  and that "the footer is dated 2026-09-28" — it does not require the literal token
  in the footer.
- Resolution: the footer note reads "Generator Expression Syntax added — the
  gen(...) reserved comprehension production ... partially supersedes EDR-050
  (syntax lock) and EDR-091 (item 3 — the prior gen(...) rejection)", dated
  2026-09-28, describing the addition without the literal "EDR-092" token. Gate
  passes (count == 2); footer dated correctly; both table rows carry the link.

**2. [Plan gate unsatisfiable vs. mandatory anti-memory — prioritised must_haves + action text] Task 2 `grep -c 'yield from' == 0` gate not met.**
- The Task 2 `<verify>` gate asserts `grep -c 'yield from' == 0` for GENERATORS.md.
  This is unsatisfiable alongside the plan's own mandatory content: the pre-existing
  header amendment and Decision-History bullets (kept as anti-memory) reference the
  withdrawn `yield` / `yield from` keywords, and the action text explicitly requires
  explaining "why there is NO yield from keyword". The original file already had 4
  lines containing "yield from".
- Resolution: satisfied the authoritative must_haves ("no yield-from because
  emit-only") and action text (Variant-B rationale + preserved anti-memory), which
  necessarily keep the phrase "yield from". All other Task 2 gate conditions pass
  (gen( >= 4, nested form, gen(sub), EDR-092 link at ../../how/, no bare
  parenthesised examples, no `fn`).

**3. [Rule 1 - bug] Collapsed a pre-existing double-keyword typo.**
- `how/concepts/research/essential/COMPILER_AS_STATIC_ANALYZER.md` contained
  `fun fn compute() -> Int` (stray duplicate keyword). Corrected to `fun compute()`
  rather than mechanically producing `fun fun compute()`. Part of Commit B.

## Out-of-scope observations (not fixed)

- Several pre-existing files contain non-English (Russian) prose:
  `notes/prototype-vs-trait.md`, `notes/idioms-deep-dive.md`,
  `notes/idioms-overview.md`, and a heading in
  `how/concepts/research/important/TYPE_LEVEL_COMPUTATION.md`. This task only
  touched `fn`/bracket tokens on specific lines and introduced no non-English
  content; a full translation is a separate concern (AGENTS §10.9) outside the
  scope of these two mechanical migrations.
- `fn` used inline (outside ```orthon fences), e.g. in EDR-021's Gate Validation
  table cell (`fn name() -> Iterator<T>`), was intentionally left unchanged —
  Task 3's discrimination rule is explicitly scoped to declarations inside
  ```orthon fenced blocks.

## Self-Check: PASSED

- EDR-092 file exists; Human Sign-off line present verbatim (1 match).
- INDEX.md contains EDR-092 exactly twice (both tables); Accepted count reads 77.
- Three commits present: `2305845` (feat), `84b87ea` (fix), `78697ed` (refactor),
  each ending with the attribution trailer.
- No `.claude/` or `.planning/` files were modified by any of the three commits.
