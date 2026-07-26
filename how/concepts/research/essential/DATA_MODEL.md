# Orthon Data Model

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 0 (Foundation) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

Every programming language needs a data model — a formal description of
what kinds of values exist, how they relate to each other, and how they
are represented. The core problem: **how does Orthon define its type
hierarchy and representation system** in a way that is orthogonal,
consistent, and LLM-generable?

Languages with ad-hoc data models (Java's primitive vs. object divide,
Python's `__magic__` method protocol) create fragmented mental models.
A well-designed data model should be learnable once and applicable
everywhere — a small set of primitive types, a single composition
mechanism, and a transparent representation system.

## Principles

1. **Data First** — All computation operates on Data. Data has no
   imposed semantic meaning until a Data Modifier transforms it.
2. **Orthogonality** — Data representations combine freely. Any
   representation can contain any other.
3. **Simplicity** — Few primitive types. Composition via tuples,
   sequences, and sets — no special-purpose type forms.
4. **Value semantics by default** — Data is compared by its structure,
   not its identity. Copying data produces an independent value.
5. **Transparent representation** — The representation of data (Value,
   Tuple, Reference, Sequence, Set, Option, Result) is part of the type
   system and visible in the type signature.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Representation Policy | Defines how data representations are mapped to runtime memory layouts |
| Allocation Policy | Determines where data values are stored (stack, heap, arena) |
| Equality Policy | Controls how data values are compared (structural vs. identity) |

## Model (What)

Orthon's data model is built from a small set of primitives and
representations:

**Primitive types:** `Int`, `Float`, `Bool`, `Char`, `String`, `Byte`,
`Symbol`. These are the atomic building blocks from which all data is
constructed.

**Representations** (Data Modifiers that create views of data):
- `Value` — A single, self-contained value
- `Tuple` — An ordered, fixed-size collection of values
- `Reference` — A pointer to a value elsewhere in memory
- `Sequence` — An ordered, variable-size collection (lazy or eager)
- `Set` — An unordered collection with uniqueness
- `Option` — A value that may be absent (Some/None)
- `Result` — A value that represents success or failure (Ok/Err)

```orthon
42                    # Value representation
(1, "hello", 3.14)   # Tuple representation
&data                 # Reference representation
[1, 2, 3]             # Sequence representation (eager)
{1, 2, 3}             # Set representation
some(42)              # Option representation
ok(42)                # Result representation
```

## Default Strategy

All data uses value semantics by default: assignments copy data
structurally, equality is structural, and representations are
materialized eagerly. The default memory layout for tuples and
sequences is contiguous storage.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Reference semantics | Data is shared by reference; copies are shallow | Large immutable data structures; shared state |
| Lazy materialization | Sequences and compound data are computed lazily | Streaming data; infinite sequences |
| Packed representation | Data is compacted for memory efficiency | Embedded or memory-constrained targets |

## Open Questions

1. Should Orthon support sum types (tagged unions/enums) as a first-class
   representation, or should `Option` and `Result` cover the common cases?
2. How do user-defined types (records/structs) integrate into the data
   model? Are they syntactic sugar over tuples?
3. Should there be a `Symbol` type (interned string, like Lisp/Elixir/Ruby)?
4. How does the data model interact with the FFI — what representation do
   C-compatible types use?

## Decision History

- **Value semantics by default:** Adopted. Data is compared structurally
  and copied eagerly. Alternatives considered: reference semantics by default
  (rejected: violates Data First — data should be self-contained), hybrid
  (rejected: violates Consistency).
- **Seven representations:** Adopted as the complete set of Data Modifier
  views. Alternatives considered: fewer representations with runtime tagging
  (rejected: violates Orthogonality), more representations (rejected:
  violates Simplicity).
- **Deferred to Phase 3:** Sum type design, user-defined type integration,
  FFI representation mapping, Symbol type specification.
