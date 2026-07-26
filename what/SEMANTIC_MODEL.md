# Semantic Model

> **⚠️ DRAFT — Placeholder for Phase 2.**
> This document will define *what a program means* at the foundational level.
>
> **Status:** Placeholder — to be filled during Phase 2 of M1.
> **See also:** [`ROADMAP.md`](../when/ROADMAP.md) § Phase 2,
> [`DATA_MODEL.md`](../how/concepts/research/essential/DATA_MODEL.md),
> [`OWNERSHIP.md`](../how/concepts/research/essential/OWNERSHIP.md),
> [`MUTABILITY.md`](../how/concepts/research/essential/MUTABILITY.md),
> [`ALLOCATION.md`](../how/concepts/research/essential/ALLOCATION.md),
> [`EQUALITY.md`](../how/concepts/research/essential/EQUALITY.md),
> [`FOUNDATIONAL_ABSTRACTIONS.md`](../how/concepts/research/essential/FOUNDATIONAL_ABSTRACTIONS.md),
> [`PRIMITIVE_BLOCKS.md`](PRIMITIVE_BLOCKS.md)

---

## Scope

This document defines the **semantic foundation** of Orthon — the answer
to *"What does a program mean?"* It is distinct from:

- **Syntax** — how programs are written (see [`SYNTAX.md`](SYNTAX.md))
- **Execution Model** — how programs are executed (see [`EXECUTION_MODEL.md`](EXECUTION_MODEL.md))
- **Optimization Model** — what is guaranteed vs. optimised (see [`OPTIMIZATION_MODEL.md`](OPTIMIZATION_MODEL.md))

---

## Semantic Invariants

These six rules are cross-cutting — they apply to *all* six dimensions
below, not to any single one. Where a dimension's section states a rule
that appears to narrow or restate one of these invariants, the dimension
section is the specialization and this list is the general law it obeys.

1. **Every value has exactly one owner at any point in the program.**
   Ownership may move, but it is never implicitly duplicated. See
   [Ownership](#ownership).
2. **Mutation requires exclusive access; read access may be shared.**
   Many readers may observe a value concurrently; at most one writer may
   change it, and no reader may observe a value mid-mutation. See
   [Mutation](#mutation) and [Ownership](#ownership) § Aliasing.
3. **Every value has a well-defined lifetime tied to its scope.**
   Lifetime is never open-ended by default; it is anchored to a lexical
   scope and ends deterministically. See [Lifetime](#lifetime).
4. **All control flow produces a value (expression-oriented).**
   There is no statement/expression split — `if`, `when`, `try`, and
   blocks are all expressions with a well-defined value. See
   [Evaluation](#evaluation).
5. **Visibility is a compile-time guarantee with no runtime bypass.**
   What is not visible cannot be reached by any mechanism — no
   reflection, no naming-convention workaround. See
   [Visibility](#visibility).
6. **Ownership transfer is semantically explicit (syntax TBD in Phase 5).**
   The *fact* that a value's ownership changes hands must always be
   visible at the transfer site. Which concrete syntax marks that fact
   (`move`, `$`, `@ownership`) is a Phase 5 decision; that some marker
   exists is a Phase 2 semantic commitment. See [Ownership](#ownership)
   § Transfer.

These invariants are stated as absolutes because they are the semantic
floor every Implementation Strategy must guarantee (per
[`DESIGN_PRINCIPLES.md`](../how/DESIGN_PRINCIPLES.md) § Semantics Before
Optimization) — a Strategy may implement them with a borrow checker,
escape analysis, or another mechanism, but it may never relax them.

---

## Semantic Dimensions

### Identity

> What does it mean for two values to be "the same"?

<!-- To be filled during Phase 2 -->

### Ownership

> Who owns data? Linear, shared, borrowed?

<!-- To be filled during Phase 2 -->

### Mutation

> When and how do values change? Are data immutable by default?

<!-- To be filled during Phase 2 -->

### Evaluation

> When are expressions evaluated? Eager, lazy, mixed?

<!-- To be filled during Phase 2 -->

### Visibility

> What is visible where? Scoping rules, modules, privacy.

<!-- To be filled during Phase 2 -->

### Lifetime

> How long do values live? Stack, heap, arena, GC.

<!-- To be filled during Phase 2 -->

---

## Cross-Dimension Consistency

<!-- To be filled during Phase 2 — check conflicts between dimensions -->

---

## Relationship to Design Principles

<!-- To be filled during Phase 2 — verify each dimension against DESIGN_PRINCIPLES.md -->

---

## EDR

- **EDR-NNN:** Acceptance of the Orthon Semantic Model
  <!-- Created during Phase 2 -->
