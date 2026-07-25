# Prototype × Trait — сравнение и роль в Orthon

> **Заметка по материалам:** [`PROTOTYPE.md`](../how/concepts/research/PROTOTYPE.md),
> [`TRAITS.md`](../how/concepts/research/TRAITS.md).
>
> Создана: 2026-07-25

## Их роли — принципиально разные

| | **Trait** | **Prototype** |
|---|---|---|
| Статус | Первичная концепция языка | Не primary — возможна как библиотечный паттерн |
| Время связывания | Компиляция (статическая диспетчеризация) | Runtime (живая цепочка делегирования) |
| Отношение к данным | Поведение отделено от данных | Объект = данные + ссылка на прототип |
| Типизация | Статическая, проверяемая | Динамическая — трудно проверить статически |
| Производительность | Предсказуемая (monomorphisation / vtable) | Непредсказуемая (inline cache, hidden classes) |

Из `PROTOTYPE.md` (Implications, #1):

> Orthon's primary model should be class/trait-based. Prototypes introduce live
> delegation chains that conflict with static type checking, memory layout
> predictability, and compilation to native code.

Из `TRAITS.md` (Principles, #1):

> Traits define only behaviour, never data fields. Data belongs to types.

---

## 1. Trait — как это работает

Это **первичный механизм полиморфизма** в Orthon:

```orthon
// Объявление trait — контракт поведения
trait Printable
    fn format(self) -> String

// Реализация для конкретного типа
impl Printable for User
    fn format(self) -> String
        return "User({self.name})"

// Generic-функция со статической диспетчеризацией (по умолчанию)
fn print_all(items: [T]) where T: Printable
    for item in items
        print(item.format())

// Динамическая диспетчеризация — opt-in через dyn
fn process_dyn(item: dyn Processor)
    item.process()
```

```orthon
// Associated types
trait Collection
    type Item
    fn get(self, index: Int) -> Option<Self::Item>
    fn len(self) -> Int

// Default implementation
trait Stringifiable
    fn to_string(self) -> String
        return "<opaque>"

impl Stringifiable for Int
    // Используется default — можно переопределить при желании
```

---

## 2. Prototype — как это НЕ планируется (объяснение почему)

Из `PROTOTYPE.md` (Implications, #2–#4):

> A prototype-style delegation mechanism could be provided as a standard library
> abstraction — e.g., a `Delegator<T>` type that forwards method calls to a
> dynamically-assigned delegate object.

Prototype **не будет синтаксической конструкцией языка**. Вот почему:

```orthon
// ❌ НЕ будет — неявный обход цепочки прототипов (как в JS)
//    Это конфликтует с explicitness и статической типизацией.

let animal = proto {
    fn speak(self) -> String
        return "{self.name} makes a sound."
}

let dog = animal.clone()
dog.name = "Rex"
dog.speak()  // Неявная делегация → недопустимо
```

Проблемы этого подхода в контексте Orthon:

- `dog.speak()` — не видно, какой объект реально предоставляет метод
- `animal.speak` можно изменить в runtime → ломает любую статическую проверку
- `dog` не имеет предсказуемого layout в памяти
- Непонятно, кто владеет prototype (ownership model)

---

## 3. Композиция: как одно и то же выражается через Trait + явная делегация

Идиоматичный Orthon использует **композицию + traits** вместо prototype chain:

```orthon
// ✅ Как Orthon выражает то же самое — явно и статически

// 1. Определяем поведение через trait
trait Speaker
    fn speak(self) -> String

// 2. Определяем тип c именем
struct Animal
    name: String

// 3. Реализуем trait для типа
impl Speaker for Animal
    fn speak(self) -> String
        return "{self.name} makes a sound."

// 4. Композиция вместо цепочки прототипов
struct Dog
    animal: Animal        // Явное поле-делегат
    breed: String

impl Speaker for Dog
    fn speak(self) -> String
        return self.animal.speak()  // Явная, а не неявная делегация
```

**Ключевое отличие:** `self.animal.speak()` — видно, что мы делегируем, кому и
какой метод. Нет магии.

---

## 4. Библиотечный паттерн `Delegator<T>` (если prototype всё же нужен)

Из `PROTOTYPE.md` (Implications, #2):

```orthon
// ✅ Возможная stdlib-абстракция — явный Delegator, не синтаксис языка

// Делегатор — generic-тип, который хранит ссылку на delegate
struct Delegator<T>
    delegate: T

impl<T> Delegator<T>
    // Явный метод lookup, а не неявный обход цепочки
    fn lookup(self, key: String) -> Option<fn>
        // Логика поиска метода у delegate
        ...

    fn set_delegate(self, new: T)
        self.delegate = new

// Использование — явное:
let animal = Animal { name: "Generic" }
let dog = Delegator<Animal> { delegate: animal }
dog.delegate.speak()  // Явный доступ к делегату
```

То есть если вы **действительно** хотите динамическую замену поведения в
runtime — вы можете это сделать через библиотеку. Но это будет **явно**, не на
уровне синтаксиса языка, и не будет влиять на систему типов.

---

## 5. Сравнение — один и тот же сценарий тремя способами

**Сценарий:** есть «животное, которое может говорить», и «собака, которая
делегирует `speak` животному, но добавляет своё».

### JavaScript (prototype — неявно)

```javascript
const animal = { speak() { return "makes sound"; } };
const dog = Object.create(animal);
dog.bark = function() { return this.speak() + " Woof!"; };
// Не видно, откуда берётся speak()
```

### Rust (trait — статически, явно)

```rust
trait Speaker { fn speak(&self) -> String; }
struct Animal;
impl Speaker for Animal {
    fn speak(&self) -> String { "makes sound".into() }
}
struct Dog { animal: Animal }
impl Speaker for Dog {
    fn speak(&self) -> String { self.animal.speak() }
}
```

### Orthon (trait + композиция — явная делегация, как спроектировано)

```orthon
trait Speaker
    fn speak(self) -> String

struct Animal
    name: String

impl Speaker for Animal
    fn speak(self) -> String
        return "{self.name} makes a sound."

struct Dog
    animal: Animal

impl Speaker for Dog
    fn speak(self) -> String
        return self.animal.speak() + " Woof!"

fn main()
    let rex = Dog { animal: Animal { name: "Rex" } }
    print(rex.speak())  // "Rex makes a sound. Woof!"
    // ✅ Полностью статически типизировано
    // ✅ Явная, а не неявная делегация
    // ✅ Никакого runtime-обхода цепочки
```

---

## Резюме

| | **Trait** | **Prototype** |
|---|---|---|
| Статус в Orthon | ✅ **Первичная концепция** | ❌ **Не primary — библиотечный паттерн** |
| Когда использовать | Всегда для полиморфизма | Когда реально нужна runtime-замена поведения |
| Механизм | `trait` + `impl` + `where` bounds | `Delegator<T>` из stdlib |
| Диспетчеризация | static dispatch по умолчанию, `dyn` opt-in | Всегда dynamic — через lookup |
| Безопасность | Полностью статически проверяемо | Ответственность программиста |

**Главный вывод дизайна:** Prototype-делегация не отменяется полностью — она
может существовать как библиотечная абстракция поверх trait-системы. Но она
никогда не будет **синтаксической конструкцией с неявным обходом цепочки** (как
в JavaScript). Это сознательный выбор в пользу explicitness, статической
типизации и предсказуемой производительности.
