# Immutable Date/Time

## Issue (Why)

Legacy date/time APIs are mutable, thread-unsafe, have surprising behaviour, and provoke errors. Orthon should eliminate date/time-related bugs by providing a set of immutable, thread-safe date/time types as distinct types.

## Principles

1. **Immutability** — Date/time types must be immutable by default.
2. **Thread safety** — Thread-safety is a mandatory property, not an option.
3. **Type distinction** — Separate "instant" (Instant), "date without time" (Date), "date with time and zone" (DateTime), "date with timezone" (ZonedDateTime) into distinct types.
4. **Immutable formatters** — Formatters are immutable objects, not mutable singletons.
5. **Result-based parsing** — Parsing should return `Result<T>` / `Option<T>`, not throw.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Value Type Policy | Determines that date/time types are value types with value semantics |

## Model (What)

The StdLib provides immutable, thread-safe date/time types:

```orthon
# Distinct types for distinct concepts
let now = Instant.now()                    # machine timestamp
let today = Date.from("2026-07-27")?       # date without time
let meeting = DateTime.from("2026-07-27T14:30:00")?  # date+time
let flight = ZonedDateTime.from("2026-07-27T14:30:00+02:00")?  # with zone
```

Operations return new values — no mutation:

```orthon
let tomorrow = today + Duration(days: 1)   # returns new Date
```

## Default Strategy

Immutable Date/Time is a **StdLib** concept — value-semantics date/time types provided by the standard library. The types are immutable value types with Result-based parsing and immutable formatters. No new language semantics.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Mutable date/time (legacy) | Objects mutated in place — thread-unsafe, error-prone. |
| Single God object | One class for all date/time concepts — leads to constructor overload explosion. |
| Third-party library | Date/time handled by ecosystem. No standardisation, inconsistent APIs. |

## Open Questions

1. Should date/time math use operator overloading (`+`, `-`) or named methods?
2. What level of calendar system support beyond Gregorian?
3. Should formatting be locale-aware in the core types or via extension functions?

## Decision History

- **EDR-068:** Immutable Date/Time accepted as StdLib — value-semantics date/time types. The date/time domain is well-understood and purely a library design problem. No new language semantics are required.
- **Classification per D-03:** StdLib. Date/time types are library types with value semantics. The concept introduces no new language-level constructs — only the standard library's collection of types.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
