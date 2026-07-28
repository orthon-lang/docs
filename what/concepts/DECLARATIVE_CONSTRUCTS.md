# Declarative Constructs

## Issue (Why)

What makes a language construct "declarative" — and which common programming tasks should Orthon provide declarative constructs for, versus expressing imperatively through composition of primitives? A construct is declarative when the programmer specifies *what* the result should be, and the language or standard library determines *how* to achieve it.

For LLM-native design, declarative constructs reduce the probability of generation errors because the LLM specifies intent rather than implementation steps.

## Principles

1. **Declarative by default for common transformations** — Filtering, mapping, sorting, grouping use declarative syntax.
2. **Imperative escape hatch always available** — Every declarative construct has a well-defined desugaring to imperative primitives.
3. **Synthesis-friendly signature** — Minimal, predictable API surface.
4. **No hidden state or implicit dependencies** — Operate on explicit arguments only.
5. **Idiom, not framework** — Language-level or StdLib-level idioms.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Declarative Construct Policy | Defines which common tasks have built-in declarative constructs |
| Desugaring Policy | Specifies the canonical desugaring for each declarative construct |
| Collection Policy | Determines lazy vs eager evaluation semantics |

## Model (What)

### Categories of Declarative Constructs

Declarative constructs in Orthon fall into several categories:

#### 1. Collection Transformations

```orthon
# Declarative: what to do
let adults = users
    .filter(u -> u.age >= 18)
    .map(u -> u.name)
    .sorted()

# Equivalent imperative: how to do it
let adults = List::new()
for user in users:
    if user.age >= 18:
        adults.push(user.name)
adults.sort()
```

#### 2. Resource Management

```orthon
# Declarative: resource scope with automatic cleanup
using file = open("data.txt"):
    let content = file.read_all()
    process(content)
# file is closed when the block exits
```

#### 3. Sorting and Ordering

```orthon
let sorted = users.sorted(by: .age)
let by_name_age = users.sorted(by: [.last_name, .first_name, .age])
```

#### 4. Derived Structural Methods

```orthon
@derive(Eq, Hash, Show)
struct Point(x: Int, y: Int)
```

### What is NOT a Declarative Construct

- Custom operators — violate readability and LLM predictability.
- Domain-specific sublanguages — bypass compiler verification.
- Implicit conversions — all conversions must be syntactically visible.

## Default Strategy

Declarative Constructs is a **StdLib** documentation concept — the StdLib provides declarative APIs (`.filter()`, `.map()`, `.sorted()`, `using`, `@derive`). Each construct has a specified desugaring to imperative primitives. No single new language syntax — instead, the language already provides the primitives for declarative patterns. This concept documents the pattern.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Minimal declarative surface (Go) | Few special cases — higher LLM error rate for complex transformations. |
| Rich declarative surface (Kotlin/Scala) | Many declarative constructs — larger language surface. |
| Library-based declarative (Rust Itertools) | Language stays minimal — requires library awareness. |

## Open Questions

1. Should declarative constructs be lazy or eager by default?
2. How do declarative constructs interact with error handling?
3. Should query expressions (comprehensions) be part of v0.1?

## Decision History

- **EDR-073:** Declarative Constructs accepted as StdLib (documentation-only) — declarative sugar over imperative patterns. The concept documents which patterns Orthon considers declarative and how each desugars to primitives. No new language semantics — all constructs are already provided by existing concepts.
- **Classification per D-03:** StdLib (documentation-only). This is a meta-concept that catalogues existing declarative patterns. No new compiler semantics.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
