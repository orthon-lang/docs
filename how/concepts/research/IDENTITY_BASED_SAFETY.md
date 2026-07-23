# Hypothesis: Identity-Based Memory Safety

## 1. Core Principles

- **No global variables.** Everything belongs to a local scope (function/block) or a module.
- **No `mut` keyword** in signatures or calls.
- The only mechanism for distinguishing operations is the `.` and `!` operators.
- Variables can be reassigned, but data mutation is governed by uniqueness rules inferred by the compiler.
- A single assignment operator `=`.

## 2. Identity Operators

- **`v.method(args)`** — **identity-preserved** operation.
  The object `v` remains the same.
  The method can be:
  - *pure* (read-only) — allowed on any object, even a shared one;
  - *mutating* (modifies `self` fields) — requires `v` to be the unique owner of the object.
- **`v!method(args)`** — **identity-changed** operation.
  Always creates a new object without modifying the original. The result must be explicitly captured via assignment.
  Uniqueness is not required, since the original object is not mutated.
- No other syntactic markers for mutation are used. The programmer sees only `.` and `!`.

## 3. Mutability Inference by the Compiler

- Method declarations contain **no `mut`**. The compiler analyzes the method body:
  - If the method assigns to a `self` field (or calls another mutating method on `self`), the method is considered mutating.
  - Otherwise, the method is considered pure (read-only).
- Methods with `!` in their name (e.g., `sort!`, `pop!`) are pure transformations that create a new object; they cannot mutate `self` and do not require uniqueness.
- Mutability metadata is stored in the module's compiled artifact for separate compilation, avoiding re-analysis of method bodies.

## 4. Ownership and Uniqueness Rules

- **Move semantics**: assignment `y = x` moves ownership of the object; `x` is no longer valid afterward.
- To call `.method()` on a mutating method, the compiler statically checks that the owning variable is unique at that point (no prior moves into other names, no active aliases). This is guaranteed by straightforward data-flow analysis without a borrow checker.
- For pure `.`-calls and all `!`-calls, there are no uniqueness constraints — the object may be shared (e.g., through an arena handle).

## 5. Memory Management

- **Local data** is allocated in a region (arena) tied to the scope. When the scope exits, the entire region is freed in O(1).
- **Shared state** is implemented via module-level arenas using handles (index + generation). Access to data is dynamically checked.
- Memory arenas are not coupled to method signatures: mutating `.`-methods modify an object in its native arena; `!`-methods create new objects (possibly in the caller's arena); ownership transfer occurs through returns or assignment.
- No garbage collector, predictable performance.

## 6. Syntax and Examples

```
class List {
    items: Array<Int>

    // Pure methods (do not change self)
    fun len() -> Int { return self.items.len() }
    fun get(idx: Int) -> Int { return self.items[idx] }

    // Mutating methods (change self) — no mut in declaration
    fun append(i: Int) { self.items.push(i) }
    fun sort() { self.items.sort() }
    fun pop() -> Int {
        let val = self.items.last()
        self.items.removeLast()
        return val
    }

    // Identity-changing transformations (always create a new object)
    fun sort!() -> List { return List(self.items.sorted()) }
    fun pop!() -> (List, Int) {
        let new_items = self.items.withoutLast()
        return (List(new_items), self.items.last())
    }
}

fun demo() {
    let nums = List([1, 2, 3])

    // Pure calls (allowed on any object)
    let size = nums.len()
    let first = nums.get(0)

    // Mutating calls (only if nums is unique)
    nums.append(4)          // OK: nums is unique
    nums.sort()             // OK
    let last = nums.pop()   // OK, returns the value

    // If we had done let other = nums earlier, nums would no longer be unique,
    // and the following mutating call would cause a compilation error.

    // Identity-changing transformations (always allowed)
    let sorted = nums!sort()           // new sorted list, nums unchanged
    let (new_list, elem) = nums!pop()  // new list without last + the element itself
}
```

## 7. Advantages of the Model

- **Minimal syntax** — no `mut`, no `&mut`, no lifetime annotations. Only `.` and `!`.
- **Explicit intent**: looking at a call, you immediately see — object is preserved (`.`) or a new one is created (`!`). The fact of mutation is inferred by the compiler (and may be shown in the IDE).
- **Memory safety without GC**: regions + handles + move semantics prevent use-after-free, double-free, and leaks.
- **Predictable performance**: O(1) arena allocation and deallocation, no pauses.
- **Low barrier to entry**: developers familiar with Python/Java see familiar syntax with minimal additions.
- **Compiler absorbs the complexity**: mutability inference, uniqueness checks, arena management — all automatic.

## 8. Trade-offs and Limitations

- Requires getting used to the `!` operator for creating new objects (though it is semantically transparent).
- Mutability inference may produce unexpected results in complex cases (e.g., methods with side effects that do not change `self` will be considered pure — this is correct but may surprise). However, for the vast majority of code, inference is trivial.
- Complex shared-mutation scenarios may require additional patterns (arenas, handles), but these are confined to the module level and do not affect core code.

## 9. Conclusion

We obtain a hypothesis for a language that can be described as **"Python with identity-based memory safety."** It has neither `mut`, nor a borrow checker, nor GC. All safety is built on two operators (`.` and `!`), automatic effect inference, and a strict but simple ownership system. This is a path toward a systems language accessible to a broad audience and suitable for predictable high-performance systems.
