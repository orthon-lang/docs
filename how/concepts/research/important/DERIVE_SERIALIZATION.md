# Derive Serialization

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-serialization.md`

## Issue (Why)

Manually converting objects to/from JSON requires recursive code that
misses fields, mishandles cyclic references, and breaks type safety.
Every `to_dict()` / `to_json()` written by hand is a potential source of
deserialization bugs and type errors.

Orthon should eliminate manual serialization code by providing automatic
serialization/deserialization for value types through trait derivation.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Manual recursive `to_dict()` | `dataclasses` + `dataclass_json` / `pydantic` |
| Java | Legacy `JSONObject` from third-party libs | Jackson / Gson + `record` with annotations |

## Hypothesis

Orthon can eliminate manual serialization code by providing automatic
serialization/deserialization for value types (records/structs) through
trait derivation — where the compiler generates `to_json`, `from_json`,
and equivalent binary-format functions based on declarative annotations,
with type validation at parse time rather than runtime.

## Implications for Orthon

- Serialization must be automatic for value types (records / data
  classes) — no manual `to_dict()` / `to_json()`.
- Explicit annotations for customization (field renaming, field
  skipping) — declaratively, without boilerplate.
- Deserialization with type validation produces errors at parse time.
- Support not only JSON but also binary formats through a common
  trait/protocol.
- Cyclic references are a solvable problem (reference tracking), not a
  dead end.
- Related concepts: `essential/TRAITS.md`, `essential/DATA_MODEL.md`

## Open Questions

- Should serialization be a trait or a built-in compiler feature?
- How to handle serialization of types with private fields?
- Is JSON the default format, or is it format-agnostic?

## Decision History

Initial hypothesis derived from `imperative-crutch-serialization.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/TRAITS.md`
- [ ] `essential/DATA_MODEL.md`
- [ ] `what/GLOSSARY.md`
