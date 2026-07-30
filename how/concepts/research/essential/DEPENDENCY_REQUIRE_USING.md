# Dependency Slots — `require` / `using` Dual-Level Resolution

> **⚠️ DRAFT — Preliminary hypothesis.**
> Refines the Dual Parameter Model (EDR-037) by splitting `using` into
> two distinct keywords (`require` / `using`) and introducing
> class-level dependency slots as a third resolution level between
> per-call and compile-time global.
>
> **Hypothesis status:** Proposed
> **Last updated:** 2026-07-30
>
> **See also:**
> - [`CONTEXT_PARAMETERS.md`](CONTEXT_PARAMETERS.md) — Dual Parameter Model (parent concept)
> - [`EDR-037`](../../decision_records/architecture/EDR-037-context-parameters.md) — Context Parameters accepted as SEMANTIC_MODEL correction
> - [`CLASS_WITH_ACT.md`](CLASS_WITH_ACT.md) — Class as reference type with optional `act`
> - [`STRUCT_AS_NOMINAL_PRODUCT_TYPE.md`](../important/STRUCT_AS_NOMINAL_PRODUCT_TYPE.md) — Struct vs class distinction
> - [`SINGLETON_PATTERN_ANALYSIS.md`](../../deferrable/SINGLETON_PATTERN_ANALYSIS.md) — Dependency Policy (`using`/`given`)
> - [`FUNCTIONS.md`](FUNCTIONS.md) — First-class functions model
> - [`../../what/SEMANTIC_MODEL.md`](../../what/SEMANTIC_MODEL.md) — Evaluation & Visibility dimensions

---

## Hypothesis

> *If Orthon separates dependency declaration (`require`) from dependency
> resolution (`using`) as distinct syntactic roles, and extends the Dual
> Parameter Model to allow class-level dependency slots that are filled
> per-instance at construction time (distinct from both per-call function-level
> `using` and module-level `given`), then: (1) the syntax is more expressive
> because declaration versus resolution are visually distinct, (2) classes
> naturally group methods sharing a common environment without compromising
> Data First, (3) testability is preserved because each instance can receive
> its own mock dependencies, and (4) the three orthogonal resolution levels
> (per-call, per-instance, per-module) compose freely without overloading
> a single keyword.*

---

## Issue (Why)

The Dual Parameter Model (EDR-037, `CONTEXT_PARAMETERS.md`) splits function
parameters into data `()` and context `using`. This solves the signal-to-noise
problem in function signatures. Three issues remain:

### 1. `using` is syntactically overloaded

`using` does two distinct jobs:

| Role | Where | Example |
|------|-------|---------|
| **Declaration** — declares that a dependency slot exists | Signature | `fun f(x) using db: Database` |
| **Resolution** — fills that slot with a concrete value | Call site | `f(x) using prod_db` |

In Scala 3 — the closest precedent — this is a well-known source of confusion:
`given` declaration, `using` clause, and `using` argument look similar but
behave differently. An LLM-native language should not replicate this ambiguity.

### 2. No grouping mechanism for shared dependencies

When multiple functions share the same dependencies (e.g., `Database`, `Logger`),
the `using` clause must be repeated on every function. Classes exist in Orthon
but carry no mechanism to capture shared context — every method must redeclare
its `using` independently, or the programmer falls back to fields + constructor
parameters (losing compile-time resolution guarantees).

### 3. No instance-scoped resolution level

The Dual Parameter Model (EDR-037) recognises two levels:

| Level | Resolution |
|-------|-----------|
| **Per-call** | `f(x) using dep` |
| **Module/compile-time** | `given dep = ...` / implicit scope |

There is a gap: **per-instance**. When you want two objects of the same class
to use different dependencies (prod Database vs test Database), you must either
pass the dependency per-call (repetitive) or rely on module-level `given`
switches (coarse-grained). Neither is ergonomic.

---

## Proposal

### 3.1. Split: `require` (declaration) vs `using` (resolution)

```orthon
// Declaration — what does this function require?
fun process(order_id: Int) require Database db, Logger log -> Receipt

// Resolution — what am I providing for this call?
process(42) using prod_db, prod_log
```

| Keyword | Role | Location | Purpose |
|---------|------|----------|---------|
| `require` | Declaration | Signature | Lists dependency types and their local names |
| `using` | Resolution | Call / construction site | Fills slots with concrete values |

Rationale:
- Visually distinct — declaration vs provision are different cognitive acts
- LLM-friendly — `require` in the signature tells the LLM "these names are available in scope"
- No new keywords invented — both exist in other languages with similar semantics
- `require` reads as a contract ("this function needs X"), `using` reads as provision ("supplying X")

### 3.2. Class-level dependency slots (per-instance)

```orthon
class OrderService require Database db, Logger log
    // db, log are visible in all methods — filled per-instance

    fun get(id: Int) -> User
        db.query("SELECT * FROM users WHERE id = ?", id)

    fun save(user: User) -> Unit
        db.execute("INSERT INTO users ...")
```

Construction fills the slots:

```orthon
given prod_db: Database = PostgresConnection("...")
given prod_log: Logger = StdoutLogger()

svc_prod = OrderService() using prod_db, prod_log
svc_test = OrderService() using MockDatabase(), TestLogger()
```

Key properties:

- **Per-instance, not per-class.** Two instances can hold different dependencies
- **Not a field in the Java sense.** The slot is a compile-tracked binding with
  a guarantee of initialization — no `null`, no uninitialized-state errors
- **Zero-cost beyond a direct field reference.** The compiler desugars the slot
  into a constructor parameter + field, or elides it entirely if the dependency
  is unused on a given code path
- **Data First preserved.** The class groups related functions; the dependency
  slot is environment, not data. Methods still operate on data parameters

### 3.3. Three orthogonal resolution levels

| Level | Syntax | Resolved when | Scope |
|-------|--------|---------------|-------|
| **Per-call** | `fun f(x) require Dep` → `f(x) using d` | At each call site | Single call |
| **Per-instance** | `class C require Dep` → `C() using d` | At construction | Instance lifetime |
| **Module/compile-time** | `given Dep = ...` | At compile time | Compilation unit |

These compose:

```orthon
class OrderService require Database db, Logger log

    // Per-call override — function-level require shadows class-level
    fun get_debug(id: Int) require Logger debug_log -> User
        // debug_log overrides class-level log for this call
        debug_log.info("debug: ...")
        db.query(...)          // db still from class-level slot

    // No require at all — pure data function
    fun calculate_tax(amount: Decimal) -> Decimal
        amount * 0.2
```

### 3.4. Explicit naming only (no auto-naming from type)

```orthon
// ✅ Explicit:
class OrderService require Database db, Logger log

// ❌ NOT allowed — auto-naming from type:
class OrderService require Database, Logger
    // db? database? data_source? — ambiguous
```

Rationale:
- LLM must know exactly which names are in scope when generating method bodies
- Two dependencies of the same type (`Database primary, Database secondary`)
  cannot be disambiguated by type alone
- `Type name` is the Orthon convention (cf. `val x: Int`, `fun f(x: Int)`)
- Auto-naming can be added as syntactic sugar post-v0.1 if warranted

---

## Interaction with Existing Concepts

### Interaction with `given` instances

```orthon
// Module-level given provides default resolution:
given db: Database = PostgresConnection("...")

// Class slot can be filled from given or explicitly:
svc1 = OrderService()          // implicit — uses given db
svc2 = OrderService() using MockDatabase()  // explicit override
```

The resolution priority follows existing `given` rules (EDR-037):
1. Explicit `using` at construction site → highest priority
2. Local `given` → medium priority
3. Imported `given` → lowest priority
4. Missing → compile-time error (slot must be filled)

### Interaction with `act` (concurrent isolation)

```orthon
class CounterService require Logger log
    act counter: Int = 0

    act increment()
        counter += 1
        log.info("counter incremented")    // log — not act-isolated, ok
```

Dependency slots are not `act`-isolated by default (they hold environment,
not protected state). If a dependency requires isolation, it should be
declared as `act` field independently.

### Interaction with traits

```orthon
trait Queryable
    fun find(id: Int) -> User

class UserService require Database db
    implements Queryable

    fun find(id: Int) -> User
        db.query("SELECT ...")
```

The `require` clause is part of the class declaration, not the trait contract.
A trait cannot declare `require` — dependency slots are an implementation
concern. This preserves orthogonality: traits define data contracts, classes
define implementation grouping.

---

## Policy Footprint

| Policy Type | Role |
|-------------|------|
| Dependency Resolution Policy | When are dependency slots filled (per-call, per-instance, per-module)? |
| Name Binding Policy | How do `require` names map to visible identifiers in method bodies? |
| Compile-time Enforcement Policy | What guarantees does the compiler make about slot initialization? |

---

## Trade-offs

- **+** Two keywords (`require`/`using`) are more explicit than one overloaded keyword
- **+** Per-instance slots fill a real gap between per-call and module-level resolution
- **+** Works with `given` instances and explicit resolution — not an either/or
- **+** Testability per-instance (mock per object) without sacrificing compile-time guarantees
- **−** More keywords to learn (`require` in addition to `using`/`given`)
- **−** Class-level `require` may tempt users to bundle all dependencies into a single
  class (Service Locator anti-pattern) — coding guidelines needed
- **−** `given` resolution for class slots adds compiler complexity (type-directed
  matching against slot declarations)

---

## Open Questions

1. **Can a class have both `require` and regular constructor parameters?**
   `class Svc(port: Int) require Database db, Logger log` — does the slot
   interfere with explicit data parameters?

2. **Can a standalone function (outside a class) have `require`?**
   Logically yes — `fun f(x) require Dep d` is isomorphic to
   `fun f(x) using Dep d`. The question is whether `require` is allowed
   outside class declarations, or reserved for class-level slots.

3. **Should `require` be allowed on individual methods inside a class?**
   e.g., `class Svc require Database db; fun special(x) require Extra extra`.
   This composes per-call and per-instance — useful but adds complexity.

4. **How does `require` interact with effect declarations?**
   `proc f() require Logger log` — is `require` orthogonal to effects?
   (Hypothesis: yes — dependency is environment, effect is what the function
   does.)

5. **Inheritance and `require`:** If `class Derived require Database db` extends
   a base class, does the base's `require` compose? (Orthon has no class
   inheritance beyond Object, so this is moot — but if traits ever carry
   require, the question reopens.)

---

## Gate Criteria

- [ ] Define `require` syntax for function signatures and class declarations
- [ ] Define `using` syntax for call sites and construction sites
- [ ] Specify resolution priority: explicit `using` > local `given` > imported `given`
- [ ] Specify desugaring of class-level slots to constructor + field
- [ ] Define interaction with existing `given` mechanism (EDR-037)
- [ ] Document interaction with `act` modifier (CLASS_WITH_ACT.md)
- [ ] Add entries to `what/GLOSSARY.md` for "dependency slot", "require", "resolution level"
- [ ] Run Decision Pipeline to confirm this is a refinement of EDR-037, not a new concept
- [ ] Verify no conflict with EDR-078 (class not primary composition unit)
