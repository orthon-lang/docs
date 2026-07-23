# Hypothesis: Identity-Based Safety Language

## 1. Core Idea

Memory safety is achieved **without** a borrow checker, `mut`, or lifetime annotations. All control rests on:

- separating operations by **identity preservation / identity change** (`.`, `!`);
- automatic mutability inference by the compiler;
- ownership analysis with escape analysis for uniqueness checking;
- move semantics and memory regions with no GC.

## 2. Identity Operators

- **`v.method(args)`** — identity-preserved operation.
  `v` remains the same object. The method can be:
  * *read-only with respect to `self`* — does not modify `self` fields; allowed always, even in the presence of aliases.
  * *mutating `self`* — modifies `self` fields directly or by calling other mutating methods; **requires** `v` to be the unique owner (no other active references to the object).

- **`v!method(args)`** — identity-changed operation.
  Guaranteed: **the original object `v` is not modified.**
  The result is a value with a different identity — it may be either a physically new object or an existing one (e.g., an interned string, a shared structure). The compiler may apply structural sharing, copy-on-write, and other optimizations as long as observable semantics are preserved.
  Uniqueness of `v` is not required.

No `mut` keyword appears in user code. The distinction between mutating and pure methods is inferred by the compiler and stored in metadata (part of the ABI).

## 3. Mutability Inference (Compiler + ABI)

- **Definition of mutation**: a method is considered to mutate `self` if its body contains:
  - assignment to a field `self.field = ...`,
  - a call to a mutating method on `self` or one of its fields,
  - or passing `self` into a mutating context (e.g., `mutating_function(&self)` — though no such constructs exist in the language, only method calls).
- Analysis proceeds recursively through the call graph (within a module) or relies on metadata for imported functions.
- In the compiled module, each method stores a single bit `is_mutating_self`, enabling both the IDE and the compiler to determine the method's nature immediately under separate compilation.
- Side effects that do not affect `self` (I/O, modifying a global logger) are **not** considered `self`-mutation and do not require uniqueness. For other purposes (e.g., purity optimizations), the language may distinguish `read-only(self)` from `pure(world)`, but this is orthogonal to memory safety.

## 4. Ownership and Uniqueness Checking (Ownership + Escape Analysis)

- **Move semantics**: `y = x` transfers ownership; `x` is no longer usable afterward.
- **Last owner**: the compiler tracks ownership flows and determines where an object is last used (ownership checker).
- **Uniqueness for mutating calls**: for `v.method()` where the method mutates `self`, the compiler must prove that at the call site `v` is the sole owner of the object. This is more involved than straightforward dataflow, since the object could have been:
  - passed into another function that might have retained a reference (escaped),
  - placed into an arena or collection,
  - returned from a function.
  For this, **escape analysis** (including interprocedural) conservatively determines whether a reference could have leaked. If no leak is detected, the object is unique.
- If the compiler cannot prove uniqueness (e.g., the object was passed to a function with an unknown body), the mutating call is forbidden. The programmer may explicitly copy the object or use the `!`-version of the method to create a new version without modifying the original.

## 5. Memory and Regions

- Local variables are placed into **regions** tied to a scope (function, block). When the scope exits, the entire region is freed at once (O(1)).
- For shared state, **module-level arenas** with handles (index + generation) are used. Access to objects through a handle is always dynamically checked. Mutation through a handle requires special arena methods that can guarantee unique ownership within.
- `!`-methods may create new objects in the caller's region or reuse existing ones without compromising safety.

## 6. Syntax and Examples

```
class List {
    items: Array<Int>

    // Read-only with respect to self
    fun len() -> Int { return self.items.len() }
    fun get(idx: Int) -> Int { return self.items[idx] }

    // Mutating self (inferred by compiler)
    fun append(i: Int) { self.items.push(i) }
    fun sort() { self.items.sort() }
    fun pop() -> Int {
        let val = self.items.last()
        self.items.removeLast()
        return val
    }

    // Identity-changing transformations (always create a new value)
    fun sort!() -> List { return List(self.items.sorted()) }
    fun pop!() -> (List, Int) {
        let new_items = self.items.withoutLast()
        return (List(new_items), self.items.last())
    }
}

fun demo() {
    let nums = List([1, 2, 3])

    // Pure calls (identity preserved, self unchanged)
    let size = nums.len()
    let first = nums.get(0)

    // Mutating calls — only if nums is unique (escape analysis passes)
    nums.append(4)          // OK: nums is unique
    nums.sort()
    let last = nums.pop()

    // If we had previously passed nums to a function that could retain it,
    // uniqueness would be lost, and the next line would cause an error.
    // some_function(nums)
    // nums.append(5)  // Error: nums may not be unique

    // Identity-changing transformations (always allowed)
    let sorted = nums!sort()           // may return new or reused object
    let (new_list, elem) = nums!pop()  // new list + element, nums unchanged
}
```

## 7. Advantages

- **No `mut`, `&mut`, lifetime annotations** — syntax at the level of Python.
- **Explicit intent via `.` and `!`**: you can see whether the object's identity changes.
- **Memory safety without GC**: regions, move semantics, ownership + escape analysis eliminate typical errors.
- **Predictable performance**: O(1) arena allocation and deallocation, no GC pauses.
- **IDE-friendliness**: mutability and purity information is available through the ABI, enabling the IDE to hint at the nature of each call.
- **Flexible `!`-transform optimizations** without breaking semantics.

## 8. Trade-offs

- Requires sophisticated interprocedural escape analysis, which complicates the compiler — but this complexity is hidden from the user.
- No syntactic distinction between "pure" and "read-only with respect to self" methods; the difference may matter for some scenarios (e.g., concurrency), but can be added later via annotations.
- For objects living in arenas, dynamic handle checks impose a small overhead — the price for avoiding a global GC.

## 9. Conclusion

A cohesive language design is proposed in which memory safety rests on **identity-based safety**. `.` guarantees the object remains itself (and the compiler independently verifies the legality of mutation); `!` guarantees the original is unchanged under logical transformation. Powerful but hidden compiler analysis (ownership + escape) provides safety without manual annotations and without GC pauses. Such a language could become an attractive alternative to both Rust (simpler) and Go/Java (no GC), while maintaining predictable systems-level performance.
