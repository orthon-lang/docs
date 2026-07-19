# Imperative Crutch: Date and Time

## Problem

Legacy date/time APIs are mutable, thread-unsafe, have surprising behavior,
and provoke errors.

## Examples

### 1. Getting the current date

| Language | Crutch | Modern |
|----------|--------|--------|
| Java | `new Date()` (deprecated, mutable) or `Calendar.getInstance()` | `LocalDate.now()`, `LocalDateTime.now()`, `ZonedDateTime` (Java 8+) |
| Python | `datetime.datetime.now()` — generally fine | — |

### 2. Parsing and formatting

| Language | Crutch | Modern |
|----------|--------|--------|
| Java | `SimpleDateFormat` (thread-unsafe, mutable) | `DateTimeFormatter` (immutable, thread-safe, Java 8+) |
| Python | `datetime.strptime` / `strftime` — generally fine | — |

## Implications for Orthon

- Date/time types must be immutable by default.
- Thread-safety is not an option — it is a mandatory property.
- Separate "instant" (Instant), "date without time" (LocalDate),
  "date with time and zone" (ZonedDateTime) into distinct types, not one
  God object.
- Formatters are immutable objects, not mutable singletons.
- Parsing should return `Result<T>` / `Optional<T>`, not throw a checked
  exception.

**See also:** concept `DATA_MODEL.md` (value types / records)
