# Terminology Analysis and Transition to Exclusive Declarations

## 1. How Well Does `mut` Fit Its Meaning?

`mut` (from *mutable*, *mutation*) is a short and widely accepted marker for mutability. In Rust it marks mutable bindings and references; in our project it marks the effect of a method.

**Pros:** recognizability, conciseness, direct association with mutation.

**Cons:** in Rust, `mut` is not a method effect but a property of bindings/references, which can be confusing. Moreover, `mut` is a modifier that must be combined with `fun` (`mut fun`), creating two orthogonal dimensions: "this is a function" and "it mutates." This overloads the syntax.

## 2. The `proc` Idea Instead of `mut` and Exclusive Declarations

`proc` (procedure) in many languages (Pascal, Nim, early Rust) denotes a block of code that performs actions, usually with side effects, and often does not return a value. In our model, a mutating method is precisely a procedure that changes an object's state.

If we introduce `proc` as a **standalone keyword for method declaration** rather than as a modifier, we get three mutually exclusive forms:

- **`proc`** — mutating operation (identity preserved). May return a value (like `pop`) or nothing.
- **`fun`** — pure read-only operation. Does not mutate `self`, always returns a value (void is permitted but meaningless).
- **`new`** — transforming operation (identity changed). Does not mutate `self`, always returns a new value.

These three keywords simultaneously declare the method and specify its effect. No combinations like `mut fun` or `new fun` are needed. The syntax becomes cleaner and more declarative.

## 3. What It Looks Like in Code

```
class List<T> {
    items: Array<T>

    // Read-only (fun) — always return a value, self unchanged
    fun len() -> Int { self.items.len() }
    fun get(idx: Int) -> T { self.items[idx] }

    // Mutating (proc) — may return a value or nothing
    proc append(item: T) { self.items.push(item) }
    proc sort() { self.items.sort() }
    proc pop() -> T {
        let val = self.items.last()
        self.items.removeLast()
        return val
    }

    // Transforming (new) — always return a new value
    new sorted() -> List<T> { List(self.items.sorted()) }
    new pop_child() -> (List<T>, T) {
        let new_items = self.items.withoutLast()
        return (List(new_items), self.items.last())
    }
}

fun demo() {
    let nums = List([1, 2, 3])

    // fun-calls (no restrictions)
    let size = nums.len()
    let first = nums.get(0)

    // proc-calls (require uniqueness)
    nums.append(4)
    nums.sort()
    let last = nums.pop()

    // new-calls (always allowed)
    let sorted = nums.sorted()
    let (new_list, elem) = nums.pop_child()
}
```

## 4. Advantages of Exclusive Declarations

- **No effect matrix**: no need to think about whether `mut` combines with `new` or `fun` with `mut`. Three orthogonal categories, each with its own keyword.
- **Less noise**: instead of `mut fun append(...)`, simply write `proc append(...)`. Lines are shorter and more expressive.
- **Semantic precision**:
  - `proc` clearly says: "this is an action, it mutates the object."
  - `fun` — "this is a pure computation over the object."
  - `new` — "this is a factory for a new value derived from the current one."
- **Simplified ABI**: each method has exactly one effect tag, no ambiguity.
- **Familiarity**: developers are familiar with `proc` (Pascal, Nim) and `fun` (ML, Kotlin). `new` for creating new objects is universally understood.

## 5. Possible Trade-offs

- **`proc` and return values**: in some languages, `proc` implies no return value. But we can clearly state that `proc` may return a value (like a function, but with mutation). This breaks tradition, but is pragmatic. The alternative — forbidding returns from `proc` and requiring `new`-version for `pop` returning a tuple — would force the programmer to create a new variable for simple element extraction, which can be inconvenient and wasteful. Better to allow `proc pop() -> T`.
- **`new` and void**: `new` must always return a value; otherwise it is meaningless. The compiler may require a non-void return type.
- **Strictness of separation**: a method that both mutates and returns a new value (like `pop` in its `proc` version) is no longer "only mutating" — it is mixed. But this is acceptable: `proc` covers any operation involving mutation, including returning partial data. If strict purity is needed, a fourth category `extract` could be added, but that is overkill initially.

## 6. Comparison with the Previous Model (`mut`, `new`, `fun`)

| Criterion | Previous (modifiers) | New (exclusive `proc`/`fun`/`new`) |
|-----------|----------------------|-------------------------------------|
| Keywords in declaration | 2 (`mut fun`) | 1 (`proc`) |
| Effect explicitness | Effect is an extra annotation | Effect is built into the declaration kind |
| Call uniformity | `.` always | `.` always |
| Learning curve | Must remember combinations | Three non-overlapping variants |

## 7. Recommendation

Use the exclusive declaration model: `proc` for mutating, `fun` for pure, `new` for transforming operations. This makes the language more declarative, removes "effect modifiers," and aligns the syntax with the tradition where procedures and functions differ at the declaration level.

## 8. Conclusion

`mut` as a term is adequate, but the combination `mut fun` is redundant. `proc` better conveys the meaning of a mutating action and allows a clean system of three non-overlapping method declaration forms. This improves readability, simplifies the compiler, and eliminates the "effect matrix" while retaining all memory safety guarantees without GC and without a borrow checker.
