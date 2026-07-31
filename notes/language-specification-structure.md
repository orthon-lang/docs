# Language Specification Structure

> **Type:** Design methodology note.
> **Date:** 2026-07-31.
> **Context:** Discussion of what a programming language specification must
> describe — taxonomy, ontology, and the layers beyond.
> **Related:** [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md),
> [`PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md),
> [`SYNTAX.md`](../what/SYNTAX.md),
> [`EXECUTION_MODEL.md`](../what/EXECUTION_MODEL.md),
> [`CROSS_CUTTING.md`](../what/CROSS_CUTTING.md).

---

## The Question

Does a language specification describe the _taxonomy_ and _ontology_ of the
language, or something else?

**Answer:** Taxonomy and ontology are the first two layers. They are necessary
but not sufficient. A complete specification must describe six layers.

---

## The Six Layers

```
┌─────────────────────────────────────────────────────────┐
│                LANGUAGE SPECIFICATION                     │
├─────────────────────┬───────────────────────────────────┤
│ 1. ONTOLOGY         │ What entities exist                │
│    (entities)       │ Values, types, scopes, contexts,   │
│                     │ functions, modules                 │
│                     │ Ownership, lifetime, identity      │
├─────────────────────┼───────────────────────────────────┤
│ 2. TAXONOMY         │ How entities are classified        │
│    (categories)     │ Type kinds (primitive, composite,  │
│                     │   algebraic, resource)             │
│                     │ Statement kinds (binding, control  │
│                     │   flow, invocation)                │
│                     │ Context kinds (delegate, defer,    │
│                     │   spawn, fork)                     │
│                     │ Operator kinds (single-owner,      │
│                     │   distribution)                    │
├─────────────────────┼───────────────────────────────────┤
│ 3. SYNTAX           │ Concrete textual form              │
│    (surface)        │ Grammar, precedence, keywords,     │
│                     │   delimiters, literals             │
│                     │ Lexical structure (tokens,         │
│                     │   identifiers, comments)           │
├─────────────────────┼───────────────────────────────────┤
│ 4. STATIC            │ What is valid before execution    │
│    SEMANTICS         │ Type-checking rules (typing       │
│    (well-formedness) │   judgments)                      │
│                      │ Name resolution (scope, binding)  │
│                      │ Ownership/borrow checking         │
│                      │ Send/Move trait resolution        │
│                      │ Constant evaluation               │
├─────────────────────┼───────────────────────────────────┤
│ 5. DYNAMIC           │ What happens at runtime           │
│    SEMANTICS         │ Evaluation rules (small-step or   │
│    (meaning)         │   big-step operational semantics) │
│                      │ Effect system (what effects a     │
│                      │   computation may have)           │
│                      │ Context lifecycle (construction,  │
│                      │   submission, materialisation,    │
│                      │   destruction)                    │
│                      │ Concurrency model (ordering,      │
│                      │   happens-before, atomicity)      │
├─────────────────────┼───────────────────────────────────┤
│ 6. INVARIANTS         │ What must always hold            │
│    (guarantees)       │ Memory safety (no use-after-free,│
│                       │   no double-free)                │
│                       │ Type safety (no type confusion   │
│                       │   at runtime)                    │
│                       │ Ownership invariants (exactly    │
│                       │   one owner, borrows outlive     │
│                       │   referent)                      │
│                       │ Deterministic destruction        │
│                       │   (resource cleanup at scope     │
│                       │   exit)                          │
└─────────────────────┴───────────────────────────────────┘
```

---

## What Each Layer Answers

| Layer | Question | Wrong answer looks like |
|-------|----------|------------------------|
| Ontology | "What exists?" | A spec that talks about 'functions' without defining what a function _is_ — just assumes the reader knows. |
| Taxonomy | "What kinds are there?" | A spec that lists features without grouping them — a flat catalog, not a structure. |
| Syntax | "What does the programmer write?" | A spec that describes semantics without ever showing what the code looks like. |
| Static semantics | "What is rejected before it runs?" | A spec that says "the compiler checks types" but never defines the checking rules. |
| Dynamic semantics | "What happens when it runs?" | A spec that says "a function call evaluates its body" but never defines evaluation order, argument passing, or return. |
| Invariants | "What can never happen?" | A spec that guarantees "memory safety" without defining what safety _means_ in this language. |

---

## Why Taxonomy + Ontology Alone Is Not Enough

A specification that stops at layers 1–2 is an **encyclopedia**, not a
specification. It tells you what exists and how it's classified, but not:

- What any of it _means_ when the program runs (layer 5).
- What combinations are _valid_ (layer 4).
- What the programmer actually _writes_ (layer 3).
- What the language _guarantees_ (layer 6).

**Example:** A taxonomy entry for `delegate` says "delegate is a context
constructor that creates a mailbox context with single-owner semantics." That
answers _what_ and _what kind_. But it does not answer:

- What is the syntax? (`let ctx = delegate(obj)`) — layer 3.
- Is `delegate(nil)` rejected at compile time? — layer 4.
- What happens when two messages arrive simultaneously? — layer 5.
- Is the owner guaranteed to be destroyed exactly once? — layer 6.

All six layers together make a specification.

---

## How Orthon's Current Spec Maps to the Six Layers

| Layer | Orthon document | Status |
|-------|----------------|--------|
| 1. Ontology | [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) — identity, ownership, mutation, evaluation, visibility, lifetime | ✅ Accepted (Phase 2) |
| 2. Taxonomy | [`PRIMITIVE_BLOCKS.md`](../what/PRIMITIVE_BLOCKS.md) — blocks categorized by dimension; context taxonomy emerging in [`EXECUTION_CONTEXT_INVOCATION.md`](../how/concepts/research/essential/EXECUTION_CONTEXT_INVOCATION.md) | ⚠️ Primitive blocks accepted; context taxonomy in research |
| 3. Syntax | [`SYNTAX.md`](../what/SYNTAX.md) — grammar, precedence, keywords | ⚠️ Exists; needs update for two-operator family, `using` sugar, generator syntax (Phase 5) |
| 4. Static semantics | [`TYPE_SYSTEM.md`](../how/architecture/TYPE_SYSTEM.md) in `how/architecture/` is implementation architecture, not language spec; `Send`/`Move` traits defined in research but not yet formalized | ❌ No formal typing judgments yet |
| 5. Dynamic semantics | [`EXECUTION_MODEL.md`](../what/EXECUTION_MODEL.md) — execution semantics; needs update for context lifecycle | ⚠️ Exists; substantial gaps (context lifecycle, generator protocol) |
| 6. Invariants | Partially in [`SEMANTIC_MODEL.md`](../what/SEMANTIC_MODEL.md) (scope-based destruction, single-owner) and [`LIBRARY_BOUNDARY.md`](../what/LIBRARY_BOUNDARY.md) | ⚠️ Scattered; no unified invariants document |

---

## What Orthon Still Needs for a Complete v0.1 Spec

### Static Semantics (Layer 4)

A formal specification of what the compiler rejects before execution:

- **Typing judgments** — e.g., `Γ ⊢ e : τ` (in context Γ, expression e has type τ).
- **Well-formedness rules** — when is a program syntactically valid but
  semantically ill-formed?
- **Send/Move resolution** — formal rules for when captured data satisfies
  `Send`/`Move` (currently prose in OQ5 Resolution).
- **Name resolution** — how identifiers are bound to declarations, shadowing
  rules.

### Dynamic Semantics (Layer 5)

A formal specification of what happens at runtime:

- **Evaluation rules** — small-step (`e → e'`) or big-step (`e ⇓ v`).
- **Context lifecycle** — construction → submission → materialisation →
  destruction as formal state transitions.
- **Generator protocol** — `next()`/`stop()` semantics on `spawn`/`fork`
  contexts.
- **Effect system** — what effects a computation may have (currently
  implicit).

### Invariants (Layer 6)

A unified document or section that collects all guarantees:

- Memory safety invariants.
- Type safety invariants.
- Ownership invariants.
- Lifetime invariants (deterministic destruction).
- Concurrency invariants (data-race freedom conditions).

---

## The Relationship Between Layers

```
Ontology                    ← "A context is an entity that owns a resource
    ↑                         and executes submitted invocations."
Taxonomy                    ← "Context kinds: delegate, defer, spawn, fork.
    ↑                         Operator kinds: single-owner, distribution."
Static Semantics            ← "spawn() requires captured data to satisfy Send."
    ↑
Dynamic Semantics           ← "next() blocks until a result is available or
    ↑                         the context is depleted; stop() cancels pending."
Invariants                  ← "A context's destructor runs exactly once, at
                              scope exit, regardless of how the scope is
                              exited."
```

Each layer constrains the one above it. The invariants (layer 6) are the
deepest — they constrain everything above. The ontology (layer 1) is the
broadest — everything below instantiates it.

---

## Key Insight

> **A language specification is not a catalog of features.** It is a
> layered description where each layer answers a specific kind of question,
> and the layers compose to form a complete, self-consistent definition of
> what programs in the language mean.

Taxonomy and ontology are the foundation. But without semantics (what it
means), syntax (what you write), and invariants (what's guaranteed), you
have a dictionary, not a specification.
