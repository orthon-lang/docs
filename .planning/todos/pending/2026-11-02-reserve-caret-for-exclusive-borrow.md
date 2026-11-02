---
title: Reserve caret for exclusive borrow and replace provisional &mut spelling
date: 2026-11-02
priority: high
status: pending
---

## What

Adopt `^` as the single, language-wide spelling for an exclusive mutable
borrow, replacing the provisional `&mut` notation. The three source-side
ownership modes become:

```orthon
for item in collection:    # shared borrow
for item in ^collection:   # exclusive borrow
for item in $collection:   # ownership transfer
```

The change applies consistently to expressions, parameter types, return types,
iteration, iterator protocols, and all explanatory examples. `&mut` remains
only in Rust comparison material or historical alternatives; it is not valid
Orthon syntax.

## Why

The caret communicates exclusive access as a distinct, temporary authority:
`^` is a write-rent, while `$` is an irreversible ownership grant. It makes
exclusive borrowing compact and visible at the use site without introducing a
loop-specific ownership convention.

`^` is reserved for exclusive borrow. XOR must use the named form
`xor(left, right)` so every symbol retains one semantic meaning. `|` is not an
XOR substitute because it already participates in union types and the
provisional distribution operator.

## Suggested action

1. Run the syntax decision pipeline and the language-design gate for the
   language-wide `^` spelling; obtain the required human sign-off before filing
   an Architecture EDR.
2. Amend the Reference, OWNERSHIP, and MUTABILITY decisions that currently own
   the provisional `&mut` spelling, then amend EDR-094 and its accepted syntax
   record.
3. Replace `&mut` with `^` in accepted Orthon syntax and specifications,
   including `what/SEMANTIC_MODEL.md`, `what/PRIMITIVE_BLOCKS.md`,
   `what/EXECUTION_MODEL.md`, `what/syntax/ITERATION_SYNTAX.md`, and relevant
   concept documents.
4. Retain `&mut` only when explicitly describing Rust syntax, a historical
   alternative, or an external interoperability boundary; label every such use
   accordingly.
5. Add a canonical `xor(left, right)` form to the operator/reference material
   when Boolean and bitwise XOR semantics are specified.
