# Declarative Constructs

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-26
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

What makes a language construct "declarative" — and which common programming tasks should Orthon provide declarative constructs for, versus expressing imperatively through composition of primitives?

A construct is declarative when the programmer specifies *what* the result should be, and the language or standard library determines *how* to achieve it. The spectrum runs from fully imperative (step-by-step instructions) to fully declarative (constraint description, no algorithm specified).

The tension is between:

- **Expressiveness** — declarative constructs are more concise, closer to the problem domain, and easier for both humans and LLMs to generate correctly.
- **Generality** — imperative primitives compose freely. Every declarative construct adds surface area and a special case to the language. The wrong declarative construct becomes a legacy burden (see `imperative-crutch-sorting.md`).

For LLM-native design, this tension has a specific shape: **a declarative construct reduces the probability of generation errors** because the LLM specifies intent rather than implementation steps. A declaration like `sorted(list)` is harder to get wrong than an imperative bubble sort. But the set of declarative constructs must be *principled* — not a grab bag of convenience features.

## Principles

1. **Declarative by default for common transformations** — Filtering, mapping, sorting, grouping, and similar data transformations use declarative syntax (pipeline combinators, query expressions) rather than explicit loops.

2. **Imperative escape hatch always available** — Every declarative construct has a well-defined desugaring to imperative primitives. No declarative construct adds expressive power that composition of primitives cannot achieve.

3. **Synthesis-friendly signature** — A declarative construct must have a predictable, minimal API surface. The LLM should be able to generate the correct construct given a natural language description of intent.

4. **No hidden state or implicit dependencies** — Declarative constructs operate on their explicit arguments only. No implicit context, thread-local state, or ambient configuration. This preserves Orthon's explicitness principle and makes generated code predictable.

5. **Idiom, not framework** — Declarative constructs are language-level or stdlib-level idioms, not external frameworks. They are discoverable through the Schema Provider and consistently named.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Declarative Construct Policy | Defines which common tasks have built-in declarative constructs and which are left to composition. |
| Desugaring Policy | Specifies the canonical desugaring for each declarative construct (what primitive composition it replaces). |
| Collection Policy | Determines whether collection transformations use lazy/streaming semantics or eager evaluation by default. |
| Query Expression Policy | Controls whether query-expression syntax (SQL-like comprehensions) is available for collection and data-source operations. |

## Model (What)

### Categories of Declarative Constructs

Declarative constructs in Orthon fall into several categories, each with a rationale for inclusion:

#### 1. Collection Transformations

Already covered by established concepts — `map`, `filter`, `reduce`, `flatMap` as methods on collection types. The key design choice is whether these are eager (produce new collection immediately) or lazy (produce a lazy view).

```orthon
// Declarative: what to do
let adults = users
    .filter(u -> u.age >= 18)
    .map(u -> u.name)
    .sorted()

// Equivalent imperative: how to do it
let adults = List::new()
for user in users
    if user.age >= 18
        adults.push(user.name)
adults.sort()
```

For LLM generation, the declarative form is significantly more robust: the LLM specifies a chain of intent, and the compiler validates each step's type.

#### 2. Resource Management

Already addressed by resource-management primitives (defer/RAII). The declarative form is:

```orthon
// Declarative: resource scope with automatic cleanup
using file = open("data.txt")
    let content = file.read_all()
    process(content)
// file is closed when the block exits, regardless of how

// Equivalent imperative
let file = open("data.txt")
try
    let content = file.read_all()
    process(content)
finally
    file.close()
```

See `imperative-crutch-resource-management.md` for the full analysis of why a declarative resource construct eliminates a common class of LLM generation errors (forgotten cleanup, incorrect error-path handling).

#### 3. Sorting and Ordering

A declarative comparator specification eliminates error-prone manual comparison logic:

```orthon
// Declarative: specify the sort key
let sorted = users.sorted(by: .age)
let by_name_age = users.sorted(by: [.last_name, .first_name, .age])

// Equivalent imperative comparator
users.sort(fn (a, b) ->
    if a.age < b.age return Order::Less
    if a.age > b.age return Order::Greater
    return Order::Equal
)
```

See `imperative-crutch-sorting.md` for the analysis of why declarative comparators reduce LLM generation errors.

#### 4. Serialization / Deserialization

Declarative annotations derive serialization from structure:

```orthon
// Declarative: describe the mapping
@json(rename_all = "snake_case")
struct User
    name: String
    email_address: String

let json = User.to_json(user)   // {"name": "...", "email_address": "..."}
```

See `imperative-crutch-serialization.md` for the analysis of why derived serialization eliminates an entire class of maintenance bugs.

#### 5. Equality, Hashing, Copying

Automatically derived from structure, following `DATACLASSES.md`:

```orthon
struct Point
    x: Float
    y: Float
    // eq, hash, copy are derived automatically
```

#### 6. Query Expressions (Future)

For complex data-source operations, a comprehension syntax may be warranted:

```orthon
// Declarative query expression
let result = from user in users
             where user.age >= 18
             order by user.name
             select user.email
```

See `QUERY_EXPRESSIONS.md` for the full analysis. This is deferred to v0.2 or later.

### What Is NOT a Declarative Construct

The following are intentionally not candidates for declarative constructs:

- **Custom operators** — would violate readability and LLM predictability (see `CUSTOM_OPERATORS.md`).
- **Domain-specific sublanguages** — external DSLs embedded in string literals bypass compiler verification.
- **Implicit conversions** — declarative does not mean implicit; all conversions must be syntactically visible.
- **Dynamic code generation** — Orthon has `AST_MACROS.md` and `COMPILE_TIME_EXECUTION.md` for metaprogramming, which is a separate concern from declarative constructs.

### Synthesis-Friendliness Criteria

A construct qualifies as declarative (and thus an LLM-synthesis-friendly idiom) when it satisfies all of:

1. **Single intent** — The construct expresses exactly one operation. `sorted(by: key)` sorts; it does not sort, filter, and log.
2. **Deterministic output** — Given the same input, the construct always produces the same output. No randomness, no ambient state.
3. **Type-checked inputs** — All arguments participate in type checking. The compiler catches wrong key paths, mismatched comparators, etc.
4. **Canonical form** — There should be one obvious way to express the construct, not six competing variations.
5. **LLM-guessable name** — The construct's name should be predictable from a natural language description: "sort by field" → `.sorted(by: .field)`.

## Default Strategy

Provide declarative constructs for collection transformations (map/filter/reduce), resource management (using), sorting (sorted by key), derived serialization (@json/@xml), and derived structural methods (eq/hash/copy). All others expressed through composition of imperative primitives. Every declarative construct has a specified desugaring to imperative primitives. Lazy evaluation by default for collection transformations, with an explicit `.collect()` to materialize.

## Alternative Strategies

| Strategy | Languages | Trade-offs |
|---|---|---|
| **Minimal declarative surface** | Go, C | Few special cases, easy to learn. Every task written imperatively. Higher LLM error rate for complex transformations. |
| **Rich declarative surface** | Kotlin, Scala, C# (LINQ) | Many declarative constructs. Concise, expressive, but larger language surface and more to learn. Higher chance of construct overlap (multiple ways to do the same thing). |
| **Library-based declarative** | Rust (Itertools), Python (itertools) | Language stays minimal; functionality lives in libraries. Requires library awareness from LLM. Discovery through documentation, not Schema Provider. |
| **Comprehension-based** | Haskell, Python (list comprehensions) | Single general-purpose syntax for many operations. Elegant but can be cryptic for complex transformations. |

## Open Questions

1. Should declarative constructs be lazy by default or eager by default? Lazy minimizes allocations; eager matches programmer intuition about when code runs.
2. Should derived serialization be language-level (via annotations) or library-level (via trait implementation with derive macro)?
3. How do declarative constructs interact with error handling — should `filter` propagate errors from the predicate?
4. Should query expressions (comprehensions) be part of v0.1 or deferred to v0.2?
5. Should there be a mechanism for user-defined declarative constructs (e.g., custom combinators with known desugaring)?
6. How should the Schema Provider expose declarative construct signatures to guide LLM generation?

## References

- [`SORTING.md`](../important/SORTING.md) — declarative sorting comparators
- [`QUERY_EXPRESSIONS.md`](../deferrable/QUERY_EXPRESSIONS.md) — comprehension syntax for collection/data-source operations
- [`COLLECTIONS_FUNCTIONAL_API.md`](../deferrable/COLLECTIONS_FUNCTIONAL_API.md) — declarative collection transformations
- [`DATACLASSES.md`](../important/DATACLASSES.md) — derived structural methods
- [`SEMANTIC_ANNOTATIONS.md`](../deferrable/SEMANTIC_ANNOTATIONS.md) — annotations for derived serialization
- [`imperative-crutches-index.md`](../imperative-crutches-index.md) — index of imperative patterns with declarative alternatives
- [`imperative-crutch-sorting.md`](../imperative-crutch-sorting.md) — analysis of imperative sorting burden
- [`imperative-crutch-serialization.md`](../imperative-crutch-serialization.md) — analysis of imperative serialization burden
- [`imperative-crutch-resource-management.md`](../imperative-crutch-resource-management.md) — analysis of imperative resource cleanup burden
- [`COMPILER_AS_STATIC_ANALYZER.md`](../essential/COMPILER_AS_STATIC_ANALYZER.md) — how declarative constructs enable compiler verification
- [`IDEMPOTENT_GENERATION.md`](../deferrable/IDEMPOTENT_GENERATION.md) — deterministic output from declarative constructs
