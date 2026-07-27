# Slots

## Issue (Why)

How does a type declare a fixed, known set of attributes — restricting which fields can exist at runtime — and what benefits does that bring in terms of memory efficiency, safety, and programmer intent? Languages with dynamic attribute models (Python, JavaScript) allow arbitrary attribute assignment on any object at any time, creating two costs:
1. **Memory overhead** — Each object carries a dictionary to hold arbitrary attributes, even when the set is known at declaration time.
2. **Accidental attribute surface** — A typo silently creates a new attribute instead of signalling an error.

## Principles

1. **Fixed fields as the default** — Orthon's type system should enforce a fixed set of attributes by default for all declared types.
2. **Memory layout predictability** — Fixed-field types enable ABI-stable layout, essential for FFI and serialization.
3. **Progressive disclosure** — Allow starting with loose attributes and tightening to fixed slots as the design matures.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Field Storage Policy | Determines whether a type uses compact fixed-size slot storage or dynamic attribute storage |

## Model (What)

In a property-based type system (see PROPERTIES), every declared property is inherently a slot. The slot concept collapses into "a property with a backing field." No separate `__slots__`-like declaration is needed — declaring a property already reserves the slot.

```orthon
struct Point:
    x: Float       # slot — fixed storage, known layout
    y: Float       # slot — fixed storage, known layout
```

Dynamic attribute extension is an explicit opt-in:

```orthon
dynamic struct OpenRecord:
    name: String            # fixed slot
    # additional attributes may be added at runtime
```

## Default Strategy

Slots is a **StdLib** pattern — the StdLib provides a `Slot` annotation for compact field storage. Every property declaration in a type implicitly creates a slot. The compiler uses fixed-size layout by default; dynamic storage requires an explicit `dynamic` modifier.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Always dynamic (Python) | All objects carry per-instance dictionaries. Flexible but memory-heavy. |
| Always fixed (Rust/Java) | Fields are always fixed by the type system. Optimal layout but no dynamic escape hatch. |
| Opt-in fixed (Python `__slots__`) | Explicit `__slots__` declaration restricts fields. Default is dynamic. |

## Open Questions

1. Should Orthon allow anonymous/structural types with dynamic fields?
2. How do slots interact with the library boundary — can a library type expose additional slots?
3. Should serialization frameworks bypass the slot restriction via an explicit mechanism?

## Decision History

- **EDR-063:** Slots accepted as StdLib — compact field storage annotation. In Orthon's property-based model, every declared property is inherently a slot. The concept is a consequence of the fixed-field type system, not a separate feature.
- **Classification per D-03:** StdLib. Slots are a storage annotation — they affect memory layout but introduce no new semantic concepts. The field storage model is already part of the type system.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
