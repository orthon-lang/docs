# Mixin

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

How does a language enable sharing behaviour across unrelated types without forcing them into a common inheritance hierarchy — allowing a set of methods and state to be "mixed into" any type that needs them, regardless of its place in the type hierarchy?

Classical inheritance (Java, C++) requires that shared behaviour be placed in a common ancestor. This works for strict taxonomies (Animal → Mammal → Dog) but breaks down for cross-cutting concerns: a `Loggable` behaviour may be useful for a `DatabaseConnection`, a `UserService`, and a `GUIButton` — which otherwise have nothing to do with each other. Forcing them all to inherit from a shared `LoggableBase` class pollutes the hierarchy and violates the single-responsibility principle.

Mixin-based programming solves this by defining **behaviour fragments** that can be composed into any class irrespective of its inheritance chain:

| Language | Mechanism | Can carry state? | Method conflict resolution | Linearization |
|---|---|---|---|---|
| **Python** | Classes as mixins (cooperative MRO) | Yes — `__init__` calls via `super()` | MRO order — left-to-right, depth-first | C3 linearization |
| **Ruby** | `include` / `prepend` for modules | Yes — instance variables | Last include wins (or `prepend` wins) | Single linearization per class |
| **Scala** | `with` trait mixin | Yes | Last-mixin-overrides | Linearization by declaration order |
| **Dart** | `with` keyword for mixins | No (before 2.17) / Yes (`mixin class` 3.0+) | Last mixin wins | Flattened by declaration |
| **Rust** | No mixins; traits with default methods | No — no field inheritance | Explicit method resolution | N/A (no inheritance) |
| **Kotlin** | Interface delegation with default impl | No — interfaces carry no state | Explicit override required | N/A (single class inheritance) |

The core problem: **should Orthon support a mixin mechanism — a reusable behaviour fragment that can be composed into multiple types — and if so, should it carry state, how should method conflicts be resolved, and what is its relationship with Orthon's trait mechanism?**

## Principles

1. **Composition over inheritance** (from `why/MANIFESTO.md`) — Code reuse should be achieved by composing independent behaviour fragments, not by inheriting from a shared base class. Mixins are one form of composition.
2. **Explicit resolution** — When two mixins provide the same method name, the conflict must be resolved explicitly. The resolution should be visible at the composition site, not buried in a hidden linearization rule.
3. **No fragile base class problem** — Adding a method to a mixin should not silently change behaviour of types that use it. Name clashes must be surfaced at composition time.
4. **State ownership clarity** — If a mixin carries state (instance fields), the type system must make clear which fields belong to which mixin, avoiding name collisions and ambiguous ownership.

## Examples

### Python mixins (cooperative multiple inheritance)

```python
class LoggableMixin:
    def log(self, msg):
        print(f"[{self.__class__.__name__}] {msg}")

class SerializableMixin:
    def to_dict(self):
        return {k: v for k, v in self.__dict__.items() if not k.startswith("_")}

class User(LoggableMixin, SerializableMixin):
    def __init__(self, name):
        self.name = name

u = User("Alice")
u.log("created")           # [User] created
print(u.to_dict())         # {'name': 'Alice'}
```

### Ruby modules

```ruby
module Loggable
    def log(msg)
        puts "[#{self.class.name}] #{msg}"
    end
end

module Serializable
    def to_dict
        instance_variables.each_with_object({}) do |var, h|
            h[var.to_s.delete('@')] = instance_variable_get(var)
        end
    end
end

class User
    include Loggable
    include Serializable
    attr_accessor :name

    def initialize(name)
        @name = name
    end
end
```

### Scala traits (with state)

```scala
trait Loggable {
    def log(msg: String): Unit = println(s"[${this.getClass.getSimpleName}] $msg")
}

trait Timestamped {
    val createdAt: Long = System.currentTimeMillis()
}

class User(val name: String) extends Loggable with Timestamped {
    def info: String = s"$name (created at $createdAt)"
}
```

## Implications for Orthon

1. **Merge with trait model** — Orthon's trait system (see [`TRAITS.md`](TRAITS.md)) can subsume mixins if traits support default method implementations and associated state. A mixin is then a trait that provides concrete behaviour (methods + optional fields) rather than an abstract contract. This avoids a separate language construct.
2. **No implicit linearization** — Orthon should reject multiple composition that produces method name conflicts. The composition site must explicitly resolve clashes with an alias or override block, rather than relying on a deep MRO algorithm (like Python's C3 linearization) that programmers find hard to reason about.
3. **State in traits = fields in implementing type** — If a trait/mixin declares fields, those fields become part of the implementing type's layout. This is straightforward and avoids the "instance variable namespace collision" problem that Python mixins suffer from. Field name conflicts between traits are resolved at composition time, just like method conflicts.
4. **Composition without inheritance** — Orthon's mixin mechanism should work independently of an inheritance hierarchy. A type can mix in behaviour fragments without also participating in a class hierarchy. This aligns with Orthon's principle of composition over inheritance.
5. **Static dispatch** — Unlike Python's runtime MRO traversal, Orthon should resolve mixin method dispatch statically (at compile time), treating the composed type as a flat, monomorphised structure. This eliminates runtime overhead and enables inlining.

## Open Questions

- Should Orthon have a separate `mixin` keyword, or should the concept be fully subsumed by traits with default implementations?
- How does a mixin interact with slots (see [`SLOTS.md`](SLOTS.md))? If a mixin adds fields, are those part of the type's slot set?
- Does the evolution model allow adding a method to a published mixin without breaking existing implementations?
- Should mixins be parametrisable (generic mixins that take type parameters)?
- Can a mixin declare abstract methods that the mixing type must provide — making it a "template method" pattern?
