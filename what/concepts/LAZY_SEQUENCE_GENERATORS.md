# Lazy Sequence Generators

> **✅ ACCEPTED — [EDR-021](../how/decision_records/architecture/EDR-021-lazy-sequence-generators.md).**
>
> **Status:** Accepted 2026-07-27.
>
> **See also:** [`ITERATOR_PROTOCOL.md`](ITERATOR_PROTOCOL.md),
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Evaluation,
> [`GLOSSARY.md`](../GLOSSARY.md) § Lazy Sequence, Generator,
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

Manually implementing an iterator requires writing a stateful class or object, explicitly defining iteration methods — too much boilerplate for a simple idea. This forces programmers to write infrastructure code instead of business logic.

The core problem: **producing a sequence of values lazily should be as simple as writing a function that emits values one at a time**, without managing iterator state, tracking position, or implementing protocol methods manually.

Orthon's solution: **generators — functions that use `emit` to produce lazy sequences — as a first-class language feature**. Generators return iterators automatically, eliminating manual iterator implementation.

## Principles

1. **Lazy by default** — Generator bodies execute lazily, producing values on demand. No eager materialisation. Materialisation is explicit via `.collect()`. (Per Phase 3 D-06.)
2. **`emit` as the canonical form** — The `emit` keyword is the primary way to produce values from a generator. It is syntactically visible and semantically unambiguous.
3. **All canonical forms equivalent** — `emit value`, `return sequence(value)`, and `return value ->` are equivalent forms of the same operation.
4. **Composition without allocation** — Generator expressions compose via combinators without intermediate allocation of collections.
5. **Infinite sequences valid** — The language does not artificially prevent infinite sequences. The programmer controls termination.
6. **Generators produce iterators** — A generator function returns a value that implements `Iterator[T]`, making generators composable with the iterator protocol and all its combinators.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Evaluation Policy | Defines lazy evaluation semantics — generator body does not execute eagerly |
| Allocation Policy | Affects whether intermediate results are allocated (combinators are lazy, no allocation) |
| Iterator Protocol Policy | Generators implement `Iterator[T]` — the production side of the consumption/production pair |
| Desugaring Policy | Formalises `emit` keyword desugaring to iterator protocol `next()` calls |

## Model (What)

A **generator** is a function that produces a sequence of values lazily, one at a time, using the `emit` keyword. The generator body is compiled into a state machine that implements `Iterator[T]`.

### All Canonical Forms

```orthon
# Form 1: emit keyword (canonical)
fun natural_numbers() -> Iterator[Int]
    let i = 0
    loop:
        emit i
        i = i + 1

# Form 2: return sequence(value)
fun natural_numbers() -> Iterator[Int]
    let i = 0
    loop:
        return sequence(i)
        i = i + 1

# Form 3: return value ->
fun natural_numbers() -> Iterator[Int]
    let i = 0
    loop:
        return i ->
        i = i + 1
```

All three forms are semantically equivalent. The `emit` form is the canonical and preferred form for its clarity.

### Generator Semantics

```orthon
# Generator that produces a finite sequence
fun fib(limit: Int) -> Iterator[Int]
    let (a, b) = (0, 1)
    while a <= limit:
        emit a
        (a, b) = (b, a + b)

# Consumer
for n in fib(100):
    print(n)        # 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89

# Equivalent using method call
let sequence = fib(100)
sequence.for_each(|n| print(n))
```

### Generator Composition

Generators compose naturally with iterator combinators because a generator returns an `Iterator[T]`:

```orthon
# Composition without intermediate allocation
let result = natural_numbers()
    .filter(|n| n % 2 == 0)       # keep even numbers
    .map(|n| n * n)               # square them
    .take(10)                      # first 10
    .collect()                     # materialise

# Generators are composable with each other
fun interleave[T](a: Iterator[T], b: Iterator[T]) -> Iterator[T]
    loop:
        match (a.next(), b.next()):
            (Some(x), Some(y)) -> emit x; emit y
            (Some(x), None)    -> emit x
            (None, Some(y))    -> emit y
            (None, None)       -> break
```

### Infinite Sequences

Infinite sequences are valid and useful:

```orthon
# Infinite counter
fun counter() -> Iterator[Int]
    let i = 0
    loop:
        emit i
        i = i + 1

# Use with combinator that provides termination
let first_100 = counter().take(100).collect()

# Infinite Fibonacci
fun fib_infinite() -> Iterator[Int]
    let (a, b) = (0, 1)
    loop:
        emit a
        (a, b) = (b, a + b)
```

### Generator Completion

A generator completes when control flows off the end of the function body, or via an explicit `return`:

```orthon
fun read_lines(file: File) -> Iterator[String]
    while file.has_next():
        emit file.read_line()
    # implicit return — iterator is exhausted

fun limited(limit: Int) -> Iterator[Int]
    let i = 0
    while i < limit:
        emit i
        i = i + 1
    return  # explicit return — also exhausts the iterator
```

### Named Function Equivalents

Per the Named Before Symbolic principle, each form has equivalents:

```orthon
emit value                    # keyword form
return sequence(value)         # named form
yield(value)                   # free function form (legacy alias)
```

### Relationship with Iterator Protocol

Generators are the **production** side of the sequence concept. The `Iterator` trait is the **consumption** side. They are orthogonal:

- A generator function returns an `Iterator[T]`.
- Iterator combinators (`.map()`, `.filter()`, etc.) work on any `Iterator[T]`, including generators.
- You can have generators without combinators, and combinators without generators.
- Together they form the complete sequence model: produce via generators, consume via iterators.

## Default Strategy

Generator functions compile to state machines (stackless coroutines) that implement `Iterator[T]`. The `emit` keyword desugars to storing the current value and control flow state, then returning control to the consumer. On the next `next()` call, execution resumes at the point after the last `emit`. No heap allocation per generator instance — state is stored in a fixed-size struct.

## Alternative Strategies

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Stackful coroutines | Full coroutine with separate stack | When generators need deep call-stack interaction |
| Eager evaluation | Generate all values immediately | Small finite sequences with high overhead of lazy dispatch |
| Channel-based | Generator sends values through a channel | Concurrent / multi-consumer scenarios |
| Macro-based desugaring | `emit` desugars to explicit state-machine code at the AST level | Compiler implementations where stackless coroutines are not available |

## Open Questions

1. Can `emit` be used outside of generator functions (e.g., in regular functions that return `Iterator[T]`)?
2. Should generators support `emit` from within nested closures?
3. Interaction with ownership: does `emit` move or borrow the emitted value?
4. Should generators support `emit` with a destructor/cleanup for cleanup on early termination?
5. Can generators be parallelised automatically, or is that always explicit?

## Decision History

- **`emit` keyword** adopted over `yield`. Rationale: `emit` is an active verb — the generator emits a value. Consistent with Orthon's Named Before Symbolic principle. Avoids Python's `yield` confusion (which is also used in `yield from`).
- **Lazy by default** adopted (Phase 3 D-06). Rationale: Eager evaluation would break infinite sequences and increase allocation. Explicitness demands that materialisation is explicit.
- **All canonical forms equivalent** adopted. Rationale: Orthogonality — `emit value`, `return sequence(value)`, and `return value ->` are the same semantic operation in different syntax.
- **Generators implement `Iterator[T]`** adopted. Rationale: Separates production (generators) from consumption (iterators), enabling independent evolution and composition.
- **Accepted via EDR-021** on 2026-07-27.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/concepts/ITERATOR_PROTOCOL.md`
- [x] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
