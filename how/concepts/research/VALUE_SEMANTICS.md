# Value Semantics

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

How does a language distinguish between types that are copied on assignment (values) and types that are shared by identity (references), and which should be the default?

Classical OOP languages like Java make **all user-defined types reference types**. Objects are always heap-allocated, accessed through pointers, and assignment creates an alias, not a copy:

```java
// Java — every object is a reference
Point p1 = new Point(1, 2);
Point p2 = p1;       // alias, not copy
p2.x = 99;           // p1.x is also 99 — aliasing bug
```

This default-to-reference model has consequences:
- **Aliasing bugs** — mutating through one reference affects all others.
- **Defensive copying** — programmers must manually `clone()` or copy to prevent unintended sharing.
- **Equality confusion** — `==` checks identity, not structural equality; `equals()` must be manually overridden.
- **Heap allocation overhead** — even small, transient objects incur allocation and GC pressure.

A different approach: **value semantics by default**, where types are copied on assignment unless explicitly marked as shared. This is the path taken by C++ (structs are values), C# (`struct` keyword), and Swift (`struct` as the recommended default).

## Examples

| Language | Default type | Value types | Reference types | Copy semantics |
|---|---|---|---|---|
| **Java** | Reference | Primitives only (`int`, `double`) | All objects | References are copied (aliases); objects are not |
| **C#** | Reference for user types | `struct` keyword | `class` keyword | User chooses: `struct` → value, `class` → reference |
| **Swift** | Value | `struct` (preferred) | `class` | Copy on assignment for structs; `class` is shared |
| **Rust** | Value by move semantics | All types by default | `Box`, `Rc`, `Arc` | Move by default; `Clone` for explicit copy |
| **C++** | Value | All types by default | `T*` / `T&` | Copy constructor; explicit for references |
| **Python** | Reference | No distinction | All objects | Every assignment is aliasing |
| **Go** | Value | All types by default | `*T` (pointer) | Copy on assignment; explicit `&` for references |

### Swift: Structs and Classes

Swift makes `struct` the recommended building block:

```swift
// Swift — struct is a value type
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 1, y: 2)
var p2 = p1       // copy — independent
p2.x = 99         // p1.x remains 1 — no aliasing
```

Classes remain available for reference semantics (shared identity, inheritance, deinitialization), but they are the **opt-in** choice, not the default:

```swift
class SharedPoint {
    var x: Int
    var y: Int
    init(x: Int, y: Int) { self.x = x; self.y = y }
}

var a = SharedPoint(x: 1, y: 2)
var b = a          // alias — same instance
b.x = 99           // a.x is also 99
```

### C#: Struct vs Class

C# provides both but defaults to `class` (reference type), requiring explicit `struct`:

```csharp
// C#
struct Point { public int X, Y; }
class PointRef { public int X, Y; }
```

### Rust: Move Semantics

Rust goes further: **move semantics by default**. Assignment transfers ownership; copying requires an explicit `Clone` or `Copy` trait:

```rust
// Rust — move by default
let p1 = Point { x: 1, y: 2 };
let p2 = p1;          // p1 is MOVED — cannot use p1 afterwards
// let p3 = p1;       // compile error: p1 is moved
```

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Classification Policy | Determines whether types are value or reference by default |
| Copy Semantics Policy | Governs whether assignment copies (deep copy), moves, or shares (alias) |
| Allocation Policy | Stack vs heap allocation for value types |
| Equality Policy | Structural equality (`==`) vs identity equality (`===`) |

## Implications for Orthon

1. **Value semantics as default** — Orthon should make value types the default, following Swift's model. Most data (coordinates, configuration values, API payloads) benefits from copy semantics and the elimination of aliasing bugs.

2. **Explicit reference types** — If Orthon supports reference types (classes, objects with shared identity), they should be explicitly opted into — not the default choice.

3. **Copy semantics** — Copy-on-write (see [`COPY_ON_WRITE.md`](COPY_ON_WRITE.md)) provides a pragmatic middle ground: assignment is cheap (shared), mutation triggers a copy only when the value is shared.

4. **Performance** — Value types enable stack allocation and eliminate GC pressure for small, frequently-created objects.

5. **Equality** — With value semantics, structural equality (`==`) is the natural default, and identity equality (`===`) is the special case.

## Open Questions

1. Should Orthon have a separate `struct`/`class` distinction, or a single type system with annotation for reference semantics (e.g., `shared` keyword)?
2. How should value types interact with inheritance? If Orthon has no class inheritance, this is trivially solved (all types are value types with optional sharing).
3. Should large value types (e.g., a struct with 50 fields) be heap-allocated automatically, or should the programmer control allocation explicitly?
4. How does value semantics compose with mutation: can a value type have mutable fields, or is mutation only allowed through copy-and-replace?
5. Should move semantics (Rust model) be considered, or is copy semantics sufficient for Orthon's goals?
