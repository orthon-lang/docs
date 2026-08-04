# Dependency Slots — `require` / `using` Dual-Level Resolution

> **✅ ACCEPTED — [EDR-081](../../how/decision_records/architecture/EDR-081-require-using-dependency-slots.md).**
>
> **Status:** Accepted 2026-07-30.
>
> **Classification:** **Language Pattern (Level 2)** — the `require`/`using`
> keyword split and class-level dependency slots are syntax sugar with no new
> semantics. The mechanism is identical to EDR-037's Dual Parameter Model.
>
> **Refines:** [EDR-037](../../how/decision_records/architecture/EDR-037-context-parameters.md)
> (Context Parameters).
>
> **See also:** [`CONTEXT_LIMITED_MODULES.md`](CONTEXT_LIMITED_MODULES.md),
> [`GLOSSARY.md`](../GLOSSARY.md) § Context Parameter, Dependency Slot,
> [`SEMANTIC_MODEL.md`](../SEMANTIC_MODEL.md) § Evaluation (Implicit context flow),
> [`PRIMITIVE_BLOCKS.md`](../PRIMITIVE_BLOCKS.md)

---

## Issue (Why)

EDR-037 accepted the Dual Parameter Model, separating function parameters into
**data** and **context** spaces with automatic `given`/`using` resolution. Three
issues remained unresolved from EDR-037's deferred full specification:

1. **`using` is syntactically overloaded** — the same keyword serves two
   distinct cognitive roles: declaration ("this function needs a `Database`")
   and resolution ("I am providing `prod_db`"). Scala 3's `given`/`using`
   confusion is precedential — an LLM-native language should not replicate this
   ambiguity.
2. **No grouping mechanism for shared dependencies** — when multiple functions
   share the same dependencies, `using` must be repeated on every signature.
3. **No instance-scoped resolution level** — two objects of the same class
   should use different dependencies (prod Database vs test Database) without
   per-call repetition or coarse module-level `given` switches.

## Principles

1. **One cognitive role per keyword** — `require` (declaration) and `using`
   (resolution) are distinct, syntactically visible keywords.
2. **LLM-native explicitness** — Explicit name resolution; no auto-naming from
   type that would create ambiguity with same-type dependencies.
3. **Class-level dependency slots** — A class declares a `require` clause once;
   slots are visible in all methods, filled at construction, with a compile-time
   initialization guarantee.
4. **No coupling of concerns** — Traits do not carry `require`; dependency slots
   are an implementation concern, not part of a behavioural contract.
5. **Per-instance resolution** — Dependency slots enable prod/test differentiation
   at instance granularity.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Dependency Resolution Policy | Governs `require`/`using` dual-level resolution (declaration vs provision) |
| Context Resolution Policy | Builds on EDR-037's implicit context flow (SEMANTIC_MODEL § Evaluation) |
| Scope Policy | Defines slot visibility within a class and per-instance filling |

## Model (What)

### `require` / `using` Keyword Split

| Keyword | Role | Location | Purpose |
|---------|------|----------|---------|
| `require` | Declaration | Signature | "This function/class needs X" |
| `using` | Resolution | Call/construction site | "I am providing X for this call" |

```orthon
// Declaration — what does this function require?
fun process(order_id: Int) require Database db, Logger log -> Receipt

// Resolution — what am I providing for this call?
process(42) using prod_db, prod_log
```

### Class-Level Dependency Slots

A class declares a `require` clause, creating dependency slots that are visible
in all methods (no repetition), filled at construction time per-instance, and
carry a compile-time initialization guarantee (no `null`, no uninitialized state):

```orthon
class UserService require Database db, Logger log:
    fun find_user(id: Int) -> Option[User]
        // db and log are available here without redeclaration

let prod_svc = UserService(using prod_db, prod_log)
let test_svc = UserService(using test_db, test_log)
```

## Default Strategy

`require`/`using` is a Level 2 (Language Pattern) transformation — syntax sugar
with no new semantics. The mechanism is identical to EDR-037's `using`/`using`;
only the keyword distinction and class-level slot grouping are added. Per-instance
resolution is the default for class dependency slots.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Keep single `using` keyword (status quo from EDR-037) | Preserves ambiguity that an LLM-native language should not replicate — rejected. |
| Auto-naming from type (`require Database` → `db`) | Ambiguous with same-type dependencies; LLMs need explicit name resolution — rejected (deferred as post-v0.1 sugar). |
| Module-level `given` only (no per-instance) | Too coarse for test/prod differentiation at instance granularity — rejected. |
| Traits can carry `require` | Couples dependency concern to behavioural contract — rejected. |

## Open Questions

1. Should `using` support partial provision (providing a subset of required
   slots with the rest resolved from `given`)?
2. How do dependency slots interact with the module capability model
   (CONTEXT_LIMITED_MODULES)?

## Decision History

- **EDR-081:** Dependency Slots accepted as Language Pattern (Level 2), refining
  EDR-037. Adds the `require`/`using` keyword split and class-level dependency
  slots for per-instance resolution. Concept Pipeline run: 10 Decision Pipeline
  questions answered, 7 Validation Gates passed, 19 Language Design Gate
  criteria satisfied.
