# Class or Structure as Primary Composition Unit

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

[`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md) already covers the general
philosophy of why composition beats class hierarchy — the "is-a" vs. "has-a" question, and why
inheritance's fragile-base-class and diamond problems make deep hierarchies unattractive. This
document has a narrower scope: given that composition is already the chosen default, **which
specific combination of tools should a programmer reach for to compose a typical type, and when
is reaching for all of them overkill?**

Python offers three independent, non-overlapping tools for composing a type without inheritance:

1. `@dataclass` — data plus boilerplate reduction (see [`DATACLASSES.md`](DATACLASSES.md)).
2. Mixin classes with no cooperative `super()` chains — behaviour reuse across otherwise-unrelated
   types (see [`MIXIN.md`](MIXIN.md)).
3. `typing.Protocol` — structural, static-only contract checking (see
   [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md)).

No guidance currently exists on how to combine these three tools as a default recipe, or on when
a subset of them suffices. Without such guidance, programmers either over-reach for deep
inheritance — the anti-pattern `COMPOSITION_OVER_INHERITANCE.md` already rejects — or under-use the
tools independently without realizing that they compose orthogonally, one per concern.

The core problem this document answers: **should Orthon's default type/trait model bless a
specific "gentleman's set" combination of these three orthogonal tools as an idiomatic composition
pattern, and how does the language make clear when only a subset of that set is needed?**

## Principles

1. **Orthogonality** (`../DESIGN_PRINCIPLES.md` § Orthogonality) — each of the three tools solves
   exactly one problem (data layout, behaviour reuse, contract checking); none of them should be
   bundled into a single mandatory mechanism.
2. **Composition over inheritance** (`COMPOSITION_OVER_INHERITANCE.md` Principle 1) — none of the
   three tools should require or encourage deep hierarchy; each is a flat, independent addition to
   a type.
3. **Single Responsibility** (`../DESIGN_PRINCIPLES.md` § SOLID) — a data aggregate's only
   responsibility is holding data; a behaviour mixin's only responsibility is one cross-cutting
   concern; an interface's only responsibility is declaring a contract.
4. **Explicitness** (`../DESIGN_PRINCIPLES.md` § Explicitness, and `MIXIN.md` Principle 2) —
   mixins must not rely on cooperative `super()`/MRO traversal; conflicts between composed pieces
   must be visible at the composition site, not resolved by a hidden linearization algorithm.
5. **Minimal Core** (`../DESIGN_PRINCIPLES.md` § Minimal Core) — this pattern must not require a
   new language construct; it must be achievable purely by composing Orthon's existing derive
   mechanism ([`DATACLASSES.md`](DATACLASSES.md)), trait/mixin mechanism
   ([`MIXIN.md`](MIXIN.md), [`TRAITS.md`](TRAITS.md)), and structural trait satisfaction
   ([`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md)).

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Derivation Policy | Determines which structural methods (`init`, `eq`, `repr`, `hash`, ...) the data aggregate auto-generates — see [`DATACLASSES.md`](DATACLASSES.md). |
| Trait Resolution Policy | Determines whether a method supplied by a mixin-role trait counts toward satisfying a separate contract-role trait — see [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md). |
| Type Compatibility Policy | Determines nominal vs. structural satisfaction for the interface/contract layer of the pattern — see [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md) and [`TRAITS.md`](TRAITS.md). |
| Mutability Policy | Determines whether the data aggregate's fields are frozen (immutable) or mutable by default. |

*The specific list is filled in during concept design and passes verification through
`IMPLEMENTATION_INDEPENDENCE_GATE`.*

## Model (What)

### The Three Tools, Not Always All Three

The pattern treats composition as a choice of up to three orthogonal tools, each reached for only
when its specific trigger condition is met:

1. **A data-only aggregate** (dataclass-equivalent derive, per [`DATACLASSES.md`](DATACLASSES.md))
   is the default unit whenever a type is just data — it is reached for unconditionally, since
   nearly every type has some data shape.
2. **A stateless behaviour mixin** (no fields, no cooperative `super()` chains, per `MIXIN.md`'s
   rejection of implicit linearization) is added only when the *same* cross-cutting,
   magic-method-style logic repeats across two or more otherwise-unrelated data aggregates. It is
   not added speculatively to a single type "just in case."
3. **A structural/protocol-style interface** (per `STRUCTURAL_TYPING.md`'s structural trait
   satisfaction) is added only when a caller needs a statically-checkable contract without forcing
   the data aggregate to declare, inherit from, or otherwise couple itself to that contract.

None of the three tools is aware of the other two. A type may use just one, any two, or all
three — the combination is chosen per-problem, not mandated as a package deal.

### Worked Example (Python)

```python
from dataclasses import dataclass
from typing import Protocol


class Addable(Protocol):
    def __add__(self, other: "Addable") -> "Addable": ...


class AddMixin:
    """Stateless mixin: no __init__, no fields, no super() chain."""

    def _fields_tuple(self) -> tuple:
        raise NotImplementedError("composing type must provide _fields_tuple()")

    def __add__(self, other):
        a = self._fields_tuple()
        b = other._fields_tuple()
        return self.__class__(*(x + y for x, y in zip(a, b)))


@dataclass
class Vector(AddMixin):
    x: float
    y: float

    def _fields_tuple(self) -> tuple:
        return (self.x, self.y)


def sum_addables(items: list[Addable]) -> Addable:
    total = items[0]
    for item in items[1:]:
        total = total + item
    return total
```

`Vector` never declares `Addable` anywhere in its definition. The `Protocol` is satisfied
structurally and is checked only by a static type checker — it imposes zero runtime cost and zero
coupling on `Vector`. `AddMixin` supplies the one cross-cutting behaviour (`__add__`), stateless
and reusable across any future aggregate that provides `_fields_tuple()`. The `@dataclass`
supplies everything data-shaped: fields, `__init__`, `__repr__`, `__eq__`. Each of the three tools
does exactly one job, and none of them is aware of the others' existence.

### Translating to Orthon

Mapping each Python tool onto Orthon's corresponding in-repo research:

- The **data aggregate** maps to a `derive`-annotated type (see [`DATACLASSES.md`](DATACLASSES.md)).
- The **interface/contract** maps to a trait declaration used purely for its method signatures,
  satisfied per whichever satisfaction policy Orthon ultimately settles on (see
  [`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md)).
- The **cross-cutting mixin** maps to a trait carrying default method implementations — see
  `TRAITS.md`'s "Default Implementations" subsection, and `MIXIN.md`'s proposal that mixins are
  traits with concrete behaviour rather than a separate language construct.

This mapping surfaces a genuine open tension in the existing research corpus, not a resolved
fact: [`TRAITS.md`](TRAITS.md)'s own Principles state that traits carry **no data fields** and
require **explicit, nominal `impl`** (no structural satisfaction), while
[`STRUCTURAL_TYPING.md`](STRUCTURAL_TYPING.md) proposes **structural (implicit) satisfaction** as
the default, and [`MIXIN.md`](MIXIN.md) proposes that mixins-as-traits **may carry fields**. This
document's three-tool synthesis assumes the `MIXIN.md`/`STRUCTURAL_TYPING.md` view prevails over
`TRAITS.md`'s stricter model. That conflict between `TRAITS.md` and
`MIXIN.md`/`STRUCTURAL_TYPING.md` is unresolved and must be settled during Concept Design Review
before this pattern can be accepted as-is.

Separately: because Orthon's structural trait satisfaction (per `STRUCTURAL_TYPING.md`) is checked
by the compiler unconditionally, rather than by an optional external tool the way `typing.Protocol`
is checked only when a developer opts into running `mypy`, one of the Python "when NOT to use
this" caveats (see Alternative Strategies below) does not carry over to Orthon by construction.

## Default Strategy

By default, a composed type in Orthon is modeled as a `derive`-annotated data aggregate for state,
implementing zero or more traits for contracts. Behaviour-carrying (mixin-role) traits are opt-in
and are added only once duplicate trait-default logic is identified across two or more
otherwise-unrelated aggregate types — never applied speculatively to a single type.

## Alternative Strategies

Permitted reduced tool combinations, in decreasing order of ceremony:

| Combination | When to use |
|---|---|
| Data aggregate alone (no trait, no mixin) | Simple config/DTO/value types with no behaviour beyond structural equality/repr. |
| Data aggregate + one or more traits, no mixin | Each trait's implementation is unique to that one type, so a default-impl-bearing (mixin-role) trait would be unnecessary abstraction. |
| Data aggregate + a mixin-role trait, no separate contract-only trait | The shared behaviour never needs to be a generic constraint elsewhere — no caller needs to write `where T: Trait`. |
| Full three-tool composition | A cross-cutting behaviour *and* a checkable generic constraint are both needed at once (as in the Worked Example above). |

Three "when NOT to use this pattern at all" caveats from the source Python idiom, translated into
Orthon terms:

1. **Single, simple, one-off structures with no repeated behaviour** — the full three-tool
   ceremony is over-engineering for a type used in exactly one place; a bare `derive`-annotated
   type suffices.
2. **No static analysis discipline in the surrounding tooling** — in Python, this caveat exists
   because `typing.Protocol` is unenforced unless a separate type checker (e.g. `mypy`) is run.
   This does **not** apply to Orthon by construction: trait satisfaction under `STRUCTURAL_TYPING.md`
   is compiler-enforced unconditionally, not opt-in tooling, so there is no equivalent "nobody runs
   the checker" failure mode (see the "Translating to Orthon" note above).
3. **Performance-critical code needing fixed, `__slots__`-equivalent memory layout** — this caveat
   *does* carry over to Orthon: see [`SLOTS.md`](SLOTS.md) for Orthon's corresponding memory-layout
   mechanism. Adding a mixin-role trait with fields could, depending on how slots and trait fields
   interact, be at odds with a slot-like fixed layout. This is a genuine, still-applicable caveat
   in Orthon, not eliminated by construction.

## Open Questions

1. The `TRAITS.md`-vs-`MIXIN.md`/`STRUCTURAL_TYPING.md` tension on whether traits carry fields and
   whether satisfaction is nominal or structural, exactly as flagged in "Translating to Orthon"
   above — this must be resolved before this pattern can be formally accepted.
2. Does Orthon need to formally distinguish "mixin-role traits" (default-impl-bearing) from
   "contract-role traits" (signature-only), or is this purely a usage convention layered over one
   unified trait mechanism?
3. Can a `derive`-annotated aggregate satisfy a trait using a mixin-provided method, or only
   through its own directly-declared methods?
4. How does this pattern interact with [`SLOTS.md`](SLOTS.md)'s fixed memory layout — does adding a
   mixin-role trait ever force an aggregate out of a slot-like layout?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `DATA_MODEL.md`
- [ ] `../DESIGN_PRINCIPLES.md`
- [ ] `GLOSSARY.md`
- [ ] `IMPLEMENTATION_STRATEGIES.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
- [ ] Other: `DATACLASSES.md`, `MIXIN.md`, `COMPOSITION_OVER_INHERITANCE.md`, `STRUCTURAL_TYPING.md`, `TRAITS.md`, `SLOTS.md`
