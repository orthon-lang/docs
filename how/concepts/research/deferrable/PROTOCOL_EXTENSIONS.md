# Protocol Extensions

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

How does a language add behaviour to existing types without modifying their definition, or provide default implementations for protocol requirements?

Consider a common scenario: you have an `Equatable` protocol that requires `==`, and you want all equatable types to also get `!=` for free:

```java
// Java — you must manually implement every method
interface Equatable<T> {
    boolean equals(T other);
    // !== is NOT provided — every implementor must define it
}
```

Protocol extensions solve two problems:

1. **Default implementations** — A protocol can provide default implementations of some methods, so conforming types only need to implement the non-defaulted requirements.
2. **Retroactive conformance** — An existing type (including standard library types) can be extended to conform to a protocol without modifying the original type's source.

Swift's protocol extensions combine both capabilities into a single mechanism:

```swift
// Swift — protocol extension with default implementation
protocol Equatable {
    static func ==(lhs: Self, rhs: Self) -> Bool
}

extension Equatable {
    static func !=(lhs: Self, rhs: Self) -> Bool {
        return !(lhs == rhs)  // default implementation using ==
    }
}

// Now any type conforming to Equatable gets != for free
extension String: Equatable {} // String already has ==, now also has !=
```

The core problem: **how can a language provide an "open set" of behaviours that can be attached to types after those types are defined, without breaking coherence?**

## Examples

| Language | Default impl on interface | Retroactive conformance | Coherence rules |
|---|---|---|---|
| **Swift** | `extension Protocol { func ... }` | Any type can conform | No orphan rule; two modules can conflict |
| **Rust** | Trait default methods | `impl Trait for Type` in same crate | Orphan rule: impl requires either trait or type from current crate |
| **Kotlin** | Extension functions (static) | `fun Type.method()` | No coherence concerns (extension functions are statically dispatched) |
| **Java** | `default` methods (Java 8+) | No | N/A (interface must be known at type definition) |
| **C#** | `default` interface methods (C# 8+) | Partial | Similar to Java |
| **Haskell** | Typeclass default methods | Orphan instances allowed (flagged) | Orphan instances are allowed but discouraged |

### Swift: Protocol Extensions

Swift's protocol extensions are the most comprehensive mechanism:

```swift
// Define a protocol
protocol Drawable {
    func draw()
}

// Provide a default implementation
extension Drawable {
    func draw() { print("default draw") }
}

// Retroactive conformance — make an existing type conform
extension Int: Drawable {
    func draw() { print(self) }
}

// Conformance without implementation — uses default
struct Circle: Drawable {} // uses default draw()
```

Protocol extensions also enable **conditional conformance**:

```swift
extension Array: Drawable where Element: Drawable {
    func draw() { for e in self { e.draw() } }
}
```

### Rust: Trait Default Methods

Rust traits also support default implementations:

```rust
trait Drawable {
    fn draw(&self) {
        println!("default draw");  // default implementation
    }
}

// Explicit override
impl Drawable for Circle {
    fn draw(&self) { /* custom */ }
}

// Uses default
impl Drawable for Square {} // inherits default draw()
```

Rust's **orphan rule** prevents retroactive conformance conflicts: you can only implement a foreign trait for a foreign type if at least one of the two is defined in your crate. This prevents the coherence problems that can arise in Swift when two libraries both extend the same type to conform to the same protocol.

### Kotlin: Extension Functions

Kotlin extension functions are statically dispatched, not part of interface conformance:

```kotlin
// Kotlin — extension function
fun String.isEmail(): Boolean = this.contains("@")

"test@example.com".isEmail() // true
```

Extension functions are **purely syntactic sugar**: they desugar to static method calls. They cannot override existing methods and do not constitute interface conformance.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Trait/Protocol Policy | Governs whether protocols can have default implementations |
| Coherence Policy | Determines rules for conflicting retroactive conformances (orphan rule, overlapping impls) |
| Extension Policy | Controls how types can be extended without modification |

## Implications for Orthon

1. **Default implementations on traits** — If Orthon follows a trait-based architecture (see [`TRAITS.md`](TRAITS.md)), default implementations are essential. They allow the trait designer to provide common behaviour while letting conforming types override when needed.

2. **Retroactive conformance** — The ability to make existing types conform to new protocols is powerful, especially for LLM-generated code where the LLM may need to extend standard library types. However, it requires coherence rules to prevent conflicts.

3. **Orphan rule** — Orthon should adopt a Rust-style orphan rule (you can only implement a trait for a type if you own the trait or the type) to prevent coherence conflicts. This is especially important if Orthon has a package/module system.

4. **Conditional conformance** — The ability to say "Array conforms to Drawable only when Element conforms to Drawable" is valuable for generic types. This should be part of Orthon's trait system.

5. **Relationship to mixins** — Protocol extensions with default implementations overlap with mixins (see [`MIXIN.md`](MIXIN.md)). Orthon should decide whether protocol extensions replace mixins entirely, or whether mixins provide additional capabilities (e.g., carrying state).

## Open Questions

1. Should protocol extensions carry state? Swift protocol extensions cannot define stored properties; this is intentional (it preserves the interface-like nature of protocols). Should Orthon allow state in extensions?
2. How should conflict resolution work when two protocol extensions define the same method? Swift uses the most specific override; Rust requires explicit disambiguation.
3. Should Orthon have extension functions separate from protocol conformance (Kotlin model), or should all extensions be tied to protocol/trait conformance (Swift/Rust model)?
4. What are the implications for the name resolution system in [`how/architecture/NAME_RESOLUTION.md`](../architecture/NAME_RESOLUTION.md)?
