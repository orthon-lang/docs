---
created: 2026-10-18T13:47:48.000Z
title: Normalize lambda syntax: replace pipe-style examples with fn
area: general
severity: minor
files:
  - what/concepts/LAZY_SEQUENCE_GENERATORS.md:87,97,98
  - what/concepts/GENERATORS.md:122,133
  - what/concepts/ITERATION_LOOP.md:153
  - what/concepts/ITERATOR_PROTOCOL.md:104-106,135,136,142,146
  - what/concepts/RANGE.md:76,77
  - how/gates/DECISION_LOG.md
  - how/concepts/research/important/SEQUENCE_METHODS.md:26,27
  - how/concepts/research/important/GENERATORS.md:63
  - how/concepts/research/essential/ITERATOR_PROTOCOL.md:37,38,133,134,140,144
  - how/concepts/research/deferrable/TYPED_HOLES.md:74
  - how/decision_records/architecture/EDR-092-generator-expression-syntax.md
  - how/decision_records/architecture/EDR-093-generator-expression-single-clause.md
  - how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md
---

## Problem

The specification shows two incompatible lambda notations. Collection and
operator docs (GLOSSARY, COMPOSABLE_COLLECTION_OPS, COMMAND_PATTERN_VIA_DELEGATE)
write `fn(x) -> x * 2`; the iterator, range and generator docs write the
pipe-style closure `|n| n * n`. The user decided on 2026-10-18 that lambdas are
written with `fn` (`fn` reads as *mini-function*: a lambda is a mini-function),
so pipe-style examples are stale. The contracts work (postconditions as
`fn(result) -> ...` predicates) depends on one settled lambda syntax.

A second inconsistency sits inside the `fn` form itself: the GLOSSARY Lambda
entry shows the block form (`fn (x: Int) -> Int` followed by an indented body),
while the collection docs show the inline form (`fn(x) -> x * 2`).

## Solution

1. Re-grep to get the live list (line numbers above are from 2026-10-18):
   `grep -rnE '\.(map|filter|for_each|flat_map|fold|reduce|find|any|all|take_while)\(\|' --include='*.md' what how`
   (also check bare `|x| ...` forms outside method calls).
2. Rewrite each pipe-style closure as `fn(...) -> ...`, keeping the example's meaning.
3. EDR-092/093/094 are accepted records: decide between an editorial correction
   of the examples (no status change, as done for the `≈` fix in quick task
   261018-i9n) and an explicit note; do not change status or numbering.
4. Decide whether the GLOSSARY block form and the inline form are both
   canonical; document both in the Lambda entry or drop one.
5. English only; docs-only change; run through `/gsd-quick`.
