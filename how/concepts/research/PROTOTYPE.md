# Prototype

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

How does a language model objects that share behaviour through a live delegation chain — where an object can defer an attribute or method lookup to another object at runtime — rather than through a static class hierarchy?

Most mainstream languages use a **class-based** object model: a class defines the shape and behaviour of all its instances, and inheritance is a static relationship declared at class definition time. Prototype-based languages (JavaScript, Self, Lua) use a different model: an object can be created as a clone of an existing object (its **prototype**), and any property or method not found on the object itself is looked up on the prototype chain at runtime.

| Aspect | Class-based (Java, C++, Python) | Prototype-based (JavaScript, Self, Lua) |
|---|---|---|
| **Object creation** | `new ClassName(args)` — instantiation from a class | `Object.create(proto)` or `{}` — cloning or literal |
| **Behaviour sharing** | Inheritance declared at class definition, static | Delegation — live chain, mutable at runtime |
| **Type of a class** | A class is a compile-time construct | A prototype is a runtime object |
| **Changing shape** | Impossible — class is fixed | Trivial — add/remove properties on any object |
| **Performance** | Optimised by JIT — vtable dispatch | Harder to optimise — inline caches, hidden classes |
| **Metaprogramming** | Reflection API | Direct — objects are just dictionaries with a link |

The core problem: **should Orthon support a prototype-based object model as a complement to (or instead of) a class-based model, and what use cases would justify the complexity of having both?**

## Principles

1. **Minimal core** (from `why/MANIFESTO.md`) — Every object model Orthon supports must be justified by concrete use cases. If prototypes can be expressed as a library pattern on top of a class model (or vice versa), one of them is redundant.
2. **Explicitness** (from `DESIGN_PRINCIPLES.md`) — If an object delegates to another, that delegation must be syntactically visible. Implicit prototype chain walking (as in JavaScript's `obj.foo` walking up the chain) obscures which object actually provides the behaviour.
3. **Composability** — The prototype model, if it exists, must compose with the rest of the type system: traits, generics, and pattern matching should all work uniformly.
4. **Performance predictability** — A prototype chain with runtime-mutable links makes static analysis difficult. Orthon must make clear whether a given object uses prototype delegation, so the programmer can reason about performance.

## Examples

### JavaScript (prototype-based)

```javascript
const animal = {
    speak() { return `${this.name} makes a sound.`; }
};

const dog = Object.create(animal);
dog.name = "Rex";
dog.speak();  // "Rex makes a sound." — delegation to animal

// Prototype chain: dog → animal → Object.prototype → null

// Runtime mutation:
animal.speak = function() { return `${this.name} barks.`; };
dog.speak();  // "Rex barks." — live delegation
```

### Lua (prototype via metatables)

```lua
local Animal = { name = "" }
function Animal:speak()
    return self.name .. " makes a sound."
end

local dog = { name = "Rex" }
setmetatable(dog, { __index = Animal })
print(dog:speak())  -- "Rex makes a sound."
```

### Self (pure prototype language)

```
"Self — no classes, only prototypes"
traits point = (    "a prototype object, not a class"
    x = 0.
    y = 0.
    print = ( 'x: ', x, ' y: ', y ).
)

"Create a clone with specific values"
p: traits point copy.
p x: 10.
p y: 20.
p print.
```

## Implications for Orthon

1. **Not a primary object model** — Orthon's primary model should be class/trait-based (see [`TRAITS.md`](TRAITS.md) and [`OBJECTS_AND_SINGLETONS.md`](OBJECTS_AND_SINGLETONS.md)). Prototypes introduce live delegation chains that conflict with static type checking, memory layout predictability, and compilation to native code.
2. **Possible as a library pattern** — A prototype-style delegation mechanism could be provided as a standard library abstraction using Orthon's existing metaprogramming facilities — e.g., a `Delegator<T>` type that forwards method calls to a dynamically-assigned delegate object. This keeps the core language clean while enabling the pattern where needed.
3. **Not for typed, performance-sensitive code** — Prototype chains are inherently dynamic and hard to optimise. Orthon's default strategy (see `DEFAULT_STRATEGY.md`) targets predictable performance; prototype delegation would belong in a scripting/interpreted strategy at best.
4. **Relationship with dynamic records** — If Orthon supports dynamic attribute sets (see [`SLOTS.md`](SLOTS.md)), a prototype chain could be modelled as a linked list of dynamic records. This is a natural composition but should be explicitly constructed, not implicit in the property lookup rule.

## Open Questions

- Are there use cases in the Orthon domain (systems programming, data processing, embedded) that genuinely require prototype-based delegation rather than traits or composition?
- Could a prototype chain be modelled in Orthon's type system as a generic `Chain<T>` type with an explicit `.lookup(key)` method, rather than implicit property delegation?
- How does prototype delegation interact with Orthon's ownership and lifetime model — who owns the prototype object?
- Is there value in a limited form of delegation that is statically typed and checked (e.g., a "delegates to" clause that is resolved at compile time)?
