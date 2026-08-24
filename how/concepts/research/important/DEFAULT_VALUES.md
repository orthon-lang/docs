# Default Values

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-08-24

## Issue (Why)

How does a programmer specify the value a parameter takes when no argument is supplied at a call site? Without default values, every optional parameter must be handled through a combination of overloads, `Option<T>`, or builder patterns — all of which add noise without adding expressiveness.

The core problem: **a parameter's "absent" state should not require a separate type wrapper** (`Option<T>`) when the absent case maps naturally to a well-defined sensible default.

Default values sit at the intersection of three concerns: type semantics (what is the effective type when a default is used?), binding semantics (does the default preserve or widen a literal type?), and call-site ergonomics (can the caller omit the argument cleanly?).

## Principles

1. **Defaults are part of the signature** — A parameter with a default is still a typed parameter; the default is metadata on the parameter, not a separate mechanism.
2. **Explicit wins** — An explicitly supplied argument always overrides the default; there is no ambiguity.
3. **Literal type preservation** — If the default value is a literal, the parameter type should preserve the literal type (consistent with LITERAL_TYPES widening rule for `let`-style bindings).
4. **Composable with Option<T>** — A parameter with a default is NOT the same as `Option<T>`: the former says "a sensible default exists", the latter says "the caller must acknowledge absence". Use `Option<T>` when the caller must consciously choose.
5. **No runtime overhead by default** — Defaults are resolved at the call site when possible; no allocation or closure unless the default expression requires it.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Default Policy | Governs how default values are resolved: eagerly (at call site) or lazily (at first use) |
| Type Compatibility Policy | Determines whether a defaulted parameter's type widens (mutable var) or preserves literal type (let-equivalent) |
| Narrowing Policy | Affects how defaults interact with pattern matching on the caller's side |
| Evaluation Policy | Controls whether the default expression is evaluated once (shared) or per-call (copied) |

## Model (What)

### Syntax

```orthon
# Function parameter with default
fun connect(host: String = "localhost", port: Int = 8080, tls: Bool = false)
    ...

# Call site — all three forms equivalent
connect()                           # all defaults
connect(host: "example.com")        # host overridden
connect(tls: true, port: 443)       # named, out-of-order
```

### Default value and literal type preservation

```orthon
# With literal type default preserved (immutable parameter binding)
fun method_tag(tag: String = "GET") -> String
    tag  # type: "GET", not String

# With widening (mutable parameter binding — if Orthon allows this)
fun optional_tag(var tag: String = "GET") -> String
    tag  # type: String (widened)
```

### Default vs. Option<T>

```orthon
# Default — caller does not need to think about this parameter
fun timeout(duration: Duration = Duration.seconds(30))

# Option<T> — caller must consciously acknowledge absence
fun user_name(user: Option<String> = None)
    match user:
        case Some(name) => ...
        case None      => ...
```

### Interaction with struct fields

```orthon
# Struct fields with defaults (see also OBJECT_INITIALIZATION)
struct Config
    host: String = "localhost"
    port: Int = 8080

# Object initialization uses named parameters with defaults
let cfg = Config(host: "example.com")  # port uses default
```

### Evaluation semantics

```orthon
# Eager default (default policy = eager)
fun timestamp(created: Instant = Instant.now())  # evaluated at each call

# Lazy default (default policy = lazy) — deferred beyond v0.1
fun id(counter: Supplier(Int) = Supplier(counter))  # evaluated once, shared
```

## Default Strategy

**Eager evaluation at call site** — the default expression is evaluated each time the call site omits the parameter. This is the simplest model: no closure allocation, no shared mutable state, predictable semantics.

The compiler resolves defaults at the call site, substituting the default expression inline before generating code. No heap allocation unless the default expression itself requires it.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Lazy evaluation | Default expression evaluated once on first use; subsequent calls reuse the result. Rejected for v0.1 — introduces identity issues and shared mutable state complexity. |
| Shared default (singleton) | Default evaluated once at module load, shared across all calls. Rejected — surprising identity semantics. |
| Builder-required | Omitting a parameter requires `None` to be passed explicitly (Option<T> pattern). Rejected — contradicts the principle that defaults should not require extra syntax. |

## Open Questions

1. **Lazy defaults** — Should a future policy allow lazy defaults (`Supplier<T>` wrapper) for expensive default computations (file handles, random generators)? Deferred beyond v0.1.
2. **Mutable default state** — Should a mutable default (e.g., counter) be allowed? If so, does each call site share the state or have its own copy? Deferred — identity and thread-safety concerns.
3. **Default in destructuring** — Can patterns in destructuring binds have defaults? (`let Some(x) = result else x = 0`) — interaction with ERROR_UNION needs design.
4. **Interaction with named arguments** — Can named argument syntax be used for non-defaulted parameters at the call site? (Yes — named arguments are orthogonal to defaults, see OBJECT_INITIALIZATION.)
5. **Literal type preservation rule** — Does the "literal type preserved for immutable bindings" rule apply to parameter bindings with defaults? Needs explicit decision aligned with LITERAL_TYPES widening policy.

## Decision History

_(To be filled during Concept Design Review)_

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/concepts/LITERAL_TYPES.md`
- [ ] `what/concepts/UNION_INTERSECTION_TYPES.md`
- [ ] `how/architecture/TYPE_SYSTEM.md`
- [ ] `how/concepts/research/important/OBJECT_INITIALIZATION.md`
- [ ] `how/concepts/research/important/ERROR_UNION.md`
