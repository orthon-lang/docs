# Visibility and Encapsulation

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language control access to its components — fields, methods, types — so that internal implementation details remain hidden from external consumers?

Two broad approaches dominate:

1. **Explicit access modifiers** (Java's `public`, `protected`, `private`, package-private) — keywords attached to every declaration. The programmer declares *who* can see *what*. Clear, explicit, but verbose. Every visibility permutation must be considered and annotated.

2. **Naming conventions** (Python's `_protected` and `__name_mangling`) — visibility is expressed through identifier conventions rather than language keywords. Lighter weight, but enforcement is weak: conventions are not compiler-checked. A determined caller can access anything.

The core problem: **visibility is a contract boundary, not just an access control mechanism**. When a field is marked private, the language is promising that field can change without affecting external callers. Convention-based systems cannot make this promise. Keyword-based systems can, but at the cost of ceremony.

Orthon must decide: is visibility a first-class language concern (enforced by the compiler) or a documentation concern (enforced by convention)? The choice affects every module, every API surface, and every refactoring.

## Principles

1. **Explicit is better than implicit** — Visibility rules should be syntactically visible at the declaration site, not inferred from naming patterns.
2. **Minimum necessary access** — The default should be the most restrictive visibility that still allows the component to function. Broader access must be explicitly granted.
3. **Module-boundary as the default scope** — Visibility defaults to "visible within the same module." Cross-module access is the exception that requires explicit marking.
4. **No backdoors** — There should be no mechanism (reflection, name mangling trick) to bypass visibility rules at runtime. Visibility is a compile-time guarantee.
5. **Composable with other concepts** — Visibility rules must interact cleanly with traits, generics, inheritance, and the module system without special cases.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Visibility Policy | Defines which visibility levels exist (private, module, public) and their default |
| Encapsulation Policy | Controls whether visibility is enforced at compile time, runtime, or convention-only |
| Export Policy | Governs which declarations are part of the public API of a module versus internal |
| Inheritance Policy | Determines how visibility interacts with subtyping and overriding (covariance of visibility) |

## Model (What)

The language provides a small set of visibility levels. Every declaration has exactly one visibility level. The compiler enforces access rules; there is no runtime bypass.

```
# Module scope by default — visible within the current module
fun internal_helper() -> Int
    ...

# Public — visible to all modules that import this one
pub fun public_api() -> String
    ...

# Private to the containing type or function
type Counter:
    priv count: Int = 0      # field: private to this type
    pub fun increment()       # method: part of public API
        self.count += 1
```

Key features:
- **Three levels:** `priv` (type/function-local), default (module-scoped), `pub` (exported).
- **No protected** — inheritance visibility is handled via separate mechanism (sealed types / open modules).
- **No name-mangling tricks** — `priv` fields are truly inaccessible outside their scope; the compiler rejects cross-boundary access.
- **Module is the encapsulation boundary**, not the class. This matches the principle that modules, not types, are the primary organizational unit.
- **`pub` on a declaration is an explicit contract** — changing it later is a breaking change. Non-`pub` declarations can be refactored freely within the module.

### Comparison with Java

| Aspect | Java | Python | Orthon (proposed) |
|---|---|---|---|
| Levels | `public`, `protected`, `private`, package-private | `_`, `__` (naming conventions) | `pub`, default (module), `priv` |
| Enforcement | Compiler | Convention (no enforcement) | Compiler |
| Default | Package-private | Public | Module |
| Bypass | Reflection | `_Class__method` trick | None |
| Per-member | Yes | Yes | Yes |
| Per-module | Package | No | Module |

## Default Strategy

By default, every declaration has **module-level visibility** — it is accessible from anywhere within the same module but inaccessible from outside the module. `pub` and `priv` are explicit opt-in for broader or narrower access.

This default minimizes the API surface that must be considered for breaking changes, while avoiding the ceremony of Java's per-declaration modifiers for internal code.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Convention-only | No language-level visibility keywords. Visibility expressed through naming conventions (Python-style). Compiler may warn on convention violations but does not enforce. |
| All-private default | Everything is private by default; every cross-scope access requires explicit `pub`. Safe but verbose. |
| All-public default | Everything is public by default (JavaScript/TypeScript-style). Minimal ceremony but no encapsulation guarantees. |
| File-level visibility | Visibility is scoped to a single file rather than a module. More granular but breaks module-level refactoring. |

## Open Questions

1. Should there be a `protected` level for inheritance hierarchies? Or should inheritance visibility be handled through sealed types instead?
2. How does visibility interact with the Execution Program model — can a program option restrict visibility at the boundary?
3. Should `pub` on a type imply `pub` on its members, or should members default to `priv` even on `pub` types?
4. How does visibility compose with structural typing — can a type satisfy a trait using private methods?
5. Should there be a mechanism for friends / trusted modules?

## Decision History

*To be filled during Concept Design Review (Milestone 2).*

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `what/GLOSSARY.md`
- [ ] `what/PRIMITIVE_BLOCKS.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/IMPLEMENTATION_POLICIES.md`
- [ ] `how/strategies/IMPLEMENTATION_STRATEGIES.md`
- [ ] Other: `what/SEMANTIC_MODEL.md`
