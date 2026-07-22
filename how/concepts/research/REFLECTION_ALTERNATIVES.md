# Reflection Alternatives

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How does code inspect, invoke, and generate types and members dynamically — without relying on runtime reflection?

Reflection (Java's `java.lang.reflect`, C#'s `System.Reflection`, Python's `getattr`/`type()`) provides:

- **Introspection** — querying the structure of types (fields, methods, annotations) at runtime.
- **Dynamic invocation** — calling methods and accessing fields by name determined at runtime.
- **Code generation** — creating new types or methods at runtime.

However, reflection has well-known costs:

1. **Runtime performance** — Reflective calls are significantly slower than direct calls.
2. **No compile-time safety** — Misspelled method names throw `NoSuchMethodError` at runtime.
3. **Security bypass** — Reflection can access private members, breaking encapsulation.
4. **Static analysis blindness** — Reflective usage is invisible to the compiler, linters, and IDEs.
5. **LLM unpredictability** — Reflection patterns are hard for LLMs to generate correctly because runtime type flow is not statically visible.

Many modern languages provide **compile-time alternatives** that achieve the same goals without runtime overhead:

| Goal | Reflection approach | Compile-time alternative |
|---|---|---|
| Serialization | Inspect fields at runtime via `getFields()` | Auto-derive serialization via traits/metaprogramming |
| Dependency injection | Scan classes at startup via reflection | Compile-time DI code generation |
| Logging | Inspect method signatures at runtime | Compile-time aspect weaving / macros |
| Data transfer | Map objects by field name at runtime | Schema-based code generation |
| Dynamic dispatch | Call methods by string name | Sealed types + pattern matching |

The core problem: **should Orthon have runtime reflection at all, or should all metaprogramming needs be served by compile-time mechanisms?**

## Examples

### Compile-time alternatives by language

| Feature | Java (reflection) | Kotlin (mixed) | Rust (compile-time only) |
|---|---|---|---|
| **Type inspection** | `obj.getClass()` | `obj::class` + `KClass` | `std::any::TypeId` (limited) |
| **Field access** | `field.get(obj)` | `::class.memberProperties` | Traits + derive macros |
| **Method call** | `method.invoke(obj)` | `::class.functions` + `call` | Traits + generics |
| **Code gen** | `Proxy`, `ASM`, bytecode | `inline` + `reified` | Procedural macros (`syn`, `quote`) |
| **Serialization** | Jackson/Gson (reflection) | `kotlinx.serialization` (compiler plugin) | `serde` (derive macros) |
| **DI** | Spring (reflection scan) | Koin (lazy references), Kodein | `shaku` (compile-time) |
| **Type info in generics** | Erased | `reified` (inline only) | Monomorphised (full info) |

## Implications for Orthon

1. **Compile-time metaprogramming over runtime reflection** — Orthon should provide compile-time mechanisms for cross-cutting concerns. Runtime reflection, if present at all, should be an opt-in capability behind an import or language flag.

2. **`reified` generics in inline functions** — Inside inline (or equivalent) functions, generic type parameters are reified — the compiler substitutes the concrete type, enabling type inspection without reflection:

   ```
   inline fun typeName[T]() -> String:
       return T.name     // available because T is reified
   ```

3. **Derive macros / trait auto-implementation** — Common patterns (serialization, equality, hashing, cloning) are implemented via compile-time derivation, not runtime inspection:

   ```
   @derive(Serialize, Deserialize)
   struct Config:
       host: String
       port: Int
   ```

4. **Sealed types + pattern matching for dynamic dispatch** — Instead of calling methods by string name, use sealed types and exhaustive pattern matching (see `ALGEBRAIC_DATA_TYPES.md`, `PATTERN_MATCHING.md`).

5. **Schema-based code generation** — For truly dynamic scenarios (deserializing unknown JSON, database rows), use a schema-based approach: define the schema as a type, and the compiler generates the mapping code.

6. **No `getField("name")` pattern** — String-based field access is not permitted. Field access must always be statically verified.

## Open Questions

1. Should Orthon have **any** runtime reflection capability, or should it commit to a fully compile-time metaprogramming model like Rust?
2. If runtime reflection is included, should it be restricted to public API (no private member access)?
3. How do `reified` generics interact with the dispatch policy — can reified type parameters be used at non-inline call sites?
4. Should the compiler support a `reflect` module that exposes type metadata (like Kotlin's `KClass`) but only for explicitly annotated types (`@Reflectable`)?

## Decision History

Initial research — no decisions recorded yet.

---

### Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `GENERICS.md`
- [ ] `METAOBJECTS.md`
- [ ] `TRAITS.md`
- [ ] `PATTERN_MATCHING.md`
- [ ] `what/GLOSSARY.md`
- [ ] `IMPLEMENTATION_POLICIES.md`
