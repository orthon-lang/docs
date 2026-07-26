# Hypothesis: Region-Based Memory Management

## 1. Core Principles

- All dynamic memory is allocated exclusively in **arenas** (synonym: regions) — linear buffers whose lifetime is strictly bounded by a lexical scope (function, block, module).
- When the scope exits, the entire arena is freed in O(1) without individual object deallocation and without a garbage collector.
- The language has **no reference types** (no `&`, `&mut`) and **no manual memory management** (`malloc`/`free`).  
  Indirect access is allowed only through indices/handles into arrays stored within the same or a parent arena, or through owning slices (see below).
- All values follow a **linear / affine ownership discipline**: each value has exactly one owning name at any point in time. Passing a value = moving ownership, after which the original name becomes inaccessible.
- **No global objects in the traditional sense**, but **module-level arenas** exist — arenas whose lifetime matches that of the module. They are created implicitly when the module loads and destroyed when it unloads. Mutable data in a module arena is forbidden (constants only), or mutation is allowed only through explicitly synchronized primitives.

## 2. Arena Structure

- **Local arena** is tied to a specific block `{ }` or function body.  
  The compiler inserts its creation at the start of the scope and its reset at the end.  
  All local variables declared in that scope are placed in this arena.
- **Expression arena** — for intermediate values not bound to a name, a temporary arena may be used, living until the end of the enclosing expression.
- **Module arena** — one per module, its lifetime matching that of the module. Filled during module initialization (with constants, statically known structures) and, possibly, dynamically allocated objects explicitly placed there by the programmer.
- Moving a value from one arena to another is performed by bytewise copying (if the object contains no interior pointers; see type constraints). After copying, the memory in the source arena is considered uninitialized.

## 3. Data Types and Indirection

- **Primitive types** — integers, floats, booleans — are trivially copyable.
- **Composite types** — structs and enums — are stored inline, without pointers to fields. Nested values lie entirely inside the enclosing object.
- **Arrays and vectors** — dynamic arrays (`Vec<T>`) place their elements in the same arena as the vector descriptor itself. The descriptor contains: a pointer to the data start, length, and capacity (all three are numbers stored directly in the struct). When a vector is moved, its data is physically copied into the target arena; the source vector is invalidated.
- **Slices** — absent as a reference type. Instead, an **owning slice** (`OwnedSlice<T>`) is used: a descriptor with pointer, length, and capacity, similar to `Vec`, but not allowing reallocation. It can be obtained from a `Vec` by discarding the ability to grow.
- **Strings** — `String` = `Vec<u8>` (UTF-8 encoded), `Str` = `OwnedSlice<u8>` with UTF-8 guarantee. No reference `&str`.
- **Recursive data structures** (lists, trees) are built not through pointers but through **arena indices**: the type `Index<T>` — a pair (arena ID, offset within it). However, this requires:
  - either static lifetime annotations for the arena,
  - or region inference that determines index validity from the scope.
  In the first iteration, recursive structures are modeled through arrays with integer indices, without lifetime safety guarantees (programmer responsibility or runtime checks).

## 4. Mutation and `mut`

- A variable declared as `mut` can change its value in place without moving to a new arena. The object remains in the arena where it was created and mutates within that arena's memory.
- If the value is moved from a mutable variable (e.g., passed to another function as non-`mut`), a full copy occurs into the caller's ownership domain, and the original variable becomes uninitialized (it cannot be used until reassigned).
- Non-`mut` variables are immutable; their value cannot be modified, but can be moved (consuming the variable).

## 5. Closures

- Closures capture variables from the enclosing scope **only by move**. When a closure is created, all captured values are copied into an arena associated with the closure itself (the arena is created at the closure definition point and lives as long as the closure is reachable).
- A closure returned from a function is moved into the caller's arena in the usual way (by copying). If the closure captured large structures, they are copied as well.
- Capture by mutable reference is not supported, since no references exist. To simulate mutable state between calls, a new closure must be returned together with the updated state, or objects in a shared module arena must be used (see concurrency).

## 6. Module Arenas and Shared State

- A module arena contains only **immutable data**, or data protected by synchronization primitives (mutex, atomic cells).
- Access to module arena data from other functions is possible only through **full copying** of the value from the arena into a local variable (for immutable data), or through special APIs (for synchronized cells).
- Thus, no globally shared mutable state arises, which rules out data races at the type level.

## 7. Concurrency

- An arena is owned by a single thread; it cannot be shared across threads.
- To transfer data to another thread, a **move** of the value into a channel is used. The value is copied into the receiver's arena.
- If immutable large data needs to be shared, it can be placed in a special *read-only* arena that may be read from multiple threads (but not modified). Access to such an arena is synchronized either implicitly through reference counting at the lifetime level, or the arena lives forever (like a module arena).

## 8. Lifetime Analysis (Region Inference)

- Since direct references are absent, the main source of errors is using an index into an arena that has been freed. To prevent this, the language may include **region inference** that does not require user annotations. It tracks which arenas are alive at each program point and forbids retaining an index beyond the arena's lifetime.
- Alternatively, in the first iteration, only indices into "global" (module) arenas are permitted, or runtime checks (with a panic on access to a dead arena) are employed.

---

## Open Questions

### Expressiveness and Types
1. Are any forms of pointers (e.g., raw pointers for FFI) allowed at all? If so, how is it guaranteed that they do not outlive their arena?
2. How to model recursive data structures without significant performance loss? Is a purely index-based approach with manual "memory" management inside an array acceptable?
3. Are algebraic types with nested recursion needed (like `data List a = Nil | Cons a (List a)`) and how do they fit into an inline representation?
4. What to do with traits (interfaces) and dynamic dispatch if virtual tables require references?

### Performance
5. How critical is deep copying of vectors and strings on every move? Can lightweight "owning slices" be introduced, avoiding data copying while remaining lifetime-safe? Would this require implicit lifetime annotations?
6. What is the overhead of a bump allocator arena? Is frequent creation of small arenas (e.g., inside loops) efficient?
7. Is memory reuse within an arena permissible if objects inside it are destroyed earlier (e.g., via explicit `drop`), or is the arena strictly monotonic?

### Lifetime Analysis
8. Is fully automatic region inference without annotations feasible, and how does it interact with polymorphism and higher-order functions?
9. If arena indices are allowed, should the language require the programmer to annotate lifetimes for structs that contain them? Or would a simplified system like "RC-like" regions be adopted?

### Module Arenas and Shared State
10. What syntax does the user use to explicitly place a value into a module arena? What are the immutability guarantees?
11. How to reconcile module arenas with generics, if the module arena is parameterized by types? (arena as a static resource)
12. Are mutable static variables wrapped in atomics/mutexes permitted without violating guarantees?

### Closures and Higher-Order Functions
13. Should closures that capture nothing be convertible to plain function pointers? If so, this again introduces indirection.
14. Is it possible to optimize so that a closure that captured immutable data does not copy it in full, but instead places it in a separate arena and stores an "index"? How would this affect safety?

### Concurrency and Parallelism
15. How to organize efficient parallel processing of immutable data placed in a shared read-only arena? Is a reference-counting mechanism for the arena needed to free it when all readers have finished?
16. Is a "one-arena-per-thread" model feasible, but with the ability to send an entire arena to another thread as a single ownership unit (move arena), to avoid copying large object graphs?
17. How to interact with external libraries that use threads and require shared memory?

### Ergonomics and Ecosystem
18. How acceptable is it for the programmer to constantly copy even small composite values on every assignment/pass? Is implicit copy elision (similar to C++ NRVO) required?
19. Can the compiler automatically infer that a value does not leave the arena and allow implicit borrowing within the same arena as syntactic sugar? Or would this dilute the guarantees?
20. How to reconcile the model with deterministic destructor execution? If an arena is freed as a whole, destructors may not be called. Is it acceptable for the language to have no guarantee of destructor invocation for "erased" arenas?

---

## References

Region-based memory management is a classic term from research (Tofte, Talpin, ML Kit), where regions may be lexical or dynamic. This model is a variant of region inference without explicit annotations, tied to scopes.
