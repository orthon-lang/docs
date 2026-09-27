---
title: Restore correctness-by-construction glossary terms dropped by commit 8dcf201
date: 2026-09-27
priority: medium
status: pending
---

## What

Six terminology entries were added to `what/GLOSSARY.md` by commit `74f77b9`
("docs: add Separation Logic foundation, invariant classification, and glossary
terms", 2026-08-04) and are absent from the current GLOSSARY:

- Boolean Blindness
- Correctness by Construction (CbC)
- Frame Condition
- Invariant Classification
- Parse Don't Validate
- Refinement Type

## Evidence

- `git show 8dcf201 -- what/GLOSSARY.md` shows these sections being deleted
  (`-### Boolean Blindness`, `-### Correctness by Construction (CbC)`,
  `-### Frame Condition`, `-### Refinement Type`, ...) inside the
  "accept Execution Context Invocation (EDR-085)" commit, which rewrote 488
  lines of GLOSSARY.
- The terms are still used by in-repo documents:
  `notes/parse-dont-validate-idiom.md`, `notes/correct-by-construction-and-ai.md`,
  `how/concepts/research/important/CORRECTNESS_BY_CONSTRUCTION.md`,
  `how/concepts/research/deferrable/FRAME_CONDITIONS.md`,
  `how/concepts/research/deferrable/REFINEMENT_TYPES.md`,
  `how/concepts/research/essential/MAKE_ILLEGAL_STATES_UNREPRESENTABLE.md`.

## Why it matters

`AGENTS.md` §6 and §10 require every new term to be defined in
`what/GLOSSARY.md` with `Source` and `See also` links. The definitions currently
live only in research/notes files, so the ubiquitous-language layer is out of
sync with the documents that use it.

## Action

1. Recover the entry text from the pre-`8dcf201` revision
   (`git show 74f77b9:what/GLOSSARY.md`) — first check whether the current
   GLOSSARY structure renamed or relocated any of them.
2. Re-insert alphabetically with `Source` + `See also` links.
3. Sweep the referencing files for other terms introduced since that commit.

## Note

Surfaced while draining agent memory: the deleted-entry fact survived only in a
private Copilot memory note (`invariant-cbc-research-2026-08-04.md`), which is
why it is filed here before that note is deleted.
