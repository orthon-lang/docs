# Extension Functions

## Issue (Why)

How does a programmer add behaviour to an existing type without modifying its source, extending it via inheritance, or wrapping it in an adapter? Classic solutions each have drawbacks:

1. **Inheritance** — Requires the type to be open for extension, couples new behaviour to the type hierarchy, and cannot be applied to types the programmer does not control.
2. **Wrapper/Adapter** — Requires boilerplate forwarding for every existing method, and callers must use the wrapper type.
3. **Utility functions** — Static methods that take the target type as first parameter. Call-site syntax is less natural — `sort(list)` instead of `list.sort()`.

## Principles

1. **No inheritance required** — Extension functions operate on any type, including types the programmer does not control.
2. **No access to private members** — Extension functions operate through the public API only, preserving encapsulation.
3. **Static dispatch** — Extension functions are resolved at compile time based on the **static type** of the receiver, not the runtime type.
4. **Import control** — Extension functions from other packages must be explicitly imported.
5. **Member precedence** — If a type defines a member function with the same signature, the member wins.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Name Resolution Policy | Governs extension function resolution — static receiver type, shadowing rules, precedence |
| Visibility Policy | Controls access to private members — extension functions cannot access private members |
| Import Policy | Determines how extension functions are imported across module boundaries |

## Model (What)

Extension functions are defined outside their receiver type but called with method-call syntax:

```orthon
# Definition (anywhere in the package)
fun String.isEmail() -> Bool:
    contains("@")

# Callsite (same syntax as method)
email = user.email_address.isEmail()
```

### Extension Properties

The same mechanism works for properties:

```orthon
fun String.isEmail: Bool
    get: contains("@")
```

Extension properties cannot have backing fields — they must be computed.

## Default Strategy

Extension functions are a **Language** construct — a function defined outside its receiver type but called with receiver syntax. The compiler resolves extension functions at compile time based on the static type of the receiver. Static dispatch ensures no vtable modification or monkey-patching.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Trait-based (Rust) | Extension functions are trait implementations. More disciplined but requires trait declaration. |
| C# static methods | Extension methods as static methods with `this` parameter. Same semantics, different syntax. |
| Monkey-patching (Python) | Dynamic method addition at runtime. No access control, fragile, unpredictable. |

## Open Questions

1. Can extensions be applied to types across module boundaries?
2. How does resolution work if two imported packages define the same extension function?
3. Should extension functions participate in trait/interface satisfaction?
4. Can extensions be conditionally applied (e.g., only when a generic parameter satisfies a bound)?

## Decision History

- **EDR-058:** Extension Functions accepted as Language construct — method-call syntax on receiver type requires compiler-level recognition for name resolution and static dispatch.
- **Classification per D-03:** Language. Extension functions require the compiler to recognize receiver-call syntax on types from other modules and resolve dispatch based on static type. Not expressible via composition of primitives.
- **Cross-reference:** TRAITS (EDR-019) — extension functions interact with trait method resolution (member precedence).

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
