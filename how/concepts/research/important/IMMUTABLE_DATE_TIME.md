# Immutable Date and Time

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-datetime.md`

## Issue (Why)

Legacy date/time APIs are mutable, thread-unsafe, have surprising
behaviour, and provoke errors. Java's `Date`/`Calendar` debacle is the
canonical example — mutable objects where calling `addDays(1)` mutates
the original, and non-thread-safe `SimpleDateFormat` causes data
corruption under concurrency.

Orthon should eliminate date/time-related bugs by providing a set of
immutable, thread-safe date/time types as distinct types.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Java | `new Date()` (mutable, deprecated) | `LocalDate.now()`, `ZonedDateTime` |
| Java | `SimpleDateFormat` (thread-unsafe, mutable) | `DateTimeFormatter` (immutable, thread-safe) |
| Python | `datetime.datetime.now()` — generally fine | — |

## Hypothesis

Orthon can eliminate date/time-related bugs by providing immutable,
thread-safe date/time types (`Instant`, `Date`, `DateTime`,
`ZonedDateTime`) as distinct types in the standard library — not a
single mutable God object — with immutable formatters and
`Result<T>`-returning parsing.

## Implications for Orthon

- Date/time types must be immutable by default.
- Thread-safety is not an option — it is a mandatory property.
- Separate "instant" (Instant), "date without time" (LocalDate),
  "date with time and zone" (ZonedDateTime) into distinct types.
- Formatters are immutable objects, not mutable singletons.
- Parsing should return `Result<T>` / `Optional<T>`, not throw.
- Related concepts: `essential/DATA_MODEL.md`, `essential/ERROR_HANDLING.md`

## Open Questions

- Should date/time math use operator overloading (`+`, `-`) or named
  methods?
- What level of calendar system support beyond Gregorian?

## Decision History

Initial hypothesis derived from `imperative-crutch-datetime.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/DATA_MODEL.md`
- [ ] `essential/ERROR_HANDLING.md`
- [ ] `what/GLOSSARY.md`
