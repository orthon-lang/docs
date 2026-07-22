# Default Interface Methods

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

How should a language evolve an interface contract without breaking existing implementations?

Java originally required interfaces to declare only abstract methods. Adding a new method to an interface was a binary breaking change because every implementing class had to define the new method. C# introduced **default interface methods** to allow interface authors to provide a default implementation, enabling interface extension without immediately breaking existing implementors.

The core problem is: should Orthon permit interfaces to carry implementation alongside declaration, or should interfaces remain pure contracts with all behavior supplied by concrete types or traits?

## Examples

| Language | Interface/trait implementation | Evolution model | Breakage on extension |
|---|---|---|---|
| **Java (pre-8)** | Abstract methods only | No default behavior | Breaking if new methods added |
| **Java 8+** | Default methods in interfaces | Mixed contract + behavior | Minor compatibility in many cases |
| **C# 8+** | Default interface methods | Allows interface method bodies | Backward-compatible additions |
| **Kotlin** | Interfaces can contain default implementations | Similar to C# | Compatible for existing implementors |
| **Rust** | Traits include default method bodies | First-class method defaults | Compatible unless required methods change |
| **Swift** | Protocol extensions provide defaults | Separate from protocol declaration | Compatible; not virtual by default |

### Java Example

```java
interface Logger {
    void log(String message);

    default void logInfo(String message) {
        log("INFO: " + message);
    }
}
```

### C# Example

```csharp
public interface ILogger {
    void Log(string message);

    void LogInfo(string message) {
        Log("INFO: " + message);
    }
}
```

## Implications for Orthon

1. **Interface methods may carry default bodies** — Interfaces can specify a default behavior for some operations while still allowing concrete types to override the implementation.
2. **Evolution without breakage** — Interface authors can add new methods with default implementations, reducing the compatibility burden on existing code.
3. **Behavioral contracts remain explicit** — Default implementations should be clearly annotated so the contract is not hidden in implementation details.
4. **Conflict resolution rules are required** — When a type inherits defaults from multiple interfaces, Orthon must define which implementation wins or whether explicit override is required.
5. **Interfaces as behaviour carriers** — This makes interfaces more similar to traits and mixins, which can change the way Orthon designs abstract APIs.

## Open Questions

1. Should default interface methods be allowed only for optional, non-essential helpers, or for any interface method?
2. How should Orthon resolve diamond inheritance when two interfaces provide conflicting default implementations?
3. Should default implementations be able to access private helper members of the interface, or only public/protected declarations?
4. Should interface defaults participate in dynamic dispatch, or be statically bound at compile time?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [INTERFACE.md](INTERFACE.md) (if / when interface research exists)
- [PATTERN_MATCHING.md](PATTERN_MATCHING.md)
- [FUNCTIONS.md](FUNCTIONS.md)
