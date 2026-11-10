# Immutability: Levels and Trade-offs

> Discussion note, 2026-11-10. Related research: [Mutability](../how/concepts/research/essential/MUTABILITY.md); current design: [Semantic Model — Mutation](../what/SEMANTIC_MODEL.md#mutation). This note records analysis, not a new language decision.

Immutability reduces uncertainty about changes to existing data. It does not, by itself, specify whether a transformation allocates, copies data, shares structure, or returns an existing object. A caller must distinguish the observable effect on the original, the returned value, and the physical cost of producing it.

## Levels of Design

| Level | Mechanism | Benefit | Cost or limitation |
|---|---|---|---|
| Primitives & Types | Immutable types, mutable or read-only access, value semantics. | Some guarantees can be checked statically. | Type-system complexity; an immutable container may contain mutable values. |
| API & Language Constructs | Declarations or API contracts distinguish reading, mutation, and transformation. | Effects can be discovered and checked. | More constructs; documentation-only contracts require manual attention. |
| Coding Primitives | Assignment, copying, borrowing, ownership transfer, and local mutation. | Explicit control over access and data movement. | Programmers must understand sharing and copying. |
| Coding Patterns | Defensive copies, builders, copy-on-write, and local mutation behind an immutable interface. | Isolates mutation from consumers. | Additional code and potentially copying or lifecycle complexity. |
| Algorithmic Building Blocks | Persistent collections, structural sharing, snapshots, mutable buffers. | Previous versions can remain available without necessarily copying everything. | Memory, indirection, and operation costs depend on the data structure. |
| Algorithmic Patterns | Transformations, reductions, immutable state transitions, batched updates. | Easier reasoning, testing, and retention of intermediate states. | Intermediate values and extra passes can be expensive. |
| Problem Solving | Model histories and versions as values; model resources or counters as mutable entities when appropriate. | Aligns state management with the problem. | Immutability can be awkward or costly for frequently changing state. |

## Language Comparison

| Language | Main mechanism | What the caller must know | Trade-off |
|---|---|---|---|
| Java | Type implementation and API contract; `String` is immutable, while `final` prevents rebinding rather than object mutation. | Whether an operation mutates, returns a result, or exposes shared state; an unmodifiable view need not be deeply immutable. | Flexible APIs, but immutability often depends on documentation and defensive copies. |
| Python | Mutability depends on the type: `str` and `tuple` are immutable; `list` and `dict` are mutable. | `+=` can mutate a list but produce a new tuple; a tuple can contain mutable objects. | Convenient built-in types, with behavior and deep sharing dependent on runtime values. |
| Go | Strings are immutable; value, pointer, slice, and API semantics govern other data. | Slices can share an underlying array even when their descriptors are copied. | A small language model, with shared mutation and synchronization largely managed by programmers. |
| Rust | Immutable bindings by default, explicit mutable access, ownership, and borrowing. | Moves, borrows, `clone`, and interior mutability; cloning may share underlying data. | Static access checks, at the cost of learning and satisfying ownership rules. |
| Orthon | Proposed immutable defaults, value semantics, operation declaration kinds, and exclusive mutation access. | The declaration of a method and the cost of its operation; mutation is not marked at the call site in the semantic model. | Stronger language-level contracts, with ownership complexity and unresolved design details. |

Sources: [Java collections](https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/util/doc-files/coll-overview.html), [Python programming FAQ](https://docs.python.org/3/faq/programming.html), [Go specification](https://go.dev/ref/spec), [Rust borrowing](https://doc.rust-lang.org/stable/book/ch04-02-references-and-borrowing.html), [Rust interior mutability](https://doc.rust-lang.org/reference/interior-mutability.html), and [Rust Clone](https://doc.rust-lang.org/stable/std/clone/trait.Clone.html).

## Orthon Implications

The semantic model describes `fun` as read-only on `self`, `proc` as mutating `self`, and `new` as producing a distinct value without mutating `self`. The operation kind is declared, but no additional mutation marker appears at the call site. IDE support could expose the declaration kind; that is a tooling opportunity, not a current guarantee. A read-only receiver does not alone establish that an operation has no other effects.

Value semantics specifies observable independence, not eager physical copying. Copy-on-write and other optimizations may preserve that contract. API documentation still needs to communicate relevant allocation behavior, structural sharing, and algorithmic complexity separately from mutation semantics.

The older mutability research is a draft and describes `mut` at the call site, whereas the semantic model describes `fun`/`proc`/`new` without a call-site marker. This discrepancy requires reconciliation during concept review. Interior mutability and mutation captured by closures remain deferred in the semantic model.
