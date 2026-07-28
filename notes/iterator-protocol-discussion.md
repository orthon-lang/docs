# Iterator Protocol — Discussion Notes

> Date: 2026-07-28
> Context: Review of `what/concepts/ITERATOR_PROTOCOL.md` (EDR-022, accepted 2026-07-27)
> Key questions: Sequence relationship, Iterable vs IntoIterator, lazy/eager semantics,
> `for` desugaring, lambda composition, closure capture safety.

---

## 1. Iterator vs. Sequence

Three-layer relationship:

```
Sequence (abstract concept — what the result is, not how it's produced)
    ↑ production: emit (generators) — LAZY_SEQUENCE_GENERATORS.md
    ↑ consumption: Iterator[T] (protocol) — ITERATOR_PROTOCOL.md
```

- **Sequence** is a fundamental type: "values produced over time". It describes *what*, not *how*.
- **Generator** (`emit`) produces a Sequence.
- **Iterator** (`Iterator[T]` trait) consumes a Sequence.
- The two sides are **orthogonal** — you can have iteration without generation (collection iteration) and generation without iteration (producing values).

## 2. No Separate `Iterable` — `IntoIterator[T]` Fills That Role

Orthon does **not** define a separate `Iterable` concept. Instead:

```orthon
trait IntoIterator[T]
    fn iter(self) -> Iterator[T]
```

This is what `for` loops and combinator entry points accept. It covers:
- Collections (implement `IntoIterator[T]`)
- `Iterator[T]` itself (implements `IntoIterator[T]`, returning `self`)
- Range expressions (`0..10`, `0..=10`)
- I/O streams

Rationale: A separate `Iterable` would be a redundant entity — `IntoIterator` already provides the same abstraction ("this can be turned into an iterator") with an explicit `iter()` method, consistent with **Minimal Core** and **Explicit Over Implicit**.

## 3. Lazy by Default, Eager Only via Explicit `.collect()`

**Lazy** is the default and the architectural invariant:
- Combinators (`.map()`, `.filter()`, `.take()`, `.fold()`) are **always lazy** — no intermediate allocation.
- The only way to get eager materialisation is explicit **`.collect()`**.
- Range iterators are zero-cost — compile to simple counter loops, no heap allocation.
- See `PRIMITIVE_BLOCKS.md`: "produces values on demand, not eagerly."

Alternative strategies (implementation-level, not semantic):
| Strategy | Description | When to Use |
|----------|-------------|-------------|
| Eager iteration | Materialise all elements before iteration | Small collections where lazy overhead dominates |
| Parallel iteration | Split iterator across threads | Large datasets, multi-core targets |
| Streaming iteration | Iterator yielding `Result<T, E>` | I/O streams |

## 4. `for` Loop Desugaring

```orthon
for item in collection:
    process(item)

# Desugars to:
let mut it = collection@iter()    # IntoIterator::iter() via @ protocol access
loop:
    match it@next():               # Iterator::next() via @ protocol access
        Some(item) -> process(item)
        None       -> break
```

Key mechanisms:
- **`collection@iter()`** — calls `IntoIterator::iter()` via `@` protocol access (D-07).
- **`it@next()`** — calls `Iterator::next()`, returns `Option[T]`.
- Compiler rejects non-iterable types at compile time.
- `@` prefix distinguishes protocol method access from attribute access.

### All Canonical Forms

```orthon
# Form 1: for loop (preferred for simple iteration)
for item in collection:
    process(item)

# Form 2: Explicit next() calls
let mut it = collection@iter()
loop:
    let item = it@next()
    match item:
        Some(value) -> process(value)
        None        -> break

# Form 3: Combinator chain (preferred for transformations)
collection
    .filter(|x| x > 0)
    .map(|x| x * 2)
    .for_each(|x| process(x))
```

## 5. Iterator + Lambda Composition

Three composition patterns:

### A. Combinators accept lambdas directly
```orthon
collection
    .filter(|x| x > 0)
    .map(|x| x * 2)
    .fold(0, |acc, x| acc + x)
```

Combinator signatures:
```orthon
fn map[U](self, fn: T -> U) -> Iterator[U]
fn filter(self, pred: T -> Bool) -> Iterator[T]
```

### B. Named function equivalents (Named Before Symbolic)
```orthon
collection.filter(pred)     # method form
filter(collection, pred)     # free function form
```

### C. Manual iteration with lambdas
```orthon
let it = collection@iter()
loop:
    match it@next():
        Some(item) -> process_with_lambda(item, |x| transform(x))
        None       -> break
```

## 6. No `i=i` Problem — Explicit Closure Capture

Orthon avoids the classic JavaScript/Python "loop variable capture" problem through **explicit closure capture** (FUNCTIONS.md, principle 3):

> *"Closure capture is explicit — Variables captured by a closure must be explicitly listed or syntactically visible. No silent capture of the surrounding scope."*

### Problematic pattern (implicit — rejected):
```orthon
for i in 0..10:
    funcs.append(fn () -> i)   # ERROR: implicit capture
```

### Correct patterns (explicit):
```orthon
# By-value capture (copied at closure creation time)
for i in 0..10:
    funcs.append(fn [val = i] () -> val)

# By-reference capture via Mutable
let counter = fn [count: Mutable(0)] () -> Int
    count += 1
    count
```

### Additional safety: single-pass semantics

Iterator combinators are consumed immediately, so even in `for`:
```orthon
for i in 0..10:
    process_with_lambda(i, |x| x * 2)  # lambda called on the same iteration
```

The `i=i` problem only arises when a lambda **outlives** its iteration (stored in a list, passed to deferred callback). Orthon's explicit capture makes this visible and intentional.

### Comparison with other languages

| Aspect | JavaScript/Python | Orthon |
|--------|-------------------|--------|
| Capture | Implicit (by reference) | **Explicit** — `[val = i]` |
| `i=i` workaround | Required (default-arg hack / `let`) | **Impossible** — implicit capture is a compile error |
| By-value capture | Hack via default args | Native syntax `[val = expr]` |
| By-reference capture | Default | Via `Mutable` or explicit `ref` |

---

## Open Questions from the Document

1. Should `Iterator[T]` support `size_hint()` for optimising collection pre-allocation?
2. Should there be a `DoubleEndedIterator[T]` (`.next_back()`) for bidirectional traversal?
3. Should combinators support parallel execution (`.par_map()`) or should that be a separate concept?
4. Should `for` accept owned collections directly (via `IntoIterator`) or only references?
5. How does the iterator protocol interact with the `delegate` execution policy?

---

## Affected Documents (still unchecked)

From ITERATOR_PROTOCOL.md's checklist:
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `what/SYNTAX.md`
- [ ] `what/SEMANTIC_MODEL.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
