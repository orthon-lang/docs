# Semantic Model

> **✅ ACCEPTED — [EDR-013](../how/decision_records/architecture/EDR-013-semantic-model.md).**
> This document defines *what a program means* at the foundational level,
> synthesizing 10 essential/important-tier research documents into six
> semantic dimensions: Identity, Ownership, Mutation, Evaluation,
> Visibility, Lifetime.
>
> **Status:** Accepted 2026-07-27 (Phase 2 of M1). Two open items are
> intentionally deferred to Phase 5 (Syntax Design) — see
> [Design Principles verification](#relationship-to-design-principles)
> § Net Flags.
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

**Model.** Orthon is **expression-oriented**: every control flow
construct produces a value, and there is no statement/expression split
in the grammar. This directly implements Semantic Invariant 4.

```
status = if age >= 18 then "adult" else "minor"

greeting = {
    prefix = "Hello"
    name = load_name()
    "$prefix, $name!"      # last expression in the block is its value
}
```

- **`if`, `when`, `try`, and blocks are all expressions.** A block's
  value is its last expression. Because `if` is already an expression, a
  separate ternary operator is redundant and is not part of the core.
- **Compiler-enforced exhaustiveness.** Since every branch of an `if`/
  `when`/`try` must produce a value of a consistent type, a missing
  branch is a compile-time type error — not a runtime null or an
  unreachable-code surprise.
- **Explicit discard.** When an expression's value is intentionally
  unused, that intent must be visible: `_ = expr`. Silently dropping a
  value is not permitted to happen invisibly.
- **Definite assignment analysis.** Every binding must be assigned on
  every path before it is read; the compiler statically verifies this
  (shared with [Mutation](#mutation)'s `val`/`var` model and
  `DECLARATION_BY_ASSIGNMENT.md`'s declaration-by-first-assignment
  rule).

**Eager by default; laziness is explicit.** Expressions evaluate
immediately at the point they are reached, in program order. Function
arguments are evaluated before the call executes. Deferred computation
requires an explicit marker (a `sec` keyword or equivalent — concrete
syntax deferred to Phase 5); there is no implicit laziness anywhere in
the default evaluation model.

**Defined evaluation order.** Sub-expressions evaluate left-to-right.
This is a semantic commitment, not an optimization detail: per
`DESIGN_PRINCIPLES.md` § Deterministic Behavior, the same source must
produce the same observable order of side effects regardless of
Implementation Strategy or optimization level.

**Side-effect visibility within expressions.** Because expressions can
appear nested arbitrarily deeply (an `if` inside a function argument
inside another `if`), the left-to-right evaluation order above is what
makes side effects predictable: a reader can determine the order in
which any two sub-expressions' side effects occur purely from their
syntactic position, without knowing anything about the compiler.

**No statement/expression grammar split.** Side-effecting constructs
(assignments, calls returning nothing meaningful) still produce a value
— the `Unit` value — when they appear in expression position. This keeps
the grammar uniform: nothing in Orthon is "only" a statement.

**Sequence production via `emit`.** Producing a sequence of intermediate
results (what other languages call a generator) uses `emit`, not
`yield`. `yield` was considered and **rejected**: in Python, `yield`
conflates too many responsibilities (suspension point, value production,
two-way communication via `.send()`), which conflicts with Semantic
Purity (`DESIGN_PRINCIPLES.md` § Semantic Purity — one symbol, one
meaning). `emit` is scoped to exactly one responsibility: producing the
next value of a [Sequence](GLOSSARY.md#sequence). A Sequence
describes *what* the result is, not *how* it is produced, and remains an
ordinary value — it can be returned, stored, passed, transformed, or
consumed incrementally like any other value, consistent with
`FOUNDATIONAL_ABSTRACTIONS.md`'s canonical-forms treatment (`emit value`,
`return sequence(value)`, and `return value ->` as equivalent forms).

**Source:** `how/concepts/research/essential/EXPRESSION_ORIENTED_LANGUAGE.md`,
`how/concepts/research/important/DECLARATION_BY_ASSIGNMENT.md`

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

Six dimensions produce fifteen pairwise interactions. Per
`DESIGN_PRINCIPLES.md` § Orthogonality, the goal for each pair is
**orthogonal composition** — each dimension answers a different
question, and nothing about one dimension's answer constrains or is
inferred from another's. Where a pair is not fully orthogonal (an
edge case exists), the resolution is documented so it is settled once,
not rediscovered per-feature in later phases.

| # | Pair | Relationship | Resolution |
|---|------|--------------|------------|
| 1 | Identity ↔ Ownership | Orthogonal | Binding identity (does a name alias a storage location) is distinct from value identity (does a shared/reference type persist across mutation). A value's identity model (plain vs. shared type) never constrains its ownership model — see detail below. |
| 2 | Identity ↔ Mutation | Orthogonal | Structural equality (`==`) is defined in terms of a value's *current* structure — a `proc` mutation changes that structure and therefore can change `==` results, but this is a consequence of Mutation acting on data, not an interaction requiring special-casing. Identity's "is this the same entity" question (for shared types) survives mutation by construction (identity ≠ structure), so mutating a shared-identity value never breaks its identity. |
| 3 | Identity ↔ Evaluation | Orthogonal | Whether an expression's result is a fresh (identity-exempt) value or a named binding does not affect *when* it is evaluated (eager by default) or *what value* the expression produces. |
| 4 | Identity ↔ Visibility | Orthogonal | A type's visibility level says nothing about whether it has identity (plain value vs. shared/reference type), and vice versa — a `priv` shared-state type and a `pub` plain-value type are both fully legal, independent choices. |
| 5 | Identity ↔ Lifetime | Orthogonal | A value copy (per value semantics) has both independent identity-relevant structure and an independent lifetime — the two properties travel together for plain values by construction, and for shared/reference types, lifetime tracks the referent while identity tracks the reference's persistence across the program; neither constrains the other's rules. |
| 6 | Ownership ↔ Mutation | Bridges via shared invariant | Both dimensions instantiate Semantic Invariant 2 (mutation requires exclusive access) from different angles: Ownership defines *who may hold access* (single owner, shared XOR mutable borrows); Mutation defines *what that access permits* (`fun`/`proc`/`new`). A `proc` call is only legal exactly when Ownership's exclusivity rule is satisfied for its receiver — this is the sharpest cross-dimension coupling in the model, and it is coupling by design, not accident: it is the same invariant, not two independent rules that happen to agree. |
| 7 | Ownership ↔ Evaluation | Orthogonal | Eager evaluation determines *when* an expression's value is computed; that timing is independent of whether the resulting value is then owned, moved, or borrowed. A moved value and a copied value are evaluated identically before the transfer/copy decision applies. |
| 8 | Ownership ↔ Visibility | Edge case documented | A `priv` type may appear in a `pub` function's signature (e.g., a public constructor returning an internal handle type used to prevent external construction). Visibility governs *who may name/reference* a declaration; Ownership governs *what happens to the value's accountability* once referenced. The two never need to agree, because they answer unrelated questions — see the [Visibility](#visibility) section for the full example. |
| 9 | Ownership ↔ Lifetime | Bridges via scope | A move ends the source binding's meaningful lifetime early (before its enclosing scope exits) without triggering destruction — destruction happens once, at the new owner's scope exit. Ownership decides *whether* a given binding is still the value's owner at a point in the program; Lifetime decides *when* the value still owned is destroyed. These compose without conflict because Lifetime only ever asks its question of the *current* owner. |
| 10 | Mutation ↔ Evaluation | Interacts at evaluation order | Because Orthon is expression-oriented, a `proc` call can appear in expression position (e.g., inside an `if` condition or as a sub-expression of a larger expression). The defined left-to-right evaluation order (see [Evaluation](#evaluation)) is what makes the *order* of such mutating sub-expressions' side effects predictable — without a defined order, mutating expressions nested in larger expressions would have implementation-dependent side-effect visibility, violating Deterministic Behavior. |
| 11 | Mutation ↔ Visibility | Orthogonal | A `priv` method may be `proc` (mutating) or `fun` (pure) independent of its visibility; visibility restricts *who may call* an operation, mutation defines *what the operation does* once called. No composition rule is needed beyond "both apply independently." |
| 12 | Mutation ↔ Lifetime | Orthogonal | Mutating a value in place does not change its lifetime — the same binding continues to denote the same scope-bound storage before and after a `proc` call. A `new` operation, by contrast, produces a value with its own independent lifetime, but this follows directly from `new` producing a *distinct* value (Identity), not from any Lifetime-specific rule about transformation. |
| 13 | Evaluation ↔ Visibility | Orthogonal | Expression evaluation order and exhaustiveness checking apply uniformly regardless of the visibility of the functions or values involved — a `priv` function's body is exhaustiveness-checked exactly like a `pub` one. |
| 14 | Evaluation ↔ Lifetime | Orthogonal | A block expression's value (its last sub-expression) may reference bindings scoped to that block only if the value itself does not depend on those bindings outliving the block — this is Lifetime's ordinary scope-exit rule applied to the last expression like any other, not a special evaluation-order carve-out. |
| 15 | Lifetime ↔ All | Cross-cutting by construction | Every dimension operates on values, and every value has a scope-bound lifetime (Semantic Invariant 3). Lifetime is therefore not merely "one more pairwise interaction" — it is the substrate every other dimension's examples are drawn against (an owned value's scope, a borrowed reference's scope, a mutated binding's scope, an evaluated expression's scope, a visible declaration's scope). No dimension's rules are permitted to imply a lifetime that contradicts scope-based destruction. |

**Detail: Identity ↔ Ownership (pair 1).** This is the pair most likely
to be conflated because both dimensions concern "sameness" in some
sense. They must be kept separate: Identity asks *"are these two values
the same entity, structurally or referentially?"* Ownership asks *"who
is currently accountable for this value's lifecycle?"* A plain value
(no identity beyond structure) can still be moved, borrowed, and owned
exactly like a shared/reference type — Ownership's single-owner
invariant applies to *every* value, not only to those with reference
identity. Conversely, a shared/reference type's persistent identity
across mutation does not exempt it from having exactly one owner of the
reference itself at any point. The two dimensions are validated as
orthogonal because no rule in either section depends on a term defined
by the other.

**Detail: Ownership ↔ Mutation (pair 6).** Restated for emphasis because
it is the tightest coupling in the model: `proc` (Mutation) is legal
exactly when the compiler can prove exclusive access to `self`
(Ownership). This is not two rules that must be kept in sync by
convention — it is one invariant (Semantic Invariant 2) with two
dimension-level names for its two halves. Any future Implementation
Strategy that enforces Ownership (borrow checker, escape analysis, or
otherwise) automatically enforces this half of Mutation's contract as a
side effect; a Strategy cannot legally implement one without the other.

**Detail: Mutation ↔ Evaluation (pair 10).** Expression-orientation
means a `proc` call is not confined to a statement position — it can sit
inside any sub-expression. This makes the defined left-to-right
evaluation order (see [Evaluation](#evaluation)) load-bearing for
Mutation in a way it would not be in a purely eager, purely
statement-oriented language: without a fixed sub-expression order, two
semantically identical-looking expressions containing `proc` calls could
observe different intermediate states under different Implementation
Strategies, which `DESIGN_PRINCIPLES.md` § Deterministic Behavior
forbids.

**Detail: Visibility ↔ Ownership (pair 8).** The `priv`-type-in-`pub`-
signature edge case is not a defect requiring reconciliation between the
two dimensions — it is the expected, desired outcome of true
orthogonality. Visibility's scope is "can this name be written in this
location"; Ownership's scope is "who is accountable for this value once
it exists." A function's public callability and its parameter/return
types' visibility are simply different declarations, each governed by
its own dimension's rules.

**Detail: Lifetime ↔ All (pair 15).** Because every other dimension's
model section above defines its rules in terms of values that exist
within some scope, Lifetime cannot be evaluated as "compatible" or
"incompatible" with any single other dimension in isolation — it is the
common ground all five other dimensions are built on. The Design
Principles verification below treats this explicitly when checking
Orthogonality: Lifetime's universality is itself evidence that the
model's foundation (Semantic Invariant 3) is sound, not evidence of a
layering violation.

---

## Relationship to Design Principles

Each dimension is checked against the five principles most load-bearing
for a semantic model — **Explicitness**, **Orthogonality**, **Semantic
Purity**, **Minimal Core**, and **LLM Generability** (the latter via the
`LLM_GENERABILITY_GATE` criteria in `how/gates/DECISION_VALIDATION.md`,
since `DESIGN_PRINCIPLES.md` does not itself enumerate LLM Generability
as a named principle but treats it as a first-class validation gate).
Every other principle in `DESIGN_PRINCIPLES.md` (Data First, Consistency,
Uniformity, Deterministic Behavior, etc.) is satisfied by construction
where referenced inline in the dimension sections above; this table
focuses on the five principles where a dimension's design required an
explicit trade-off or verification step.

| Dimension | Explicitness | Orthogonality | Semantic Purity | Minimal Core | LLM Generability |
|---|---|---|---|---|---|
| **Identity** | Pass — reference semantics is opt-in, never implicit; value semantics is the silent default but is a *documented*, uniform default rather than a hidden rule. | Pass — orthogonal to Ownership by construction (see pair 1); no shared vocabulary between the two sections. | Pass — `==` has exactly one meaning (structural); no context-dependent overload. | Pass — one default (value semantics) plus one opt-in mechanism (explicit shared types); no third mode. | Pass — an LLM never has to infer whether a type has reference semantics from context; it is declared. |
| **Ownership** | Pass for the semantic contract (transfer must be visible); **Flag** for concrete syntax — `@ownership`/`$`/`move` are all still open, so today an LLM/human cannot yet write one canonical form. Resolved by explicit Phase 5 deferral, not by the semantic model itself. | Pass — deliberately non-prescriptive about enforcement mechanism, so it does not entangle with an Implementation Strategy choice (`IMPLEMENTATION_INDEPENDENCE_GATE`). | Pass — "ownership" names exactly one concept (accountability for a value's lifecycle), not conflated with Identity or Lifetime despite close interaction. | Pass — single-owner + move + borrow is the minimal rule set that satisfies memory-safety-without-GC; no additional ownership modes introduced. | Flag (same root cause as Explicitness) — schema-serializability of ownership transfer cannot be finalized until Phase 5 chooses a concrete syntax; the *semantic* contract is fully schema-serializable today. |
| **Mutation** | Pass — the three-way `fun`/`proc`/`new` split makes an operation's effect on `self` visible at the declaration, and no `mut` inference is required. | Pass — one clean coupling to Ownership (pair 6), explicitly identified as shared-invariant rather than incidental; no other cross-dimension entanglement. | Pass — three keywords, three non-overlapping meanings; no combination forms (no `mut fun`) that would blur the boundary. | Pass — three declaration kinds is the minimum needed to distinguish the three observably different effects (read-only, mutate-in-place, transform); collapsing to two would lose information the type checker needs. | Pass — the effect of a call is fully determined by which of three keywords its declaration uses; no ambiguity for a generator to resolve. |
| **Evaluation** | Pass — laziness requires an explicit marker; eagerness (the invisible default) is nonetheless a single, uniformly-applied rule, not a per-construct special case. | Pass — no dimension depends on evaluation timing except Mutation (pair 10), and that dependency is on evaluation *order*, not evaluation *strategy* (eager vs. lazy). | Pass — `emit` has exactly one responsibility (produce the next Sequence value); rejecting `yield` was explicitly justified by a Semantic Purity violation in the rejected alternative. | Pass — no separate statement grammar; unifying statements and expressions is a *reduction* of the core grammar, not an addition. | Pass — every construct being an expression removes an entire class of generation ambiguity (should this be a statement or produce a value?) that a statement/expression-split language forces an LLM to resolve per-construct. |
| **Visibility** | Pass — three explicit levels, no naming-convention fallback; `priv`/`pub` are always visible at the declaration site. | Pass — one identified edge case (pair 8, Ownership) is documented as expected orthogonality, not a violation. | Pass — each of the three levels (`priv`, default, `pub`) has exactly one meaning; no level is contextually reinterpreted. | Pass — three levels is the minimum needed to distinguish type-local, module-local, and exported scope; a `protected` fourth level was explicitly rejected as unnecessary given sealed types/open modules as the future alternative mechanism. | Pass — no runtime bypass means an LLM-generated program's visibility guarantees cannot be silently violated by generated code that happens to compile; violations are caught at compile time. |
| **Lifetime** | Pass — scope-bound lifetime is visible from the block/function structure itself; no lifetime is ever open-ended without a visible scope. | Pass — universality (pair 15) is itself the orthogonality argument: Lifetime constrains no other dimension's rules, it only requires that all of them respect scope-boundedness. | Pass — "lifetime" means exactly one thing (the span between creation/binding and scope-exit destruction); it is never conflated with Ownership's "who is accountable" question. | Pass — scope-based lifetime with deterministic destruction is the minimal rule; no GC, no reference counting, and no additional lifetime annotations are part of the Core (regions/arenas are Strategy-level, not Core). | Pass — deterministic, scope-derived destruction means an LLM never needs to reason about when a value is freed beyond reading the enclosing block structure it already had to generate. |

**Net Flags:** Two flags, both on the same root cause — **Ownership's
concrete transfer syntax is not yet chosen** (Phase 5 work). Both are
tracked as an explicit, intentional Phase 5 dependency rather than a
Phase 2 defect: the semantic requirement (transfer must be visible) is
fully specified; only its notation is open. This satisfies
`DECISION_VALIDATION.md`'s rule that a Flag "may proceed but the flagged
issue must be resolved before the final gate" — the final gate for
syntax choice is Phase 5's own validation cycle, not Phase 2's.

**Minimal Core check across all six dimensions.** Per
`DESIGN_PRINCIPLES.md` § Minimal Core, the Core changes "only when new
semantics cannot be expressed through composition of existing Core
primitives." All six dimensions were tested against this bar during
synthesis: no candidate seventh dimension (e.g., a separate "Aliasing"
or "Concurrency" dimension) survived the check, because aliasing is
fully explained as Ownership's borrowing rules plus Mutation's exclusive-
access requirement, and concurrency-safe mutation is a *consequence* of
Ownership + Mutation composing correctly, not an additional semantic
primitive. Six dimensions is confirmed as the minimum needed to fully
characterize what an Orthon program means.

---

## Validation

The completed Semantic Model — six dimensions, six cross-cutting
invariants, and the Cross-Dimension Consistency and Design Principles
sections above — is run as a whole through all seven gates of
[`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md)'s
Gate Catalogue. Per that document's Gate Selection table, a new Core
Language semantic ("New language construct") requires all seven gates;
this section records each gate's verdict for the model as a whole,
citing evidence already established above rather than re-deriving it.

| Gate | Verdict | Justification |
|---|---|---|
| `USER_VALUE_GATE` | Pass | The model answers a problem stated in user terms: without a fixed answer to "what does an Orthon program mean," Phase 3 (Primitive Blocks), Phase 4 (Derived Features), and Phase 5 (Syntax Design) have no stable ground to decompose or derive against — this is not a compiler-implementer-only concern, it is the precondition for every subsequent design decision a programmer-facing feature will rest on. It is directly justified by `../why/VISION.md`'s Architectural Integrity and LLM Readiness pillars, and every dimension includes a realistic code example (see each dimension's fenced example above) grounding the concept in program text a programmer actually writes. |
| `LOGICAL_CONSISTENCY_GATE` | Pass | Every term used across the six dimensions is precisely defined at first use (e.g. Binding Identity vs. Value Identity in Identity; `fun`/`proc`/`new` in Mutation) and used consistently thereafter. No self-referential paradox exists — each Semantic Invariant is checked against the dimension that specializes it, not against itself. Composition with every other dimension is documented exhaustively: the [Cross-Dimension Consistency](#cross-dimension-consistency) table above resolves all fifteen pairwise interactions, satisfying this gate's "Composition ... documented for all existing concepts" Pass criterion directly rather than by inference. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Pass | The [Relationship to Design Principles](#relationship-to-design-principles) section's Minimal Core check already tested composition explicitly: two candidate seventh dimensions (Aliasing, Concurrency) were evaluated and rejected as fully expressible through composition of the existing six, satisfying this gate's central "expressible through composition" fail condition in the negative — nothing here was retained that composition could already produce. Each dimension solves exactly one question (see each dimension's opening italic question), and no dimension introduces a new keyword beyond what its Source research already proposed. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Pass | The model occupies exactly Level 0/1 (Data Model / Primitive Operations) of the Semantic Dependency Architecture (per [EDR-013](../how/decision_records/architecture/EDR-013-semantic-model.md) § Relationship to Other Records) and never references the Standard Library or a specific Implementation Strategy — see each dimension's explicit "Implementation freedom" callouts (Identity, Ownership, Lifetime). No dimension is privileged over another: all six receive identical treatment (Model, example, Source) in the Semantic Dimensions section, and the Cross-Dimension Consistency table identifies exactly one non-trivial coupling (Ownership ↔ Mutation, pair 6), documented as a shared invariant by design rather than a special case requiring a workaround. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | Pass | Every dimension explicitly separates its semantic contract from its enforcement/implementation mechanism: Identity permits CoW/SSA/NRVO/register promotion so long as observable behavior is unchanged; Ownership deliberately does not prescribe a borrow checker vs. escape analysis vs. any other enforcement strategy; Lifetime treats regions/arenas/stack/heap as Allocation Policy choices, not semantics. No dimension's observable behavior changes by Implementation Strategy — only performance characteristics do, consistent with `DESIGN_PRINCIPLES.md` § Semantics Before Optimization, cited throughout the dimension sections above. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Pass | Both open items — Ownership's concrete transfer syntax (`@ownership` / `$` / `move`) and whether `pub` on a type implies `pub` on its members — already have a documented evolution path: explicit deferral to Phase 5's own validation cycle, not an unresolved gap with no plan (see [Relationship to Design Principles](#relationship-to-design-principles) § Net Flags). The decision is reversible in the sense that matters: no enforcement mechanism or concrete syntax is locked in yet, so Phase 5 retains full freedom among the candidate forms without needing to unwind a Phase 2 commitment. Six dimensions is a floor tested against Minimal Core, not a ceiling — a future dimension remains addable if a genuine seventh question is ever identified, without invalidating the existing six. |
| `LLM_GENERABILITY_GATE` | Flag | Same root cause as the two Flags already recorded in the [Relationship to Design Principles](#relationship-to-design-principles) table: Ownership's transfer must be visible at the transfer site (the semantic requirement, fully schema-serializable today), but the concrete marker is not yet chosen, so no canonical Orthon code example yet exists to exercise the schema round-trip criterion for that one case end-to-end. Every other dimension passes cleanly — Identity's opt-in-only reference semantics, Mutation's three-keyword declaration kinds, Evaluation's uniform expression-orientation, Visibility's three explicit levels, and Lifetime's scope-derived destruction each remove, rather than introduce, a class of generation ambiguity (see the per-dimension LLM Generability column above). This is the same flag, not a new one: it resolves when Phase 5 resolves it, per `DECISION_VALIDATION.md`'s rule that "a Flag may proceed but the flagged issue must be resolved before the final gate." |

**Overall verdict:** Six gates Pass outright; one gate
(`LLM_GENERABILITY_GATE`) Flags on the single already-tracked
Ownership-syntax open item. Per `DECISION_VALIDATION.md`'s Gate Flow
rules, a Flag "may proceed" — the model is accepted (EDR-013) with this
one flagged issue explicitly deferred to Phase 5's own validation
cycle, exactly as already recorded in this document's Net Flags note
above. No gate returned Fail; no revision is required before Phase 3
proceeds.

---

## EDR

- **[EDR-013](../how/decision_records/architecture/EDR-013-semantic-model.md):**
  Acceptance of the Orthon Semantic Model — Identity, Ownership,
  Mutation, Evaluation, Visibility, Lifetime. Records how each of the
  10 source research documents' proposals were merged, modified, or
  superseded, and the rejection of the Identity-Based Safety, GC-based,
  and statement-oriented alternatives.
