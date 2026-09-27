---
title: Translate the remaining non-English content in repository files
date: 2026-09-27
priority: medium
status: pending
---

## What

`AGENTS.md` §10.9 requires every file in this repository to be in English.
Five tracked files still contain Russian text (measured 2026-09-27):

| File | Cyrillic lines |
|------|----------------|
| `how/concepts/research/essential/EXECUTION_POLICY_HYPOTHESIS.md` | 14 |
| `how/concepts/research/deferrable/DIALECTS.md` | 1 |
| `notes/idioms-overview.md` | 35 |
| `notes/idioms-deep-dive.md` | 48 |
| `notes/prototype-vs-trait.md` | 72 |

`TODO.md` carried four Russian `Ref:` notes (introduced by commit `5f6bab4`) —
translated on 2026-09-27.

## Why it matters

The §9 Review Checklist treats any non-English string as a hard FAIL, so these
files fail their own gate today. The `notes/` and research tiers feed concept
design, so a non-English passage cannot be reviewed or quoted by an English-only
agent.

## Action

1. Translate each file in place, preserving meaning and code fences; leave the
   `orthon` code examples untouched.
2. Re-run the scan:
   `git grep -lP '[\x{0400}-\x{04FF}]' -- '*.md'`
   Seven `.planning/` files also match (graph reports, phase discussion logs, UAT) — GSD
   working artifacts with their own conventions; decide separately whether they are in scope.
3. Consider adding this scan as a fitness function so the rule cannot regress
   silently.

## Note

Surfaced while draining agent memory and verifying that no Russian text was written
back into the repository.
