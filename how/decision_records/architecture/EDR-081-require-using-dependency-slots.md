# EDR-081: Dependency Slots — `require` / `using` Dual-Level Resolution

**Status:** Accepted

**Date:** 2026-07-30

**Category:** Architecture

**Scope:** Subsystem (Language Patterns — Level 2)

---

### Context

EDR-037 (Context Parameters) accepted the Dual Parameter Model as a SEMANTIC_MODEL correction, separating function parameters into **data** and **context** spaces with automatic `given`/`using` resolution. Three issues remain unresolved from EDR-037's deferred full specification:

1. **`using` is syntactically overloaded.** The same keyword serves two distinct cognitive roles: declaration ("this function needs a `Database`") and resolution ("I am providing `prod_db`"). Scala 3's `given`/`using` confusion is precedential — an LLM-native language should not replicate this ambiguity.

2. **No grouping mechanism for shared dependencies.** When multiple functions share the same dependencies (e.g., `Database`, `Logger`), `using` must be repeated on every function signature. Classes exist in Orthon as reference types (per `CLASS_WITH_ACT.md`) but carry no mechanism to capture shared environment — every method must redeclare its `using` independently, or the programmer falls back to fields + constructor parameters (losing compile-time resolution guarantees).

3. **No instance-scoped resolution level.** EDR-037 recognises two levels (per-call and module/compile-time via `given`). The gap is per-instance: when two objects of the same class should use different dependencies (prod Database vs test Database), the programmer must either pass per-call (repetitive) or rely on module-level `given` switches (coarse-grained).

The research document [`DEPENDENCY_REQUIRE_USING.md`](../../concepts/research/essential/DEPENDENCY_REQUIRE_USING.md) proposes a refinement that addresses all three issues. The Concept Pipeline was run (see pipeline pass output) — all 10 Decision Pipeline questions answered, 7 Validation Gates passed, 19 Language Design Gate criteria satisfied.

---

### Decision

**EDR-037 is refined as follows.** The Dual Parameter Model is extended with two orthogonal additions:

#### 1. `require` / `using` keyword split

`using` is split into two distinct keywords reflecting two distinct cognitive roles:

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

This is a **Level 2 (Language Pattern)** transformation — syntax sugar with no new semantics. The mechanism is identical to EDR-037's `using`/`using`; only the keyword distinction is added.

#### 2. Class-level dependency slots (per-instance resolution level)

A class may declare a `require` clause, creating dependency slots that:
- Are visible in all methods of the class (no repetition)
- Are filled at construction time, per-instance
- Carry a compile-time initialization guarantee (no `null`, no uninitialized state)
- Desugar to constructor parameter + field

```orthon
class OrderService require Database db, Logger log
    fun get(id: Int) -> User
        db.query("SELECT ...", id)

    fun save(user: User) -> Unit
        db.execute("INSERT INTO ...")

// Per-instance filling:
svc_prod = OrderService() using PostgresConnection("..."), StdoutLogger()
svc_test = OrderService() using MockDatabase(), TestLogger()
```

This is a **Level 2 (Language Pattern)** — composition of `pack` (slot layout) + `scope` (visibility across methods) + `assignment` (filled at construction) + compiler invariant (initialization guarantee). The invariant is analogous to `Option<T>` narrowing (EDR-028): a compile-time check layered on primitive composition.

#### 3. Three orthogonal resolution levels

| Level | Syntax | Resolved when | Scope |
|-------|--------|---------------|-------|
| **Per-call** | `fun f(x) require Dep` → `f(x) using d` | At each call site | Single call |
| **Per-instance** | `class C require Dep` → `C() using d` | At construction | Instance lifetime |
| **Module/compile-time** | `given Dep = ...` | At compile time | Compilation unit |

These compose freely — per-method `require` may shadow class-level slots for individual calls.

#### 4. Explicit naming only

Dependency slots use `Type name` convention (e.g., `Database db`), never auto-naming from type alone. Rationale: two dependencies of the same type cannot be disambiguated by type alone; `Type name` matches Orthon's existing `val x: Int` convention; LLMs benefit from knowing exact names in scope.

---

### Consequences

- **Positive:**
  - `require`/`using` split eliminates the Scala-3-confusion risk for both humans and LLMs — declaration and resolution are visually and semantically distinct.
  - Per-instance slots fill a real gap between per-call (verbose) and module-level (coarse) resolution, improving testability without sacrificing compile-time guarantees.
  - Works with existing `given` mechanism — not a replacement, a complementary resolution level.
  - EDR-078 compatibility confirmed: class is used as a dependency scope boundary, not as a primary composition unit. Methods still operate on data parameters.
  - The initialization guarantee prevents a class of bugs (uninitialized dependency access) that manual constructor-field patterns allow.

- **Negative:**
  - One new keyword (`require`) adds to the language's surface area, justified by eliminating `using`'s semantic overload.
  - Class-level `require` may tempt users to bundle all dependencies into a single class (Service Locator anti-pattern) — coding guidelines needed.
  - `given` resolution for class slots adds compiler complexity (type-directed matching against slot declarations at construction sites).
  - Per-method `require` override (open question 3) adds complexity if implemented — deferred to post-acceptance resolution.

---

### Compliance

1. The `require`/`using` split must be documented in the Language Specification as a Level 2 pattern — the resolution semantics are identical to EDR-037's mechanism.
2. Class-level dependency slots must be desugared to constructor parameter + field in the spec, making the compilation strategy explicit.
3. The initialization guarantee (every slot filled before any method body executes) must be listed as a compiler invariant in the specification.
4. EDR-078 compliance: no design document may present class-level `require` as making classes the "primary" composition unit. It is one resolution level among three.
5. Coding guidelines must warn against bundling >3-4 dependency slots in a single class (Service Locator anti-pattern heuristic).

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Keep single `using` keyword (status quo from EDR-037) | Rejected — preserves ambiguity that an LLM-native language should not replicate. The declaration/provision distinction is cognitively real and worth making syntactically visible. |
| Auto-naming from type (`require Database` → `db`) | Rejected per §3.4 — ambiguous with same-type dependencies; LLMs need explicit name resolution. Deferred as post-v0.1 sugar. |
| Module-level `given` only (no per-instance) | Rejected — per-instance resolution is the real gap identified in EDR-037's deferred spec. Module-level `given` switching is too coarse for test/prod differentiation at instance granularity. |
| Traits can carry `require` | Rejected — a trait declaring `require` would couple dependency concern to behavioural contract. Dependency slots are an implementation concern (EDR-078 compatible). |
| No class-level slots; use constructor parameters + fields manually | Rejected — loses compile-time initialization guarantee. The invariant (slot must be filled) is the semantic addition. |

---

### Gate Validation

All 7 gates applied per `DECISION_VALIDATION.md` § Gate Selection (semantic refinement of an existing concept — full gate set applied for completeness given the new invariant).

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass (Flag) | One flag: cognitive cost of a new keyword (`require`). Outweighed by benefit of disambiguation for both humans and LLMs. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | All terms defined; resolution priority explicit (`using` > local `given` > imported `given`); no self-referential paradoxes. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass (Flag) | Two flags: new keyword justified (eliminates overload); initialization invariant is a compiler check, not a primitive. Both within acceptable threshold. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Operates at Level 2 (Language Pattern); no layer violation; EDR-078 compliance confirmed. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | "Slot must be filled before use" is strategy-independent. All three defined strategies can implement. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | `require`/`using` follows natural language metaphor (need/provide); auto-naming from type deferrable as sugar; removable without ecosystem breakage. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | `require (Type name)*` is a regular grammar fragment; reduces ambiguity vs. overloaded `using`; missing slot = compile error. |

**Gates not applied:** None.

**Detailed reasoning:** See pipeline pass output for per-gate reasoning trail with criterion-level breakdown.

---

### Amendment (2026-08-16): Creation-time Resolution — Closure Capture Unification

**Post-acceptance finding** (from the CLOSURE_CAPTURE hypothesis review). The Dual Parameter Model gains a **creation-time resolution point** for dependency slots, unifying closure capture with environment injection.

#### 1. Creation-time resolution for function literals

A function literal `fn (x) using n -> ...` resolves its `using` slot **at creation**, binding the captured value into the resulting function value. A closure is **not a separate concept** — it is a self-contained first-class function value whose `using` slot was resolved at creation. This is consistent with the accepted primitive decomposition (`PRIMITIVE_BLOCKS.md` § 3.2.2, EDR-016): closures decompose to `function` + `reference` + `scope`.

The resolution point is determined by syntactic position:
- `using n` inside a **function literal** → creation-time binding (capture; the value is self-contained).
- `using v` at a **call / construction site** → call-time provision (as originally specified).
- `require` in a **signature** → declaration (as originally specified).

#### 2. Unified resolution priority

The same four-tier priority applies to `using` slots at **both** resolution points:

| Tier | Creation-time (literal) | Call-time / construction |
|------|-------------------------|--------------------------|
| 1 — Explicit | `n` lexically bound in the enclosing scope at creation → capture that binding | Explicit `using v` at the call / construction site |
| 2 — Local `given` | Local `given n` → bind at creation | Local `given` |
| 3 — Imported `given` | Imported `given n` | Imported `given` |
| 4 — Missing | Compile error — slot must be filled | Compile error — slot must be filled |

**Rationale:** one priority rule across both resolution points keeps the mental model uniform and supports testability — closures fall back to `given` defaults, explicit capture is achieved via a lexical binding, and per-instance overrides (EDR-081) remain unchanged.

#### 3. Superseded decision

This amendment **supersedes** the provisional `capture` keyword for closure capture (`CLOSURE_CAPTURE.md`, 2026-08-07). The unification direction is accepted; `using` remains the single injection keyword.

#### 4. Deferred

- **Mutable vs by-value capture** — separate decision.
- **Exact syntax** (keyword position, grammar, fallback edge cases) — deferred to `SYNTAX_PIPELINE` (Phase 5).
