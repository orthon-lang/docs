---
title: Fix stale "if Orthon adopts a dictionary literal" reference in TYPE_ANNOTATION_SYNTAX.md
date: 2026-08-15
priority: low
status: pending
---

## What

In `how/syntax/TYPE_ANNOTATION_SYNTAX.md` § "Colon overload", the text says
"for example, dictionaries, if Orthon adopts a dictionary literal". EDR-041
(2026-07-27) already reserved `{key: value}` as the map-literal candidate, so
the phrase should reference EDR-041 instead of treating it as an open "if".

## Why

The document is a factual comparison record; the "if" makes it look
undecided when the decision is already recorded. Cross-reference accuracy is
an AGENTS.md requirement (§10.8).

## Suggested action

1. Update the sentence to cite EDR-041 and the reserved `{key: value}`
   candidate, noting the `:` overload is a live Phase 5 decision, not a
   hypothetical.
2. Cross-check `COLLECTION_LITERAL_SYNTAX.md` and EDR-041 for current status.
