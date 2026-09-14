---
date: "2026-09-14 00:00"
promoted: false
source: "A Programming Paradigm for Spatiotemporal Composability (arXiv:2608.25512) — assessed via user summary, primary source not read"
---

## Orthon implements the spatial half of spatiotemporal composability — statically, not dynamically

### The comparison

The paradigm splits dynamic composition into two orthogonal halves:

- **Temporal composability** — a component's side effects must be fully
  revertible on unload (revertible effects; runtime LIFO undo stack).
- **Spatial composability** — a component declaratively declares what it
  provides and requires; the runtime tracks a service registry and
  activates/deactivates the component as its requirements are satisfied
  (reactive coeffects).

### Conclusion

Orthon implements **only the spatial half**, and it implements it in the
**static** domain rather than the dynamic one. The declarative half of
spatial composability is already present at all three dependency levels —
but it is checked once, by the compiler, with no runtime registry:

| Level | Orthon's static mechanism |
|---|---|
| Module | `use` dependencies + `effects:` header, compiler-verified — `what/concepts/CONTEXT_LIMITED_MODULES.md` (EDR-072) |
| Class | `require` slots filled at construction, per-instance — `what/concepts/REQUIRE_USING_DEPENDENCY_SLOTS.md` (EDR-081) |
| Method | `require` in signature, `using` at call site — EDR-037 |

These are declarative coeffects, but the resolution happens exactly once at
compile time. There is no service registry and no reactive re-evaluation,
which is the dynamics the paradigm adds.

**The temporal half does not apply to Orthon.** Orthon needs no runtime undo
stack because teardown is already guaranteed statically: deterministic
destruction, scope-based lifetime, declared effects (`what/EXECUTION_MODEL.md`,
`what/SEMANTIC_MODEL.md`). The paradigm's question — "how do we undo a
component's effects when it is unloaded?" — is answered by "the compiler
proves the lifetime and destroys deterministically", which holds for
statically-known lifetimes. Where lifetimes are *not* statically known, the
paradigm's problem has no counterpart in the language at all.

### Corollary — deliberate exclusion, not an unaddressed gap

Because both halves are handled statically, Orthon cannot express
*dynamically discovered* composition. The paradigm's real problem space —
hot reload, plugins, self-evolving agents — maps onto Orthon's deferred
research (`how/concepts/research/deferrable/INTERACTIVE_DEVELOPMENT.md`,
`SAFE_SANDBOX.md`), not onto the semantic core. `EXPLICIT_COMPOSITION.md`
explicitly rejects DI containers and service locators, so the dynamic half is
as much a deliberate exclusion as an unaddressed gap.

If dynamic composition were ever wanted from Orthon itself rather than from a
framework built on `execution_context` (EDR-085), revertible effects and
reactive coeffects would have to be pulled into the semantic model — a change
with consequences for the static lifetime/ownership model. Until that
decision is taken, the paradigm is a framework concern, not a language concern.
