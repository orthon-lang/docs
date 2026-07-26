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

**Model.** Orthon uses **value semantics by default**: assignment copies
data structurally, and `==` compares data structurally. Two values are
"the same" when they are structurally equal — not when they occupy the
same memory location.

```
a = Point(1, 2)
b = a              # structural copy — independent value
b.x = 99           # a.x is still 1 — no aliasing
a == b             # false — structurally different after mutation
```

**Identity is not universal.** Most Orthon values (`Int`, `String`,
`Point`, `List`, `Map`, and other ordinary data) have no notion of
identity distinct from their structure — asking "is this the *same*
`Point`?" is meaningless beyond "does it hold the same values?" Identity
only becomes a meaningful question for entities that represent **shared
state or an external resource** — a database connection, a file handle,
an actor mailbox, a mutable cell shared across owners. For those cases,
Orthon requires an explicit, opt-in reference type (a "shared" type);
there is no reference semantics by accident.

**Binding identity vs. value identity.** Two distinct notions must not be
confused:

- **Binding identity** — whether two *names* currently refer to the
  same storage location. This is a compiler/runtime bookkeeping concern
  (aliasing analysis, borrow tracking), not something the language
  exposes as a first-class equality operator on ordinary values.
- **Value identity** — whether two *values* are considered the same
  entity across time, independent of their current structural content.
  This only exists for the explicit shared/reference types described
  above, and is what a hypothetical identity-comparison (`===`) would
  mean if Orthon exposes one.

Structural equality (`==`) always answers the value-identity question in
value-semantics terms ("do these currently look the same"); it never
silently answers the binding-identity question ("are these the same
storage").

**Identity is orthogonal to Ownership.** Whether a value has identity
(is it a plain value or a shared/reference type) is a separate question
from who is *accountable* for it (owns it, borrows it, or has moved it).
A shared/reference type still has exactly one owner of the *reference
itself* at any point (see [Ownership](#ownership)); its distinguishing
property is that the referent it points to is not copied on assignment.
Conversely, a plain value with no identity can still participate fully
in ownership, moves, and borrows. The two dimensions compose freely: a
type's identity model does not constrain its ownership model, and vice
versa.

**Fresh-value exemption.** An unbound temporary — a literal, a
constructor call, or any expression result that has not yet been bound
to a name — has no permanent identity of any kind: it cannot be aliased
because nothing yet holds a second reference to it. This is why fresh
values may be passed directly into ownership-consuming contexts (e.g.
`delegate(List())`) without an explicit transfer marker: there is no
prior owner from which to transfer. Once bound to a name, a value's
identity (or lack thereof) is fixed by its type.

**Reference semantics via explicit opt-in.** When shared, mutable state
is genuinely required, Orthon does not forbid it — it requires the
programmer to opt in to a reference/shared type explicitly. The default
never silently becomes a reference; a type is a value type unless it
declares otherwise. This mirrors Swift's `struct`-by-default,
`class`-by-opt-in model rather than Java's reference-by-default model.

**Implementation freedom.** Value semantics is a **language contract**,
not a promise of eager, physical copying. An Implementation Strategy may
use copy-on-write, copy elision, SSA form, NRVO, or register promotion
to avoid actually copying bytes, as long as the *observable* behavior is
indistinguishable from independent values. This follows directly from
`DESIGN_PRINCIPLES.md` § Semantics Before Optimization and § Intent Over
Implementation: the compiler decides *how*, the language guarantees
*what*.

**Source:** `how/concepts/research/essential/DATA_MODEL.md`,
`VALUE_SEMANTICS.md`. `IDENTITY_BASED_SAFETY.md`'s `.`/`!` operator
model was evaluated and **rejected** for Phase 2 (see
[Cross-Dimension Consistency](#cross-dimension-consistency) and the
EDR's Alternatives Considered) — its implicit mutability inference
violates the Explicitness principle, though its ownership/escape-analysis
machinery informed the [Ownership](#ownership) section's non-prescriptive
stance on enforcement mechanism.

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
