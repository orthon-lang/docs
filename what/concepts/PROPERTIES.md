# Properties

## Issue (Why)

How does a language expose the data of a type without coupling consumers to its internal representation? The classical OOP answer treats fields as private implementation details exposed through explicit getter and setter methods, creating ceremony asymmetry and breaking uniform access.

Later languages (C#, Kotlin, Swift) solved this with **properties** — a single syntactic construct that unifies field storage and computed access behind a uniform `.name` interface. The caller writes `obj.name` regardless of whether it is a stored field, a computed value, or a validated setter.

## Principles

1. **Uniform access** — Callers use `.name` syntax regardless of whether the property is stored or computed.
2. **Properties over raw fields** — Every field in a type declaration is implicitly a property with a getter and optional setter.
3. **Getter/setter bodies optional** — A property auto-generates a trivial getter (return the backing field). Computed properties specify the getter body explicitly.
4. **Getter/setter visibility independent** — A property may have a public getter but a private setter.
5. **No call-site change on refactor** — Changing a stored property to a computed one never changes the call site.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Property Access Policy | Governs uniform access semantics — stored vs computed, getter/setter generation |
| Visibility Policy | Controls independent visibility on getter vs setter |

## Model (What)

Every field in a type declaration is implicitly a property:

```orthon
struct Person:
    name: String                          # implicit getter, no setter
    var age: Int                          # implicit getter + setter
    is_adult: Bool                        # computed property
        get: self.age >= 18
```

Properties unify field storage and computed access behind a uniform interface:

```orthon
p = Person(name: "Alice", age: 30)
print(p.name)            # uniform access
print(p.is_adult)        # uniform access (computed)
p.age = 31               # setter (if var)
```

## Default Strategy

Properties are a **StdLib** pattern — getter/setter sugar over attribute access + function calls. The compiler recognizes property declarations and generates trivial getter/setter implementations, but no new runtime semantics are introduced. Properties desugar to field access + function calls at the primitive level.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Language-level properties (C#/Kotlin) | Properties as a first-class language construct with special syntax. Full compiler support for visibility, inheritance, and override. |
| Explicit getters/setters (Java) | No property concept. Callers use `getX()` and `setX()` methods. Refactoring breaks call sites. |
| Descriptor protocol (Python) | Properties via `@property` decorator. Uniform access but runtime-based. |

## Open Questions

1. Should properties support `get`/`set` accessor modifiers beyond visibility (e.g., `protected set`)?
2. Should computed properties support caching (lazy evaluation)?
3. How do properties interact with delegation (`by` clause)?
4. Should properties be inheritable in trait definitions?

## Decision History

- **EDR-062:** Properties accepted as StdLib — getter/setter sugar over attribute access + function. The concept of "a named value with getter/setter" is entirely expressible via existing primitives (attribute access + function calls). The syntactic sugar of implicit getter generation is a convenience, not new semantics.
- **Classification per D-03:** StdLib. Properties desugar to field access + function calls. No compiler-level semantics beyond what attribute access already provides.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
