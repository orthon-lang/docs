# Final by Default

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

Should a class be extensible (inheritable) by default, or should inheritance be an explicit opt-in?

Java answers: **extensible by default**. Every class can be subclassed unless marked `final`. This default has driven two decades of real-world consequences:

```java
// Java — extensible by default
public class HashMap<K, V> { ... }  // anyone can extend it
```

The problems this creates:

1. **Fragile base class** — Changes to a parent class can break all subclasses. The parent's implementation becomes part of its API contract, making it impossible to refactor without risking downstream breakage.

2. **Unintended API surface** — Every public method on a class becomes part of the inheritance API. Subclasses can override methods intended for internal use. The class designer must anticipate all possible overrides.

3. **Design-for-inheritance tax** — To safely allow subclassing, Java designers must use patterns like Template Method, document which methods are intended for override (`@Override`), and maintain compatibility guarantees that constrain evolution. Bloch's *Effective Java* item 19: "Design and document for inheritance or else prohibit it."

4. **Double maintenance** — Every bug fix or optimisation in a parent class must consider whether it breaks subclasses that may depend on the current implementation.

The alternative: **closed by default**. Classes are `final` unless explicitly declared `open` for inheritance. This is the approach taken by Kotlin (the JVM-language successor to many Java pain points): `class` is final by default, `open class` permits inheritance, and `abstract class` is always open.

## Examples

| Language | Default | Inheritable keyword | Non-inheritable keyword | Rationale |
|---|---|---|---|---|
| **Java** | Extensible | (default) | `final class` | Historical; class hierarchies were primary reuse mechanism |
| **Swift** | Closed | `open class` | `class` (default) | Inheritance as exception, not rule |
| **Kotlin** | Closed | `open class` | `class` (default) | Direct response to Java's fragile base class problems |
| **C#** | Closed | `class` | `sealed class` | Same motivation as Kotlin |
| **Scala** | Extensible | (default) | `final class` | Follows Java convention |
| **Rust** | N/A | N/A | N/A | No class inheritance; uses traits |

### Swift: `final` by default, `open` as opt-in

Swift requires explicit `open` to enable subclassing outside the module:

```swift
// Swift — closed by default
class Base {}                    // final within module, final outside module
open class ExtensibleBase {}     // can be subclassed outside module
```

A class without `open` is `final`. A class with `open` signals a deliberate design decision: the author has considered the extensibility surface and accepted the maintenance commitment.

### Kotlin: Same approach

```kotlin
// Kotlin
class Closed {}                  // final — cannot be subclassed
open class OpenForExtension {}   // explicitly designed for inheritance
```

Kotlin's designers made this change explicitly in response to Java's experience: "We decided that classes should be closed by default" (JetBrains, Kotlin design).

### Rust: No inheritance at all

Rust sidesteps the question entirely — there are no classes, no inheritance. Code reuse happens through composition (struct embedding), traits (behavioural abstraction), and generics. This is the most radical solution and the one that most completely eliminates the fragile base class problem.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Inheritance Policy | Determines whether implementation inheritance exists, and whether it is opt-in or opt-out |
| Type Classification Policy | Affects whether class types are open or closed by default |
| API Surface Policy | Governs what methods/fields are part of the inheritance contract |

## Implications for Orthon

1. **Closed by default** — If Orthon has any form of class or type inheritance, the default should be "closed" (not inheritable), and inheritance should require explicit `open`. This follows the proven Kotlin/Swift model and avoids replicating Java's fragile base class problems.

2. **Consider whether Orthon needs inheritance at all** — Rust's approach (no class inheritance, only trait-based polymorphism) is the cleanest solution. If Orthon follows a protocol/trait-oriented architecture (see [`TRAITS.md`](TRAITS.md), [`COMPOSITION_OVER_INHERITANCE.md`](COMPOSITION_OVER_INHERITANCE.md)), then "final by default" becomes moot because there are no classes to inherit from.

3. **Protocol/trait inheritance** — If Orthon uses traits or protocols, the question shifts: should a trait require explicit `open` for extension by other traits? Swift protocols are extensible by default (any type can conform); Rust traits require an explicit `impl` but follow coherence rules.

4. **LLM Generability Gate** — Closed-by-default simplifies LLM reasoning: the LLM can assume a type's behaviour is fixed unless explicitly told otherwise. Open inheritance introduces non-local reasoning: modifying a parent may affect unseen subclasses.

## Open Questions

1. Does Orthon need class inheritance at all? If the language is built on value types (see [`VALUE_SEMANTICS.md`](VALUE_SEMANTICS.md)) and protocols/traits, the question of "final by default" never arises.
2. If Orthon has both value types and reference types, should only the reference types support inheritance?
3. Should there be a middle ground between `final` and `open` — e.g., a class that can be subclassed only within the same module (Swift's `public` vs `open`)?
4. How does "final by default" interact with testing and mocking? Java's mocking frameworks rely on inheritable classes; closed-by-default requires protocol-based mocking (which Swift does with protocol mocks).
