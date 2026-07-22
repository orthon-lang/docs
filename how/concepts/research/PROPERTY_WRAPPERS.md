# Property Wrappers

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

How does a language encapsulate repeated property logic — lazy initialisation, observation, validation, thread-safe access — without requiring boilerplate getters/setters in every type?

Many properties follow recurring patterns that are identical in structure but differ in configuration:

- **Lazy properties** — initialised on first access, cached afterwards.
- **Observed properties** — a callback fires when the value changes.
- **Clamped/validated properties** — values are constrained to a range.
- **Thread-safe access** — reads and writes are synchronised.
- **Derived from UserDefaults / database** — transparent persistence.

In Java, each pattern requires manual getter/setter boilerplate:

```java
// Java — lazy initialisation: ~10 lines of boilerplate
private ExpensiveObject _lazy;
public ExpensiveObject getLazy() {
    if (_lazy == null) {
        _lazy = new ExpensiveObject();
    }
    return _lazy;
}
```

This boilerplate is mechanically identical for every lazy property in every class. The same pattern repeats for observation, validation, etc.

Swift's **property wrappers** (and Kotlin's **delegated properties**) solve this by making the property *behaviour* a reusable declaration:

```swift
// Swift — property wrapper
@Lazy var expensive = ExpensiveObject()  // declarative lazy init
@Published var name: String              // observable property
@Clamped(to: 0...100) var percentage: Double  // validated range
```

## Examples

| Language | Mechanism | Reusable? | Configured at use-site? | Can wrap computed props? |
|---|---|---|---|---|
| **Swift** | `@propertyWrapper` | Yes — define once | Yes — arguments in attribute | Yes (projected value) |
| **Kotlin** | `by` delegate | Yes — define once | Yes — delegate constructor args | No (delegates to stored prop) |
| **Java** | None — manual boilerplate | No | N/A | N/A |
| **C#** | No built-in; aspects via Fody | External tooling | Partial | Via IL weaving |
| **Python** | `@property` decorator | Per-property only | No — must redefine per property | Yes |
| **Dart** | No built-in wrapper | No | N/A | N/A |

### Swift: `@propertyWrapper`

Swift's property wrappers are defined as types annotated with `@propertyWrapper`:

```swift
// Swift — defining a property wrapper
@propertyWrapper
struct Clamped<T: Comparable> {
    private var value: T
    private let range: ClosedRange<T>

    init(wrappedValue: T, range: ClosedRange<T>) {
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
        self.range = range
    }

    var wrappedValue: T {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }
}

// Usage
struct Settings {
    @Clamped(range: 0...100) var brightness: Double = 50.0
}
```

Key features:
- **`wrappedValue`** — the property get/set behaviour.
- **`projectedValue`** (optional) — accessed via `$` prefix (e.g., `$name` for `@Published` provides a publisher).
- **`init(wrappedValue:)`** — default initialisation.
- **Composable** — multiple wrappers can be applied, though not directly in Swift (nesting is manual).

Swift's standard library provides:
- `@Published` — observable property (Combine framework)
- `@State`, `@Binding`, `@StateObject` — SwiftUI state management
- `@AppStorage` — transparent `UserDefaults` persistence
- `@ScaledMetric` — dynamic type scaling

### Kotlin: Delegated Properties

Kotlin uses delegated properties with the `by` keyword:

```kotlin
// Kotlin — delegated property
class Example {
    var p: String by Delegate()
}

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

Standard delegates include:
- `lazy {}` — lazy initialisation
- `observable(initial) { prop, old, new -> }` — observation
- `vetoable(initial) { prop, old, new -> }` — validation
- `NotNull()` — nullable-to-non-null after init

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Property Policy | Governs whether property wrappers/delegates are part of the language |
| Syntax Policy | Determines the attribute syntax (`@wrapper` vs `by delegate`) |
| Metaprogramming Policy | Property wrappers are a limited form of metaprogramming — affects the overall metaprogramming story |

## Implications for Orthon

1. **Eliminating getter/setter boilerplate** — Property wrappers are a natural extension of the property concept (see [`PROPERTIES.md`](PROPERTIES.md)). If Orthon has properties, it should consider property wrappers to eliminate the recurring patterns (lazy, observed, clamped) that would otherwise require boilerplate.

2. **Relationship to annotations/attributes** — Property wrappers overlap with [`METADATA_ANNOTATIONS.md`](METADATA_ANNOTATIONS.md) in syntax but are semantically different: annotations are passive metadata consumed by the compiler or runtime, while property wrappers actively transform the property's behaviour. Orthon should decide whether these are the same mechanism or separate.

3. **LLM Generability Gate** — Property wrappers improve LLM generability because the LLM can use a declarative `@Lazy` or `@Published` annotation instead of writing imperative getter/setter code. The LLM needs to know the wrapper's name and configuration only, not its implementation.

4. **Standard library vs language built-in** — Should property wrappers be implemented as a language mechanism (Swift), a library feature (Kotlin delegates via conventions), or not at all? The library approach is simpler to implement but less discoverable.

5. **Composition** — Can multiple wrappers be applied to a single property? If so, what is the composition order and conflict resolution?

## Open Questions

1. Should Orthon support property wrappers at all, or should the same patterns be expressed through higher-order functions and explicit composition?
2. If property wrappers are supported, what is the minimum viable set that should be in the standard library? (`lazy`, `observed`, `clamped`)
3. Should property wrappers project a value (Swift's `$` prefix) for additional functionality?
4. How do property wrappers interact with the mutation model (see [`MUTABILITY.md`](MUTABILITY.md)) and value semantics (see [`VALUE_SEMANTICS.md`](VALUE_SEMANTICS.md))?
5. Should property wrappers be exposed via the type system (i.e., `@Lazy String` is a distinct type from `String`)?
