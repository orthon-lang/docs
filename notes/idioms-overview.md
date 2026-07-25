# Идиомы языков программирования — обзор

> Сводная таблица: как 9 языков решают одни и те же задачи по-своему.
> Детальный разбор системы типов, памяти и метапрограммирования — в [[idioms-deep-dive|idioms-deep-dive]].

## Легенда: уровни идиом

Идиомы живут на разных слоях языка:

| Уровень | Что значит |
|---------|------------|
| **синтакс** | встроено в грамматику языка — доступно сразу, без импорта |
| **система типов** | проверяется компилятором на этапе типизации |
| **runtime** | поведение обеспечивается VM/runtime |
| **библиотека / stdlib** | идиома живёт в API, который надо импортировать |
| **comptime / макрос** | исполняется/раскрывается на этапе компиляции |

---

## Основные идиомы

| Категория | Java | Python | C# | Go | Clojure | Kotlin | Scala | Rust | Zig |
|-----------|------|--------|-----|----|---------|--------|-------|------|-----|
| **Уровень языка** | высокий (JVM) | высокий (interpreted) | высокий (.NET CLR) | средний (GC) | высокий (JVM/JS) | высокий (JVM/native) | высокий (JVM/native) | **низкий + безопасный** | **низкий** (no GC, no hidden control flow) |
| **Парадигма** | OOP → FP (8+) | multi: OOP+FP+proc | OOP-first + FP | procedural + structured | **FP-first** | multi: OOP+FP+structured | FP-first + OOP | **FP + structured** | procedural + structured + comptime FP |
|  |  |  |  |  |  |  |  |  |  |
| **Основные структуры** | массивы, `ArrayList`, `HashMap` | `list`, `dict`, `tuple`, `set` | `List<T>`, `Dictionary<,>` + LINQ | слайсы, `map` | `vector`, `map`, `set` (persistent) | `List`, `Map`, `Set` | `List`, `Map`, `Set` | `Vec`, `HashMap`, `BTreeMap` + slices | слайсы, `ArrayList`, `AutoHashMap` |
| **Уровень** | библиотека | **синтакс** | библиотека + синтакс | **синтакс** | **синтакс** | библиотека + синтакс | библиотека + синтакс | библиотека + синтакс (slices) | **синтакс** (slices) |
|  |  |  |  |  |  |  |  |  |  |
| **Циклы / итерации** | `for-each`, `Stream::map/filter` | **comprehensions / generators** | `foreach` + LINQ | `for range` — единственная форма | `for [x xs]`, `map/filter/reduce` | `for`, sequence operators | **`for-yield` comprehension** | `.iter().map().filter()` | `for` над слайсами, `inline for` |
| **Уровень** | библиотека | **синтакс** | синтакс + библиотека | **синтакс** | **синтакс** | библиотека | **синтакс** | библиотека (traits) | синтакс + comptime |
|  |  |  |  |  |  |  |  |  |  |
| **Null handling** | `Optional<T>` | `None` / EAFP | nullable ref types + `?.`/`??` | `nil` + `if err != nil` | `nil` (редко) | **Elvis `?:`**, `?.`, **non-nullable by default** | `Option[T]` | **`Option<T>` — нет null** | `?T` + `orelse` |
| **Уровень** | библиотека | библиотека | **синтакс + типы** (opt-in) | синтакс + runtime | runtime | **система типов** | **система типов** | **система типов** | **система типов** |
|  |  |  |  |  |  |  |  |  |  |
| **Error handling** | checked/unchecked exceptions | exceptions + try/except | exceptions + try-catch | **error as value** (`if err != nil`) | exceptions (редко) | `Result<T,E>` + `runCatching` | `Try[T]`, `Either[L,R]` | **`Result<T,E>`**, `?`, `match` | **error union `E!T`**, `try`/`errdefer` |
| **Уровень** | синтакс + runtime | runtime | синтакс + runtime | **синтакс** | runtime | библиотека / runtime | **система типов** | **система типов** + синтакс | **синтакс + типы** (built-in) |
|  |  |  |  |  |  |  |  |  |  |
| **Concurrency** | `CompletableFuture`, virtual threads (21) | `async/await`, asyncio | `async/await`, `Task<T>` | **goroutines `go f()` + channels** | core.async — channels | **coroutines `suspend`** + `Flow` | ZIO, Cats Effect | async/await (tokio) | `@asyncCall` (не идиома) |
| **Уровень** | библиотека → синтакс | библиотека | синтакс + библиотека | **синтакс + runtime** | библиотека | **синтакс** (suspend) | библиотека (external) | библиотека (external) | библиотека |
|  |  |  |  |  |  |  |  |  |  |
| **Pattern matching** | `switch` (17+, preview) | `match` (3.10+) | `switch` expr + `is` pattern | нет | `match` — core | **`when`** + destructuring + sealed | глубокое, extractors | **`match`** — exhaustive | `switch` + `inline` + comptime eval |
| **Уровень** | синтакс | синтакс | синтакс + типы | — | **синтакс** | **синтакс + типы** | **синтакс + типы** | **синтакс + типы** | синтакс + comptime |
|  |  |  |  |  |  |  |  |  |  |
| **Deferred cleanup** | try-with-resources | `with` + context managers | `using` | **`defer`** | — | `.use { }` | `using` | **RAII — `Drop`** (автоматически) | **`defer` + `errdefer`** |
| **Уровень** | синтакс | синтакс | синтакс | **синтакс** | — | библиотека | синтакс | **типы + компилятор** | **синтакс** |
|  |  |  |  |  |  |  |  |  |  |
| **Строковая интерполяция** | `String.format` / `STR.` (21+) | `f"..."` | `$"..."` | `fmt.Sprintf` | `(str ...)` | `"$x"` | `s"..."` / `f"..."` | `format!(...)` (макрос) | `std.fmt.comptimePrint` |
| **Уровень** | библиотека → синтакс | **синтакс** | **синтакс** | библиотека | библиотека | **синтакс** | **синтакс** | **макрос** | **comptime** |

---

## Ключевая закономерность

Чем **ниже** уровень языка (Rust, Zig) — тем больше идиом живёт в **системе типов и компиляторе**.
Чем **выше** (Python, Clojure) — тем больше в **синтаксисе**.
Go — исключение: при среднем уровне языка его ключевые идиомы встроены в **синтаксис и runtime**, а не в типы.

```
                   SYNTAX          TYPE SYSTEM      RUNTIME         LIBRARY/STD     COMPTIME/MACRO
                   ──────          ──────────       ───────         ────────────     ──────────────
Go                 for range,      —                goroutines,     fmt.Sprintf      —
                   defer,                            channels,
                   error return                      nil
                   ────────────────────────────────────────────────────────────────────────────────

Python             comprehensions, —                exceptions,     asyncio          —
                   f-strings,                       duck typing
                   with, match
                   ────────────────────────────────────────────────────────────────────────────────

Rust               ? propagation,  Option/Result,   —               tokio,           format! macro
                   for over        borrow checker,                  iterators
                   iterators       Drop (RAII)
                   ────────────────────────────────────────────────────────────────────────────────

Zig                try, defer,     optional types,   —              std.fmt,         comptime eval,
                   errdefer        error unions                                   inline for, @as
                   ────────────────────────────────────────────────────────────────────────────────

Kotlin             ?., ?:,         nullable types,   coroutines      .use { }        —
                   $x, when        inline classes    (suspend)
                   ────────────────────────────────────────────────────────────────────────────────

Clojure            [x xs],         —                 —               core.async,     macros
                   for, match,                                        persistent ds
                   reader macros
                   ────────────────────────────────────────────────────────────────────────────────

Scala              for-yield,      Option, Try,      —               ZIO, Cats       macros (Scala 3)
                   s"", match      given/using
                   ────────────────────────────────────────────────────────────────────────────────

Java               switch (17+),   Optional (API),   —               Streams,         —
                   try-with        records                             CompletableFuture
                   ────────────────────────────────────────────────────────────────────────────────

C#                 async/await,    nullable ref      —               LINQ (stdlib),  source generators
                   $"", using,     types                               TPL
                   foreach
```
