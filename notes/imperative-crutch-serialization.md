# Imperative Crutch: Serialization / Deserialization

## Problem

Manually converting objects to/from JSON requires recursive code that
misses fields, mishandles cyclic references, and breaks type safety.

## Examples

### Converting an object to JSON

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | Manual recursive `to_dict()` | `dataclasses` + `dataclass_json` / `pydantic` |
| Java | Legacy `JSONObject` from third-party libs, manual population | Jackson / Gson + `record` (Java 16+) with annotations |

## Implications for Orthon

- Serialization must be automatic for value types (records / data
  classes) — no manual `to_dict()` / `to_json()`.
- Explicit annotations for customization (field renaming, field
  skipping) — declaratively, without boilerplate.
- Deserialization with type validation produces errors at parse time, not
  `ClassCastException` later.
- Support not only JSON but also binary formats (MessagePack, custom
  binary) — through a common trait/protocol.
- Cyclic references are a solvable problem (reference tracking), not a
  dead end.

**See also:** concepts `DATA_MODEL.md`, `METAOBJECTS.md`, `LLM_NATIVE_TOOLCHAIN.md`
