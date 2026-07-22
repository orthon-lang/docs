# Metadata and Annotations

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22

## Issue (Why)

How does a language attach metadata — configuration, constraints, documentation, or behavioural modifiers — to declarations (types, fields, functions, parameters)?

Three broad approaches exist:

1. **Annotation systems (Java, C#, Python decorators, Kotlin)** — special syntax (Java's `@Override`, `@Deprecated`; C#'s `[Obsolete]`; Python's `@dataclass`) that attaches structured metadata to declarations. Annotations can carry parameters and can be processed at compile time (annotation processors in Java, source generators in C#) or at runtime (reflection). Powerful, but often complex: custom annotations require defining the annotation type, specifying retention policy, and writing a processor. The annotation processing pipeline is a separate compilation phase with its own rules.

2. **Struct tags (Go)** — string literals attached to struct fields via backtick syntax:

   ```go
   type User struct {
       Name string `json:"name" validate:"required,min=3"`
       Age  int    `json:"age"  validate:"gte=0,lte=150"`
   }
   ```

   Tags are raw string literals. They have no type safety — the tag content is parsed by library code (encoding/json, validation libraries). There is no compile-time processing of tags; they are accessed at runtime via reflection (`reflect.StructTag`). The approach is minimal (no new syntax beyond backtick strings), pragmatic, and trusted by convention rather than enforced by the compiler.

3. **No built-in metadata mechanism** — metadata is encoded through naming conventions, documentation comments, or external configuration files. Typical in older systems languages (C, early C++).

The core problem: **every annotation/attribute system adds complexity to the language** — new syntax, new processing rules, and new interactions with the type system. The question is whether the benefit (structured metadata at declaration sites) justifies this complexity, or whether simpler alternatives (struct tags, naming conventions) suffice.

For Orthon's LLM-native design, this is especially relevant: annotations can provide structured hints to the LLM Toolchain (e.g., `@generated` to mark auto-generated code, `@deprecated` to suppress generation of certain APIs).

## Principles

1. **Minimal mechanism** — Metadata should require the smallest possible addition to the language. If a naming convention or comment convention suffices, do not add syntax.

2. **No annotation processing pipeline** — If metadata is present, it should be parseable without a separate annotation processing compilation phase. A struct-tag approach (simple string values parsed by library code) is preferred over Java-style annotation processors.

3. **Reflection not required** — Metadata should be accessible without requiring full runtime reflection. If compile-time extraction is possible, it should be preferred.

4. **LLM-accessible** — Metadata should be exposed in the Schema Provider as structured data, so the LLM Toolchain can read annotations without parsing code.

5. **No special syntax for common cases** — Common metadata (JSON serialization hints, validation rules, deprecation markers) should not require defining a new annotation type.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Metadata Policy | Determines whether the language has built-in metadata syntax, struct tags, or relies on conventions |
| Reflection Policy | Governs whether metadata is accessible at runtime via reflection |
| Schema Policy | Controls how metadata is exposed in the Schema Provider for LLM consumption |
| Compile-Time Processing Policy | Governs whether metadata is processed at compile time (annotation processors) or left to library code |

## Model (What)

The lightweight metadata model (Go-inspired) attaches string-valued tags to declarations:

```orthon
type User {
    name: String  `json:"name" validate:"required"`
    age: Int      `json:"age"`
}
```

Tags are:
- Raw string literals attached to fields (and optionally to types, functions, and parameters)
- Parsed by library code, not by the compiler
- Accessible via a minimal reflection API (or schema query)
- Exposed in the Schema Provider as key-value pairs

**Deprecation** is handled as a language-level attribute rather than a tag:

```orthon
@deprecated("Use UserService instead")
type OldUser { ... }
```

## Default Strategy

**Struct tags** as the primary metadata mechanism (Go-inspired). A small set of language-defined attributes for built-in concerns (deprecation, experimental, generated). No annotation processor pipeline. Tags are exposed in the Schema Provider for LLM access.

## Alternative Strategies

1. **Full annotation system (Java-style)** — Annotation types, retention policies, compile-time processors. More powerful but significantly more complex. Justified only if compile-time code generation is a core language feature.

2. **Convention-based metadata (Python-style)** — Naming conventions (`_`, `__`) and documentation comments only. Zero language complexity, zero compiler enforcement.

3. **Attribute system (C#-style)** — Bracketed attributes attached to declarations. Similar to Java annotations but with simpler syntax and without separate annotation types for common cases.

## Open Questions

1. If Orthon uses struct tags, should tags be typed (parsed into structured data by the compiler) or remain raw strings (parsed by library code)?
2. Should the language support user-defined attributes/annotations, or are built-in markers (`@deprecated`, `@experimental`) sufficient?
3. How does metadata interact with the Schema Provider — should every tag be schema-visible, or only a subset?
4. Should metadata be available at runtime (reflection) or only at compile time (schema query)?

## Cross-References

- See `LLM_NATIVE_TOOLCHAIN.md` for how the Schema Provider exposes metadata
- See `VISIBILITY_AND_ENCAPSULATION.md` for how annotations interact with visibility
- See `SEMANTIC_ANNOTATIONS.md` for semantically meaningful annotations vs. configuration metadata
- See [`../../why/MANIFESTO.md`](../../why/MANIFESTO.md) § Minimal Core for the principle that favours simpler mechanisms
