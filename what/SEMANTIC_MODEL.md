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

**Model.** Orthon defines exactly three visibility levels, each with a
single, unambiguous meaning:

- **`priv`** — visible only within the containing type or function.
  The most restrictive level; used for implementation details.
- **default (no keyword)** — visible within the containing module. This
  is the default for every declaration that carries no visibility
  keyword.
- **`pub`** — visible to any module that imports the declaring module.
  An explicit, opt-in contract; changing a `pub` declaration later is a
  breaking change.

```
type Counter:
    priv count: Int = 0      # visible only inside Counter
    fun peek() -> Int         # module-scoped by default
        self.count
    pub proc increment()      # exported — part of the public API
        self.count += 1
```

**Module is the encapsulation boundary**, not the type. This is why the
*default* (no keyword) visibility is module-scoped rather than
type-scoped: two types in the same module may freely see each other's
default-visibility members, matching the principle that the module — not
the class — is Orthon's primary organizational unit.

**No `protected`, no backdoors.** Orthon deliberately omits a
`protected` level; inheritance-oriented visibility concerns are deferred
to sealed types / open modules (a future, separate mechanism — not part
of this dimension). There is no reflection API, name-mangling trick, or
other runtime mechanism that can reach a `priv` or default-visibility
declaration from outside its permitted scope. This directly instantiates
Semantic Invariant 5.

**Compile-time enforcement only.** Visibility is checked entirely at
compile time. There is no visibility check performed at runtime, and
therefore no way to defeat it dynamically — a `priv` field is not merely
discouraged from external access, it is *unreachable* by any well-typed
program outside its scope.

**Open question (deferred to Phase 5):** whether `pub` on a type implies
`pub` on all of its members, or whether members default to `priv`/module
visibility even when their containing type is `pub`. Both the
`priv`-fields-under-a-`pub`-type example above and Orthon's
minimum-necessary-access principle suggest the latter (members do *not*
automatically inherit their type's `pub`), but the concrete rule and its
syntax are left to Phase 5 (Syntax Design).

**Visibility and Ownership are orthogonal.** A declaration's visibility
level says nothing about how ownership flows through it, and Ownership's
move/borrow rules say nothing about who may call a function. The
resulting edge case is legal and expected: a `priv` type may appear in a
`pub` function's signature (for example, a `pub` constructor function
returning a `priv` internal handle type used only to prevent external
construction, while the function itself is part of the public API). This
is not a special case requiring reconciliation — it is the two
dimensions composing exactly as orthogonal dimensions should. See
[Cross-Dimension Consistency](#cross-dimension-consistency).

**Source:** `how/concepts/research/essential/VISIBILITY_AND_ENCAPSULATION.md`

### Lifetime

> How long do values live? Stack, heap, arena, GC.

**Model.** Every value's lifetime is **scope-based**: a value lives from
its point of creation (or binding) until the end of its enclosing scope
(a `{}` block or a function body), at which point it is deterministically
destroyed. This directly instantiates Semantic Invariant 3.

```
fun process():
    data = load()      # data's lifetime begins here
    transform(data)
    # scope exits here — data is deterministically destroyed
```

**Deterministic destruction.** Destruction happens at a well-defined
point in program order — scope exit or, for a moved-from binding, the
point of the move — never at an unpredictable point chosen by a runtime
collector. This is what makes scope-based lifetime compatible with
Orthon's **No GC by default** commitment (see [Ownership](#ownership)):
there is no need for a collector to *discover* when a value is no longer
needed, because the language already knows, statically, exactly when
that is.

**Reference lifetime ≤ referent lifetime.** Any reference (borrow) into
a value must not outlive the value it points to. This is the lifetime
dimension's contribution to memory safety and composes directly with
Ownership's borrowing rules (see [Ownership](#ownership) § Borrowing):
Ownership defines *what kind* of access a borrow grants (shared vs.
exclusive); Lifetime defines *how long* that access, or the borrowed-from
value itself, may remain valid.

**Value semantics: copies are independent.** Per [Identity](#identity),
assignment copies structurally by default. A consequence for Lifetime is
that a copy is a wholly independent value with its own lifetime, bound
to its own scope — destroying the original does not affect the copy, and
vice versa. Lifetime dependencies (one value's destruction implying
another's) only arise through explicit reference/borrow relationships,
never through ordinary value copies.

**No GC by default.** Garbage collection is not part of Orthon's core
semantic model. Scope-based lifetime plus ownership/move semantics is
sufficient to guarantee that every value is destroyed exactly once, at a
statically known point, without runtime tracing. Opt-in GC or reference-
counted strategies are an **Implementation Strategy** concern (Phase 7),
not a Core Language semantic — a Strategy may choose to *implement*
deterministic destruction using GC/RC bookkeeping internally, but the
*observable* semantics (scope-bound, deterministic destruction order)
must be indistinguishable from the default. This follows
`DESIGN_PRINCIPLES.md` § Semantics Before Optimization.

**Regions/arenas as implementation freedom, not semantics.** Whether the
compiler actually allocates scope-bound values on a stack, in a region,
in an arena, or promotes them to registers is entirely an Allocation
Policy decision within an Implementation Strategy (Phase 7). The
semantic guarantee — a value's lifetime is bound to its scope and ends
deterministically — holds regardless of which allocation mechanism a
given Strategy chooses.

**Source:** `how/concepts/research/essential/SCOPED_RESOURCE_LIFECYCLE.md`,
`OWNERSHIP.md`

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
