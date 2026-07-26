# Strategy Pattern Elimination

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

How does a programmer parameterise the behaviour of a function or type without
writing a Strategy interface and multiple one-off implementation classes?

In classical GoF terms, the Strategy pattern decouples an algorithm from its
host by encapsulating it behind a common interface:

```java
// Java — Strategy pattern with interface + concrete classes
interface SortStrategy {
    <T> void sort(List<T> items, Comparator<T> cmp);
}

class QuickSort implements SortStrategy {
    public <T> void sort(List<T> items, Comparator<T> cmp) { /* ... */ }
}

class MergeSort implements SortStrategy {
    public <T> void sort(List<T> items, Comparator<T> cmp) { /* ... */ }
}

// Usage
Sorter sorter = new Sorter(new QuickSort());
sorter.sort(list);
```

The Strategy pattern requires:
- A Strategy interface declaring the algorithm contract.
- One concrete class per algorithm variant.
- Manual injection of the strategy into the host.

This is ~15–30 lines of boilerplate per strategy family — code that differs only
in the algorithm body, not in structure.

The core question: **if the language supports first-class functions (closures,
lambdas, delegates), does the Strategy pattern dissolve into ordinary behavioural
parameterisation?** Instead of a Strategy type + concrete classes, the caller
passes behaviour directly:

```
list.sort(by: fn (a, b) -> a.name < b.name)
```

## Examples

| Language | Strategy pattern needed? | Alternative |
|---|---|---|
| **Java (pre-8)** | Yes — no lambdas, no method references | Strategy interface + concrete classes is idiomatic |
| **Java (8+)** | Rarely | Lambdas + method references + `Comparator.comparing` |
| **Kotlin** | Rarely | Lambdas + function types `(T) -> R` + extension functions |
| **Python** | Rarely | First-class functions, `functools.partial`, `__call__` |
| **Rust** | Rarely | Closures + generics + `Fn`/`FnMut`/`FnOnce` traits |
| **Go** | Sometimes | Interface-based; functions satisfy interfaces, but explicit type is needed for non-trivial strategies |
| **JavaScript** | Rarely | First-class functions — callbacks are idiomatic |
| **C#** | Rarely | Delegates, lambdas, `Func<>` / `Action<>` types |

## Implications for Orthon

1. **First-class functions make Strategy implicit** — Because Orthon functions
   are first-class values with a unified call syntax (see
   [`../essential/FUNCTIONS.md`](../essential/FUNCTIONS.md)), passing behaviour
   is a natural operation. No separate Strategy type is needed:

   ```orthon
   # Passing behaviour directly — no Strategy interface
   users.sort(by: fn (a, b) -> a.name < b.name)

   # Same with a named function
   fn by_name(a, b) -> a.name < b.name
   users.sort(by: by_name)

   # Same inline with compact syntax
   users.sort(by: (a, b) => a.name < b.name)
   ```

2. **No separate Strategy type** — The `Strategy` interface + concrete class
   structure disappears. Behaviour is represented by the function type
   signature alone: `(ArgType) -> ReturnType`.

3. **Named functions serve as reusable strategies** — Any named function with
   a compatible signature can be used where a strategy is expected:

   ```orthon
   fn by_price_asc(a: Product, b: Product) -> Bool
       a.price < b.price

   fn by_price_desc(a: Product, b: Product) -> Bool
       a.price > b.price

   # Both are valid sort strategies — no interface required
   products.sort(by: by_price_asc)
   products.sort(by: by_price_desc)
   ```

4. **Closures extend strategies with state** — When a strategy needs
   configuration or accumulated state, closures supply it without a separate
   stateful object:

   ```orthon
   fn make_validator(max_length: Int) -> fn (String) -> Bool
       fn (input) -> input.len() <= max_length

   validate = make_validator(140)
   validate("hello")  # true
   validate("a" * 200)  # false
   ```

5. **When Strategy type is still warranted** — A named Strategy type remains
   useful for:
   - Strategies with complex internal state or lifecycle management.
   - Polymorphic dispatch across many strategy variants (e.g., plugin systems).
   - Explicit documentation of a behavioural extension point (the type name
     conveys intent that a bare function signature does not).
   - Cases where the strategy must be serialisable, composable, or
     introspectable beyond what closures provide.

   In these cases, a Strategy type is intentional design, not a workaround
   for a missing language feature.

## Open Questions

1. Should Orthon provide a standard `Comparator[T]` function type alias
   (`fn (T, T) -> Bool`), or leave strategy signatures entirely to user code?
2. How do first-class function strategies interact with named/default
   parameters — can a strategy parameter have a default?
3. Should the language provide built-in combinators for strategy composition
   (e.g., `and_then`, `or_else` for validation chains)?
4. Does strategy-as-function create ergonomic friction for strategies with
   more than one method (multi-method strategies such as `serialize` +
   `deserialize`)? Should these remain as explicit interfaces?

## Decision History

Initial research — no decisions recorded yet.

## Cross-References

- See [`../essential/FUNCTIONS.md`](../essential/FUNCTIONS.md) for the first-class function model that enables this elimination
- See [`../essential/COMPOSITION_OVER_INHERITANCE.md`](../essential/COMPOSITION_OVER_INHERITANCE.md) for the broader principle of behavioural composition over structural inheritance
- See [`BUILDER_PATTERN_ELIMINATION.md`](BUILDER_PATTERN_ELIMINATION.md) for a parallel analysis of another GoF pattern eliminated by language features
- See [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Composition Over Exceptions for the principle that favours passing behaviour over abstracting it

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `../essential/FUNCTIONS.md`
- [ ] `../essential/COMPOSITION_OVER_INHERITANCE.md`
- [ ] `what/GLOSSARY.md`
