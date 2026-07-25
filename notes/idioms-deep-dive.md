# Идиомы ЯП — система типов, память, метапрограммирование

> Продолжение [[idioms-overview|idioms-overview]]. Детальный разбор трёх фундаментальных категорий.

---

## Система типов

| Категория | Java | Python | C# | Go | Clojure | Kotlin | Scala | Rust | Zig |
|-----------|------|--------|-----|----|---------|--------|-------|------|-----|
| **Типизация** | статическая, номинальная | **динамическая**, утиная | статическая, номинальная + частично структурная | статическая, номинальная | **динамическая** | статическая, номинальная + структурная (inline) | статическая, **номинальная + структурная** | статическая, **номинальная** (traits — структурная) | статическая, **структурная** (comptime duck typing) |
| **Вывод типов** | локальный (`var`, Java 10+) | — (динамическая) | локальный (`var`, C# 3+) | локальный (`:=`) | — (динамическая) | **полный** (всегда, кроме сигнатур) | **полный** | **полный** | локальный (`const`/`var`) + comptime |
| **Типы nullable** | **нет** (всё nullable кроме primitives) | всё nullable | nullable ref types (opt-in, C# 8+) | `nil` — built-in | всё nullable | **nullable / non-nullable** — обязательно выбирать | `Option[T]` — нет null | **нет null** — `Option<T>` | `?T` — optional type |
| **Tuples / product types** | `record` (14+), `Pair`/`Triple` | `tuple` — **синтакс** `(a, b)` | `ValueTuple` — **синтакс** `(a, b)` (C# 7+) | — (нет) | `[a b]` vector | `Pair`, `Triple`, `data class` | `Tuple`, `case class` | **tuple** — синтакс `(a, b)` | tuple через struct (явно) |
| **Sum types / ADT** | `enum` class (5+) | `enum` (3.4+) | `enum` (нет sum types) | `iota` для констант | `defmulti`/`defmethod` | `sealed class` + `enum class` | `sealed trait` + `enum` | **`enum` — настоящие sum types** | `union` + tagged union via comptime |
| **Struct / class** | class — **всё класс** | class (объектная модель) | class + `struct` (value type) | `struct` — нет классов | — (только data structures) | class + `data class` + `value class` | class + `case class` + trait | `struct` — нет классов | `struct` — нет классов |
| **Generics / templates** | generics (erasure, Java 5+) | — (duck typing) | **generics (reified — .NET runtime)** | generics (Go 1.18+, interface constraints) | — (multimethods) | generics (reified via inline + Star) | generics (higher-kinded via type members) | **generics** (traits + associated types) | **comptime** — нет generics в runtime |
| **Перегрузка операторов** | **нет** (только `+` для String) | **да** — `__add__`, `__eq__` и др. | **да** — `+`, `==` можно | **нет** | **да** — через метапрограммирование | **да** — `operator fun plus()` | **да** — через `implicit`/`given` | **да** — через `trait Add` | **нет** — можно через comptime |
| **Индексация / slice** | `array[i]` (только массивы) | `list[i]`, `dict[key]`, `slice[a:b]` — **универсально** | `list[i]` (через indexer) | `slice[i:j]` — **built-in** | `(nth coll i)`, `(get coll i)` — функции | `list[i]` (через `get`) | `list(i)` (apply pattern) | `slice[i]` (через `Index` trait) | `slice[i..j]` — built-in |

### Эволюция систем типов

```
Dynamic            ┌─────────────────────┐
                   │  Python, Clojure    │
                   └─────────────────────┘

Nominal + GC       ┌─────────────────────┐
                   │  Java, C#, Go       │
                   └─────────────────────┘

Nominal + FP       ┌─────────────────────┐
                   │  Scala, Kotlin      │
                   └─────────────────────┘

Traits / Struct    ┌─────────────────────┐
                   │  Rust (traits)      │
                   └─────────────────────┘

Structural/Comptime┌─────────────────────┐
                   │  Zig (comptime)     │
                   └─────────────────────┘
```

---

## Объекты vs Примитивы

| Модель | Языки | Что происходит с `int x = 5` |
|--------|-------|------------------------------|
| **Чистые примитивы** | Go, Rust, Zig | `x` — это 4/8 байт на стеке, нет объекта |
| **Всё объекты** | Python, Clojure, Scala (AnyVal) | `x` — выделенный объект в heap (Python) или оптимизированный value class (Scala) |
| **Dual model** (примитивы + объекты) | Java, C#, Kotlin | `int` = стек; `Integer` = heap; автоboxing/unboxing |

| Категория | Java | Python | C# | Go | Clojure | Kotlin | Scala | Rust | Zig |
|-----------|------|--------|-----|----|---------|--------|-------|------|-----|
| **Есть примитивы?** | **да** — `int`, `boolean`, `char` | **всё объекты** | **да** — `int`, `bool`, `double` | **да** — `int`, `bool`, `float64` | **всё объекты** | **да** — `Int`, `Boolean` (boxing transparent) | **всё объекты** (AnyVal — без boxing) | **да** — `i32`, `u64`, `bool`, `f64` | **да** — `u8`, `i32`, `f64`, `bool` |

---

## Управление памятью

| Категория | Java | Python | C# | Go | Clojure | Kotlin | Scala | Rust | Zig |
|-----------|------|--------|-----|----|---------|--------|-------|------|-----|
| **Модель памяти** | GC (JVM, generational) | GC (CPython, refcount + cycle detector) | GC (.NET, generational) | **GC** (concurrent tri-color) | GC (JVM) | GC (JVM/native) | GC (JVM/native) | **Ownership + Borrowing (no GC)** | **Manual + Arena + comptime allocator (no GC)** |
| **Аллокатор** | JVM heap | CPython heap | .NET managed heap | Go heap | JVM heap | JVM/native heap | JVM/native heap | **Allocator parameter** (явный, per-value) | **Allocator argument** — явно везде: arena, gpa, c_allocator |
| **Value vs Reference** | объекты — ref; primitives — value | всё — ref (объекты) | class — ref; struct — value | всё — value (map, slice — fat pointer) | всё — ref (persistent, structural sharing) | class — ref; `value class` — value | class — ref; `value class` — value | **value by default** (move), `Box` — heap, `Rc`/`Arc` — shared | всё — value, structs могут быть heap через allocator |
| **Move vs Copy** | GC — нет move | GC — нет | GC — нет | GC — нет | GC — нет | GC — нет | GC — нет | **move by default**; `Copy` — явно | **move by default**; `dupe` — явно |
| **RAII / destructor** | **нет** (GC; `AutoCloseable`) | **нет** (GC; `with`/context managers) | **нет** (GC; `IDisposable`) | `defer file.Close()` | — | `.use { }` (`Closeable`) | `using` (2.13+) | **да** — `Drop` trait, гарантировано | **да** — `defer` + `errdefer` (явно) |

### Спектр управления памятью

```
GC (удобство)               ────────►     No GC (контроль)
        ↑                                    ↑
    Python, Java, C#, Go, Clojure           Rust, Zig
                                           (ownership / manual)
```

### Rust vs Zig — подходы к памяти

| | Rust | Zig |
|---|---|---|
| **Механизм** | Ownership + Borrowing (проверяется компилятором) | Явный allocator parameter (никаких проверок borrow checker) |
| **Идиома** | «Компилятор доказывает корректность памяти» | «Программист явно выбирает allocator для каждой аллокации» |
| **Ошибки** | compile-time borrow checker | runtime — use-after-free возможны, но minimised |
| **Allocator** | `Box::new()`, `alloc::Global`, per-crate alloc | каждый контейнер принимает allocator аргументом |

---

## Метапрограммирование

| Категория | Java | Python | C# | Go | Clojure | Kotlin | Scala | Rust | Zig |
|-----------|------|--------|-----|----|---------|--------|-------|------|-----|
| **Уровень** | аннотации + reflection (runtime) | декораторы + metaclasses (runtime) | **атрибуты + source generators** (compile-time) + reflection | `go generate` (внешняя утилита) + `reflect` | **macros** (compile-time, homoiconic) | аннотации + KSP (compile-time) | macros (Scala 3, inline, `$`) | **macros** (proc + declarative `macro_rules!`) | **comptime** (исполнение кода в компиляторе) |
| **Code generation** | annotation processors, Lombok | — (runtime) | **source generators** (Roslyn, C# 9+) | `go:generate` — внешние команды | **macros** (`defmacro` — AST) | **KSP**, compiler plugins | `inline` + `macro` (3), `quotes`/`splice` | **proc macros** — derive + attr + fn-like | `@as`, `@compileLog`, `@field`, `@bitCast`, comptime fn |
| **Reflection** | `java.lang.reflect` — полная, runtime | `type()`, `getattr` — полная, runtime | `System.Reflection` — полная, runtime | `reflect` — ограниченная, runtime | — (macros) | `::class`, `memberProperties` | `classOf`, `getClass` | **minimal** — `std::any::TypeId` | **none** (всё через comptime) |
| **Аннотации** | `@Override`, `@Deprecated` — интеграция в компилятор | **декораторы** `@staticmethod`, `@property` — runtime | `[Serializable]`, `[Obsolete]` — compile + runtime | — (нет) | — (macros заменяют) | `@JvmStatic`, `@OptIn` | `@tailrec`, `@inline` | `#[derive(Debug)]` — макрос | `test`, `inline` — built-in |

### Пять архитектур метапрограммирования

| Подход | Языки | Время выполнения |
|--------|-------|-----------------|
| **Reflection + runtime** | Java, Python, C# | runtime |
| **Source generation** | C# (source generators), Kotlin (KSP) | compile-time (до компиляции) |
| **Homoiconic macros** | Clojure, Scala 3 | compile-time (AST rewriting) |
| **Procedural macros** | Rust (proc macros) | compile-time (TokenStream → TokenStream) |
| **Comptime** | Zig | compile-time (полное выполнение кода в компиляторе) |

---

## Связанные заметки

- [[idioms-overview|idioms-overview]] — основная таблица идиом
- [[criteria-based-algorithm-selection]] — другой подход к выбору между языками
