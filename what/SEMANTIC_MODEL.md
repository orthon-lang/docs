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

**Model.** Ownership is a semantic invariant, not an enforcement
mechanism: **every value has exactly one owner at any point in the
program** (Semantic Invariant 1). Ownership can be transferred (moved),
but never implicitly duplicated. This holds universally, but its
*practical weight* differs sharply by kind of value:

- **Ordinary values** (`Int`, `String`, `Point`, `List`, `Map`, and
  similar plain data) use pure value semantics (see [Identity](#identity)).
  Assignment copies; there is nothing to "own" beyond the copy itself.
  The vast majority of Orthon code — an estimated 95% — never needs to
  reason about ownership explicitly, because copying eliminates the
  question.
- **Resources** — anything with **exclusive responsibility**: a value
  representing a file handle, a network socket, a unique in-memory
  buffer, or any entity where duplication would be meaningless or unsafe
  — is where Ownership becomes an active concern. For these, Orthon
  adopts a Rust-like model:
  1. Single owner per value.
  2. **Move** transfers ownership; assigning or passing the value
     invalidates the source binding.
  3. **Borrowing** creates temporary access without transferring
     ownership: multiple shared (read) borrows may coexist, or exactly
     one exclusive (write) borrow — never both at once (**shared XOR
     mutable**, the same rule stated as Semantic Invariant 2).
  4. No implicit duplication: copying a resource, if ever needed,
     requires an explicit operation — never a bare assignment.

```
data = create_resource()
other = data              # move: data is now invalid
# use(data)                # compile error: data was moved
use(other)                 # OK
```

**Ownership applies only where exclusive responsibility exists.** This
is a deliberate widening from "only external resources" (an earlier,
narrower framing) to "any case of exclusive accountability" — the test
is not "is this a file handle," but "would silently duplicating this
value violate an invariant the program depends on." Plain data almost
never fails that test; resources almost always do.

**Fresh-value exemption.** A value that has just been constructed and
not yet bound to a name (see [Identity](#identity) § Fresh-value
exemption) has no existing owner to invalidate, so the compiler
transfers it implicitly:

```
consume(create_resource())   # fresh value — no transfer marker needed
existing = create_resource()
consume(existing)             # ERROR in strict mode — existing is a named
                               # binding and requires an explicit transfer
```

**Transfer is semantically explicit (concrete syntax deferred).**
Semantic Invariant 6 requires that the *fact* of an ownership transfer
be visible at the transfer site whenever the source is a named binding.
Two concrete syntaxes are under research — the `@ownership` metaproperty
and the `$` prefix operator (see
[`OWNERSHIP_METAPROPERTY.md`](../how/concepts/research/essential/OWNERSHIP_METAPROPERTY.md),
[`OWNERSHIP_TRANSFER_OPERATOR.md`](../how/concepts/research/essential/OWNERSHIP_TRANSFER_OPERATOR.md)),
alongside the plain `move` keyword baseline both documents compare
against. **This document commits only to the semantic requirement — a
transfer must be syntactically visible — and explicitly defers the
choice of concrete syntax to Phase 5 (Syntax Design).** All three
candidate syntaxes express identical semantics; none is favored here.

**No enforcement mechanism prescribed.** This section defines *what
ownership means*, deliberately not *how the compiler verifies it*.
Rust's static borrow checker, `IDENTITY_BASED_SAFETY.md`'s
ownership-plus-escape-analysis model, and other enforcement strategies
are all compatible implementations of the same semantic contract. Per
`DESIGN_PRINCIPLES.md` § Implementation Independence
(`how/gates/DECISION_VALIDATION.md`'s `IMPLEMENTATION_INDEPENDENCE_GATE`),
the choice of enforcement mechanism belongs to Implementation Strategy
(Phase 7), not to the Core semantic model.

**No GC, no RC by default.** Consistent with [Lifetime](#lifetime),
ownership plus move semantics eliminates the need for reference counting
or tracing collection to answer "when is this value done." Opt-in RC/GC
strategies remain available as Implementation Strategy choices (Phase 7)
for the specific case where sharing is genuinely required (see
`delegate`/`release` in the Source documents), but they are never the
default behavior.

**Source:** `how/concepts/research/essential/OWNERSHIP.md`,
`OWNERSHIP_METAPROPERTY.md`, `OWNERSHIP_TRANSFER_OPERATOR.md`.
`IDENTITY_BASED_SAFETY.md` is not adopted as the enforcement model (its
implicit `.`/`!` inference conflicts with Explicitness), but its
ownership + escape-analysis reasoning about uniqueness is compatible
with — and informed — the "no enforcement mechanism prescribed" stance
above.

### Mutation

> When and how do values change? Are data immutable by default?

**Model.** Orthon is **immutable by default**; mutation is always an
explicit, opt-in act, both at the binding site and at the declaration
site.

**Binding-level mutability:**

```
val x = 42        # immutable binding — explicit keyword
x = 43            # error: x is val

var y = 0         # mutable binding — explicit keyword
y = 1             # OK

count = 42        # declaration by assignment in local context —
                   # creates an immutable binding (equivalent to `val`);
                   # compiler may assist inference

const PI = 3.14   # compile-time constant — a distinct concept from
                   # an immutable runtime binding
```

**Function-level mutability: three exclusive declaration kinds.**
Rather than a `mut` modifier layered onto a single function keyword,
Orthon distinguishes mutation at the declaration level with three
mutually exclusive kinds:

| Kind | Effect on `self` | Return |
|---|---|---|
| **`fun`** | Read-only — never mutates `self` | Always returns a value |
| **`proc`** | Mutates `self`; identity preserved | May return a value or nothing |
| **`new`** | Never mutates `self`; produces a distinct value | Always returns a new value |

```
type List:
    items: Array<Int>

    fun len() -> Int              # read-only
        self.items.len()

    proc append(item: Int)        # mutating, identity preserved
        self.items.push(item)

    new sorted() -> List           # transforming, identity changed
        List(self.items.sorted())
```

The contract lives in the declaration kind, not in a caller-side
annotation: **there is no `mut` at the call site.** A caller of
`nums.append(4)` knows `nums` is mutated because `append` is declared
`proc` — the call site itself carries no extra marker. This directly
implements Semantic Invariant 2 (mutation requires exclusive access) at
the granularity of individual operations: only `proc` operations require
the compiler to establish exclusive access to `self`; `fun` and `new`
never do, because they never mutate.

**Four governing principles** (mirrored from `MUTABILITY.md`, now
expressed in terms of `val`/`var` and `fun`/`proc`/`new` rather than a
`mut` modifier):

1. **Immutable by default** — every binding is `val`-like unless
   declared `var`; every method is `fun`-like unless declared `proc` or
   `new`.
2. **Explicit mutation** — mutation is visible in the declaration kind
   (`var`, `proc`) — never inferred from usage.
3. **Aliasing control** — the compiler tracks whether a value can be
   mutated through multiple references, per Ownership's shared-XOR-
   mutable rule (see [Ownership](#ownership)).
4. **No hidden mutation** — no implicit mutation through property
   setters, operator overloading, or method calls whose declaration kind
   does not say `proc`.

**Mutation requires exclusive access (Semantic Invariant 2).** A `proc`
call is only legal when the compiler can establish that its receiver is
not simultaneously observable through another live reference — the same
requirement Ownership's borrowing rules impose on any exclusive (`&mut`-
style) access. Mutation and Ownership share one invariant, viewed from
two angles: Ownership asks "who may access this," Mutation asks "what
may that access do." See [Cross-Dimension Consistency](#cross-dimension-consistency).

**Deferred to Phase 3:** interior mutability (`Cell`/`RefCell`-style
patterns) and mutation captured by closures are explicitly out of scope
for this semantic model — they are Primitive Block-level questions, not
foundational semantic commitments. Similarly, whether `mut` (a binding
modifier) and `&mut` (a reference modifier) collapse to one keyword or
remain two is left open; the *semantic* distinction between "this
binding may change" (`var`) and "this operation changes its receiver"
(`proc`) is settled here regardless of eventual keyword choice.

**Source:** `how/concepts/research/essential/MUTABILITY.md`,
`EXCLUSIVE_DECLARATIONS.md`

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
