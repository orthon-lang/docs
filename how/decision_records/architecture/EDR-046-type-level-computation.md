# EDR-046: Type-Level Computation — Closed Set of Compiler Intrinsics

**Status:** Accepted

**Date:** 2026-07-27

**Category:** Architecture

**Scope:** Subsystem (Type System)

---

### Context

TypeScript demonstrates the power of a type-level computation language — conditional types, mapped types, template literal types, `keyof`, `typeof` (in type position), Indexed Access Types, `infer`, and Utility Types built from them — enabling operations like `Partial<T>`, `Pick<T, K>`, and `Omit<T, K>` that derive new types from existing ones.

However, TypeScript's type-level language is widely documented as Turing-complete — community type-level programs have implemented string calculators, JSON parsers, and Game of Life simulations inside the type checker. This directly conflicts with Orthon's stated minimal-core and LLM-generability goals.

The core question: does Orthon want:
1. A Turing-complete (or highly expressive) compile-time type-computation language?
2. A closed set of built-in type-level utilities (Partial/Pick/Omit-equivalents as compiler intrinsics with no user-defined type-level functions)?
3. No type-level computation at all beyond ordinary generics?

The research document at `how/concepts/research/important/TYPE_LEVEL_COMPUTATION.md` explores this in depth, noting that a derive-macro model (already present via EDR-029's `@derive`) may cover the same DTO-shaping ergonomics without introducing a second computational layer.

---

### Decision

**1. Orthon provides a closed set of built-in type-level computation intrinsics — NO user-extensible type-level programming language.** The set is:

| Intrinsic | Semantics | Example |
|-----------|-----------|---------|
| `KeyOf<T>` | Produces a union of literal property-name types of `T` | `KeyOf<User>` → `"id" | "name" | "email"` |
| `Pick<T, K>` | Produces a type with only keys `K` from `T` | `Pick<User, "id" | "name">` |
| `Omit<T, K>` | Produces a type with all keys except `K` from `T` | `Omit<User, "password">` |
| `Partial<T>` | Produces a type where all keys of `T` are optional | `Partial<User>` |
| `Required<T>` | Produces a type where all keys of `T` are required | `Required<PartialUser>` |
| `Record<K, V>` | Produces a type with keys `K` and values `V` | `Record<"a" | "b", Int>` |
| `Readonly<T>` | Produces a type where all keys are read-only | `Readonly<User>` |
| `ElementOf<T>` | Produces the element type of a collection type `T` | `ElementOf<List<Int>>` → `Int` |

**2. These intrinsics are NOT user-extensible.** No `infer`, no conditional type expressions, no mapped type syntax. The compiler ships exactly the intrinsics listed above. Adding a new intrinsic requires an EDR.

**3. Intrinsics are evaluated entirely at compile time.** No runtime cost — the compiler transforms the type before code generation.

**4. Intrinsics are non-recursive.** No type-level recursion is permitted. This eliminates the Turing-completeness failure mode (compiler hangs, stack overflows) that TypeScript suffers from.

**5. The derive/macro mechanism (EDR-029) is the recommended alternative for any type-level operation not covered by the intrinsic set.** If a programmer needs custom type derivation, they write a `@macro` function that operates at compile time. This keeps the type-level language closed while allowing metaprogramming through a separate, explicit mechanism.

```orthon
# Using compiler intrinsics
type CreateUserPayload = Omit<User, "id" | "created_at">

# Using derive macro for struct-based derivation
@derive(From<User>)  # macro generates the conversion
type CreateUserRequest(name: String, email: String)
```

---

### Consequences

- **Positive:**
  - Eliminates Turing-completeness failure mode (no type-level recursion).
  - Keeps the type system LLM-generable — closed set of intrinsics with fixed, documented semantics.
  - Covers the essential use cases (DTO derivation, property-key extraction) that motivate type-level computation in TypeScript.
  - Derive/macro mechanism (EDR-029) provides an escape hatch for custom type-level operations without opening the type system to user-extensible computation.
  - Non-recursive intrinsics guarantee compiler termination.

- **Negative:**
  - Less expressive than TypeScript's type-level language — no user-defined conditional types, no `infer`, no mapped types.
  - New intrinsics require EDR process — adding a type-level operation is not something users can do.
  - Some legitimate use cases (e.g., excluding specific string patterns from a union) require a macro instead of a type-level construct.

---

### Compliance

1. Each intrinsic must have a fixed, documented semantic definition with no hidden behaviour.
2. Type-level recursion must be detected and rejected at compile time.
3. Intrinsics must compose freely — e.g., `Partial<Omit<User, "password">>` is valid.
4. The set of intrinsics is enumerable and documented — no mechanism exists for users to add new ones.
5. Macros (EDR-029) remain the mechanism for custom compile-time type derivation beyond the intrinsic set.

---

### Alternatives Considered

| Alternative | Rationale for Rejection |
|-------------|-------------------------|
| Full TypeScript-style type-level language (conditional types, mapped types, infer, user-extensible) | Direct tension with Minimal Core and LLM Generability. Turing-complete type systems produce compiler hangs and non-local reasoning. Rejected as contrary to Orthon's design philosophy. |
| No type-level computation at all — macros only | Acceptable, but macros are heavier machinery for simple type transformations (Partial, Pick, Omit). Compiler intrinsics provide better ergonomics for the most common DTO-shaping operations. |
| Recursive intrinsics with depth limit | A depth limit solves compiler hangs but not the reasoning complexity. Non-recursive intrinsics are simpler and eliminate the failure mode entirely. |
| Type-level computation via trait-associated types only (Rust style) | Less ergonomic for simple DTO transformations. Rust's `typenum` crate demonstrates the ceremony required. |
| Compile-time function execution (comptime) as the type-level computation mechanism (Zig style) | Comptime (EDR-031) is already available for general metaprogramming. The intrinsics in this EDR provide a lighter-weight mechanism for common type transformations without requiring comptime awareness. |

---

### Gate Validation

| Gate | Method | Verdict | Notes |
|------|--------|---------|-------|
| `USER_VALUE_GATE` | Working Backwards | Pass | Type-level intrinsics solve a real ergonomic problem — deriving DTO types from domain types without manual parallel type declarations. The closed set covers the essential use cases. |
| `LOGICAL_CONSISTENCY_GATE` | Socratic Method | Pass | Non-recursive, closed-set intrinsics have no self-referential paradoxes. Each intrinsic has a clear, fixed semantic definition. |
| `CONCEPTUAL_SIMPLICITY_GATE` | Scientific Method | Pass | A closed set of 8 documented intrinsics is the simplest possible type-level computation model. No conditional logic, no recursion, no user-extensible syntax. |
| `ARCHITECTURAL_INTEGRITY_GATE` | Logical Analysis | Pass | Intrinsics are a natural extension of the type system — no new architectural layer. Coexists with COMPILE_TIME_EXECUTION (EDR-031) for general metaprogramming and AST_MACROS (EDR-029) for custom derives. |
| `IMPLEMENTATION_INDEPENDENCE_GATE` | TRIZ | Pass | Type-level computation semantics are independent of any memory layout, allocation strategy, or execution model. |
| `LONG_TERM_MAINTAINABILITY_GATE` | Einstein's Method | Pass | Closed intrinsic set is maximally maintainable — no type-level code to version, deprecate, or debug. New intrinsics require explicit design review. |
| `LLM_GENERABILITY_GATE` | Empirical Analysis | Pass | Fixed set of documented intrinsics is the most LLM-generable model. No recursive type reasoning, no conditional type evaluation — the LLM simply looks up the intrinsic's semantics. |

**Detailed reasoning:** See `DECISION_LOG.md` entry for EDR-046 for per-gate reasoning trail.
