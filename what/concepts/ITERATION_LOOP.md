# Iteration Loop — `for`/`while` with Protocol-Based Iteration

> **✅ ACCEPTED — [EDR-053](../how/decision_records/architecture/EDR-053-iteration-loop.md);
> amended by [EDR-094](../how/decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md)
> (`for`-expression → `Optional<T>`, `for`-`else` rejected, iteration ownership).**
>
> **Status:** Accepted 2026-07-27; amended 2026-09-29 (EDR-094).
>
> **See also:** [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`LAZY_SEQUENCE_GENERATORS.md`](LAZY_SEQUENCE_GENERATORS.md),
> [`NULL_SAFETY.md`](NULL_SAFETY.md) (Optional type, EDR-018),
> [`GLOSSARY.md`](../GLOSSARY.md) § For Loop, While Loop

---

## Issue (Why)

How does a language express repeated execution over a sequence of values?

Two fundamentally different loop models exist:

1. **For-each iteration** (`for item in sequence`) — Consume values from an iterable source. The programmer describes *what* to iterate over; the language handles *how*.
2. **Condition-based looping** (`while condition`) — Repeat until a condition is false. Used when iteration is not sequence-driven.

The core problem: **should the language include a general-purpose loop (C-style `for (;;)`) alongside the safer for-each, or should it commit to for-each-only and provide range-based index iteration?**

Orthon's answer: one iteration construct (`for ... in`), one condition-based construct (`while`), and an infinite loop (`loop`). No C-style `for (;;)`.

## Principles

1. **One iteration construct** — `for item in sequence` is the only loop for consuming values from a sequence. No separate syntax for index-based vs. element-based iteration.

2. **Sequence-based** — The loop operates on sequences (values produced over time), not on indices. Index-based iteration uses range syntax: `for i in 1..len(seq)` (inclusive-inclusive norm, EDR-083).

3. **Protocol-based** — `for` desugars to the iterator protocol (`IntoIterator` + `next()`).

4. **`while` for conditions** — `while condition` is a separate construct for non-sequence looping. Clear semantic distinction from `for`.

5. **`loop` for infinite loops** — Explicit infinite loop with optional `break value` for expression-oriented patterns.

6. **No hidden allocation** — Iteration streams values without allocating intermediate buffers.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Iteration Policy | Defines which types are iterable (all `IntoIterator<T>` implementors) |
| Loop Model Policy | Determines that `for ... in` is the only iteration construct |
| Desugaring Policy | Formalises `for` loop desugaring to iterator protocol |
| Early Termination Policy | Controls `break` and `continue` semantics |

## Model (What)

### `for` Loop — Sequence Iteration

```orthon
# Basic iteration
for item in items:
    process(item)

# Index-based via range (inclusive-inclusive 1..N)
for i in 1..len(array):
    process(array[i])

# Destructuring in loop variable
for (index, value) in items.enumerate():
    print("Index {index}: {value}")

for {name, age} in people:
    print("{name} is {age} years old")
```

The `for` loop accepts any type that implements `IntoIterator<T>`:

```orthon
# Desugaring (per EDR-022)
for item in collection:
    process(item)

# → let mut it = collection@iter()
#   loop:
#       match it@next():
#           Some(item) -> process(item)
#           None       -> break
```

### `while` Loop — Condition-Based

```orthon
while queue.not_empty():
    process(queue.dequeue())

while i < limit:
    process(i)
    i = i + 1
```

### `loop` — Infinite Loop

```orthon
let result = loop:
    let item = queue.receive()
    if is_valid(item) then break item

# Without break value:
loop:
    process.next_event()
```

### `break` and `continue`

```orthon
# break — exit the loop
for item in items:
    if is_target(item):
        handle(item)
        break

# continue — skip to next iteration
for item in items:
    if skip(item):
        continue
    process(item)

# break with value (loop only)
let first_valid = loop:
    let item = stream.receive()
    if item.is_valid():
        break item
```

### Range Syntax

Range literals are first-class values defined by the RANGE concept ([`RANGE.md`](RANGE.md), EDR-083). They are inclusive-inclusive `1..N` and implement `IntoIterator`, so they drive `for` directly:

```orthon
for i in 1..10:            # 1, 2, ..., 10 (inclusive-inclusive)
for i in 1..len(array):    # index-based iteration over all elements
for i in (1..10).step(2):  # 1, 3, 5, 7, 9 — step via a method on Range
```

### `for` as an Expression — Optional Search

In **value position**, `for` is an expression that yields `Optional<T>` (the option
type of NULL_SAFETY, EDR-018):

- `break v` / `return v` yields `Some(v)`;
- running off the end without a break yields `None`;
- a bare `break` (no value) in expression position yields `None` early.

The loop's type in value position is `Optional<T>`. In **statement position**, `for`
yields no value. The simple single-predicate case stays the combinator
`coll.find(|x| ...)` (`Option<T>`); the `for`-expression is reserved for
**multi-statement bodies with early exit**, where a predicate closure is awkward.
Its scope is Optional search only — general accumulation (`fold` / `reduce`) stays a
combinator.

```orthon
# for-expression: multi-statement search, first match or None
let found = for row in table:
    let parsed = parse(row)
    if parsed.is_valid() and parsed.key == target:
        break parsed          # -> Some(parsed)
# run off the end -> None ; found : Optional<Row>

match found:
    Some(row) -> use(row)
    None      -> not_found()
```

### Iteration and Ownership

Iteration is **borrow-by-default**: bare `for x in coll` is a shared read borrow, and
the collection survives the loop. Modification and consumption are explicit, expressed
by ownership markers **on the source** (inherited from the language-wide vocabulary —
not loop-specific methods):

- `for x in coll` — shared read borrow; element type `&T`.
- `for x in &mut coll` — exclusive write-borrow; element type `&mut T`; requires `mut coll`.
- `for x in $coll` — move / consume; element type `T`; consumes the binding (rebind after).

**Structural mutation is not a `for`-mode.** A live iterator borrow freezes the
composition (iterator-invalidation safety); changing length or membership
(`coll.push` / `coll.remove`) happens *outside* the loop and requires `mut coll`.
**Interior mutability** (immutable collection with mutable elements) is **not
introduced in v0.1**, keeping a clean transitive model (`let` deeply immutable,
`let mut` a mutable path).

Authority frame: a **binding** is authority held over a storage slot (`mut` = write,
non-`mut` = read-only); a **grant** is a permanent transfer of that authority
(`$` / move / reassign); a **rent** is a temporary loan (a reference / borrow), split
by the aliasing invariant into a **read-rent** (`&T`, many concurrent) XOR a
**write-rent** (`&mut T`, exactly one). A rent ends automatically at loop scope; a
grant never returns and leaves the source binding dead.

| Mode (loop side) | Borrow | Composition | Element values | `x` type | Needs `mut coll` | Collection after |
|---|---|---|---|---|---|---|
| `for x in coll` | shared | frozen | immutable | `&T` | no | same, unchanged |
| `for x in &mut coll` | exclusive | frozen | **mutable** | `&mut T` | **yes** | same, elements changed |
| `for x in $coll` | move | dismantled | handed out by value | `T` | — (consumes) | **gone**, rebind required |
| `coll.push/remove(...)` *(outside loop)* | exclusive, no iterator | **mutable** | — | — | **yes** | composition changed |

The concrete spelling of `&` / `&mut` / `$` is owned by Reference, OWNERSHIP, and
MUTABILITY; ITERATION_LOOP inherits — does not define — these markers (see EDR-094,
boundary flags B1–B4).

## Default Strategy

All iteration is lazy and single-pass. `for` desugaring is a syntactic transformation applied by the compiler. Range types are StdLib types implementing `IntoIterator<T>`.

## Alternative Strategies

| Strategy | Description |
|---|---|
| C-style for included | Both `for (;;)` and `for ... in` available. Adds complexity with no semantic benefit. |
| While as only loop | No `for` construct — all iteration uses `while` with explicit iterator management. Verbose. |

## Open Questions

1. ~~Should `range` type (`0..10`) be a built-in literal or a StdLib constructor?~~ **Resolved 2026-08-05 (EDR-083):** the `a..b` literal is Language; the `Range` type and `range(a, b)` named form are StdLib; both are equivalent.
2. ~~Should `break` support a value in `for` loops (like Rust's `break` in `loop`)?~~ **Resolved 2026-09-29 (EDR-094):** Yes — `for` becomes an expression. In value position `break v` / `return v` → `Some(v)`, running off the end → `None`, loop type `Optional<T>`.
3. ~~Should there be an `else` clause on `for` (Python-style, executed if no `break`)?~~ **Resolved 2026-09-29 (EDR-094):** No — `for`-`else` rejected (overloads `else`, breaking one-symbol-one-meaning); the pattern is covered by the Optional `for`-expression + `match`.
4. ~~How does iteration interact with ownership — does `for item in collection` consume or borrow?~~ **Resolved 2026-09-29 (EDR-094):** Borrow by default (`for x in coll` shared `&T`); explicit `for x in &mut coll` for write-borrow (`&mut T`, requires `mut coll`) and `for x in $coll` for move / consume (`T`, rebind after).

## Decision History

- **2026-07-27** — Accepted via EDR-053. Classification: Language. `for`/`while`/`loop` constructs with protocol-based desugaring. One iteration construct (`for ... in`). No C-style `for (;;)`.
- **2026-08-05** — Range syntax delegated to EDR-083. Range literals follow the inclusive-inclusive `1..N` norm; `for i in 1..len(array)` is the canonical index iteration.
- **2026-09-29** — Amended by EDR-094. Open Questions 2/3/4 resolved: `for` becomes an expression returning `Optional<T>` in value position (`break v`/`return v` → `Some(v)`, run-off-end → `None`); `for`-`else` rejected (covered by Optional + `match`); iteration is borrow-by-default with source-side ownership markers (`coll` shared `&T`; `&mut coll` write-borrow `&mut T`; `$coll` move `T`), structural mutation is not a `for`-mode, and interior mutability is not introduced in v0.1.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [x] `what/concepts/ITERATOR_PROTOCOL.md` (referenced)
- [x] `what/concepts/RANGE.md` (referenced)
- [x] `what/concepts/ITERATION_LOOP.md` (this document — EDR-094: `for`-expression, ownership matrix, OQ2/OQ3/OQ4 resolved)
- [ ] `what/SYNTAX.md` — **DEFERRED to Phase 5 (Syntax Design)**: a `what/syntax/` record for `for`/`while`/`loop` (currently absent) will carry the concrete surface syntax, referencing EDR-094. Not created in this task.
- [ ] `what/EXECUTION_MODEL.md` — **DEFERRED to Phase 7 (Execution & Optimization Model)**: lazy/single-pass and no-hidden-allocation guarantees, and iteration × Execution Contexts. Not created in this task.
