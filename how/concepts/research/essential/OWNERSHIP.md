# Ownership

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created during Milestone 1 (Language Inventory) as exploratory work.
> It will be formally reviewed through the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-20

## Issue (Why)

How do you efficiently manage memory and prevent leaks without runtime overhead or unpredictable pauses?

The evolution of memory management reveals a fundamental tension:
- **Manual management** (`malloc`/`free`) — full control, but use-after-free, double-free, and memory leaks are endemic. In C, these account for ~70% of critical CVEs.
- **Tracing garbage collection** (Java, Go, Python) — automatic safety, but stop-the-world pauses, unpredictable latency, and memory overhead (GC header per object).

The core problem: **memory safety requires automation, but GC's runtime cost is unacceptable for systems programming**. We need compile-time memory safety with no runtime overhead.

## Principles

1. **No runtime overhead** — Memory management decisions are made at compile time; there is no GC, no reference counting (by default), no runtime cost.
2. **Compile-time safety** — Use-after-free, double-free, and memory leaks are prevented by the type system and borrow checker.
3. **Single owner** — Every value has exactly one owner. Ownership can be transferred (move), but not duplicated implicitly.
4. **Temporary borrowing** — References to owned data are temporary and subject to lifetime rules.
5. **Deterministic destruction** — Destructors run at a well-defined point (end of scope or move) — no GC timing.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Allocation Policy | Determines how memory is allocated (heap, arena, static, GC) |
| Lifetime Policy | Governs how lifetimes are tracked — static borrow checking, regions, or runtime tracing |
| Mutability Policy | Controls whether borrowed references permit mutation (shared XOR mutable) |
| Destruction Policy | Defines destructor ordering and drop semantics |

## Model (What)

Ownership is a set of rules enforced by the compiler:

1. **Each value has exactly one owner** at any point in the program.
2. **Ownership can be moved** — when a value is assigned to a new variable or passed to a function, ownership transfers. The old binding is invalidated.
3. **Borrowing creates references** without transferring ownership:
   - Multiple shared read references (`&T`) are allowed concurrently.
   - Exactly one mutable reference (`&mut T`) is allowed, and no shared references exist simultaneously.
4. **Lifetimes** — every reference has a lifetime, checked by the compiler to ensure no reference outlives its referent.
5. **Destructors** run when a value goes out of scope, freeing resources deterministically.

```
// Move semantics
let a = createLargeData()
let b = a           // a is moved into b; a is now invalid
// print(a)         // compile error

// Borrowing
func length(s: &String) -> Int
    return s.len()

let msg = String("hello")
let len = length(&msg)   // shared borrow
// msg is still valid here
```

## Default Strategy

Arena-based allocation with ownership and static borrow checking (Rust-style). No GC. `&T` is shared borrow, `&mut T` is exclusive mutable borrow. Lifetimes are inferred or explicitly annotated.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Reference counting | `Rc<T>` / `Arc<T>` for shared ownership with runtime counting. |
| Tracing GC | Full garbage collection for cyclic graphs (opt-in). |
| Region-based | Bump allocation with region scope for short-lived workloads. |
| Linear types | Values must be used exactly once; no implicit drop. |
| Quantitative Type Theory (QTT) | Resource multiplicities (0, 1, ω) on bindings — a unified formalization of linear/affine usage and erasure. Enforcement-level counterpart to the single-owner invariant; a Strategy option, not Core-Language semantics (see `.planning/notes/qtt-vs-orthon.md`). |

## Open Questions

1. Should the language support a garbage-collected mode for non-system targets?
2. Can we support non-lexical lifetimes (NLL) by default?
3. How does ownership interact with closures — can they capture by move or by reference?
4. Should there be a way to "pin" a value to prevent moves after initialisation?
5. Can the borrow checker support partial moves from compound types?
6. What is the concrete syntax for ownership transfer? See `OWNERSHIP_TRANSFER_OPERATOR.md` for the `$` prefix operator hypothesis (vs. `move` keyword). This document defines the semantic model; that document explores syntax alternatives.
7. How is a returned borrow's relationship to its owner expressed or inferred, including through aggregates, iterators, nested calls, and multiple inputs? See the decision and scenarios below.

## Returned Borrows — Working Decision and Scenarios

> **Working decision:** Allow returned borrows within the owner's explicitly
> bounded scope when the compiler can prove the result's relationship to the
> owner and enforce the applicable lifetime and access rules. A returned
> borrow must not outlive its owner or its own explicit borrow scope; an
> active borrow also restricts conflicting access, including within the
> owner's scope. If the required proof cannot be established, reject the
> program at compile time.
>
> **Rationale:** A scope provides a visible upper bound for resource use,
> while returned views support resource-backed APIs and borrowing iterators.
> [EDR-094](../../../decision_records/architecture/EDR-094-for-expression-and-iteration-ownership.md)
> already specifies borrowed iteration; the
> [iterator protocol](../../../../what/concepts/ITERATOR_PROTOCOL.md) describes
> a returned iterator that borrows its source. The exact mechanism for
> expressing or inferring that relationship is not yet specified.
>
> **Alternative considered:** Prohibit returned borrows, including indirect
> returns in iterators and other aggregates. This reduces cross-function
> lifetime modelling, but restricts borrowing APIs; copies, positions, and
> scoped callbacks have different costs and limitations.
>
> **Open:** Relationship and scope syntax, multiple owners, reborrowing,
> aggregate and iterator dependencies, and interactions with FFI and
> concurrency. Whether every resource requires an execution context remains
> unresolved; this decision introduces no such requirement. This is a working
> research decision, not an accepted EDR or a completed design gate.

This comparison is research evidence, not an accepted design or executable
test. It applies [Nyquist Validation](../../../gates/methods/NYQUIST_VALIDATION_METHOD.md)
to the [semantic model's](../../../../what/SEMANTIC_MODEL.md) lifetime and
shared-versus-exclusive access invariants. The aim is to expose violations
and practical losses, not to prove complete coverage. The working assumption
is that each borrow lasts to the end of an explicitly bounded lexical scope;
the notation for that boundary remains open.

Compare two alternatives:

- **A — prohibition:** a function cannot return a borrowed view. Its caller
  must use another interface, such as a copy, a range, or a scoped callback.
- **B — linked return:** a function may return a borrowed view when the
  compiler can prove its relationship to a sufficiently long-lived owner and
  enforce access restrictions throughout the returned borrow's scope.

The following is illustrative pseudocode, not proposed Orthon syntax. `&T`
and `&mut T` denote read and exclusive borrows; `scope` marks an explicit
boundary. `Buffer` is a move-only resource. `select`, `header`, and
`header_mut` return views, not copies or independently owned handles.

### 1. Local Owner Escapes

```text
fun local_view() -> &Bytes:
    local = create_buffer(...)
    return view(local)
```

**A:** reject the returned borrow. **B:** reject it because `local` ends with
the call; the result would outlive its owner. Returning an owned buffer would
be a different interface. This scenario distinguishes safe return from an
unconditional permission to return references.

### 2. Owner Moved or Destroyed While Borrowed

```text
buffer = create_buffer(...)
scope:
    bytes = header(&buffer)
    consume(move buffer)  // tries to transfer or destroy the owner
    read(bytes)
```

**A:** reject the `header` interface because it returns a borrow. **B:**
reject the move: the returned view remains active until the end of `scope`,
and transferring or destroying its owner would invalidate the view or remove
the basis for proving its validity. A use after the move makes the hazard
observable. Whether some moves can be proven safe with a different storage
model remains open; this scenario assumes the move invalidates the view.

### 3. Result Depends on One of Several Inputs

```text
fun select(left: &Buffer, right: &Buffer, choose_left: Bool) -> &Bytes:
    if choose_left:
        return view(left)
    else:
        return view(right)

left = create_buffer(...)
right = create_buffer(...)
scope:
    bytes = select(&left, &right, runtime_choice())
    right.clear()
    read(bytes)
```

**A:** reject the `select` interface because it returns a borrow. **B:**
reject `right.clear()` unless the compiler can prove that `bytes` cannot
refer to `right` for this call. A conservative rule may require both inputs
to remain valid and unmodified until the result's scope ends. How the
relationship is expressed or inferred, and whether more precise proofs are
supported, remains open; accepting an untracked ambiguous relationship is
not an option. The runtime choice makes the possible conflict observable.

### 4. Returned Exclusive View

```text
buffer = create_buffer(...)
scope:
    bytes = header_mut(&mut buffer)
    read(buffer)              // conflicts with the exclusive view
    write(bytes, ...)
read(buffer)                  // after the scope
```

**A:** reject the `header_mut` interface because it returns a borrow. **B:**
reject `read(buffer)` inside the scope because the returned exclusive view is
active. Reading the owner after the scope is permitted if no other rule
prevents it. The same restriction applies to competing mutation, another
exclusive borrow, and a move that invalidates the view.

### 5. Valid View of a Caller-Owned Resource

```text
buffer = create_buffer(...)
scope:
    bytes = header(&buffer)
    read(bytes)
    buffer.clear()            // conflicts with the shared view
buffer.clear()                // after the scope
```

**A:** reject the `header` interface. **B:** accept the returned view and
reject the inner `clear`; permit the outer `clear` if no other restriction
applies. This is the positive case as well as an exclusivity check: ending
the call does not itself end a returned borrow of the caller's data.

### 6. A Borrow Hidden in a Returned Value

```text
fun make_iterator(buffer: &Buffer) -> Iterator<&Byte>:
    return iterator_over(buffer)

fun make_callback(buffer: &Buffer) -> Callback:
    return closure_that_reads(buffer)
```

**A:** reject both results if prohibition covers indirect escape through
containers, iterators, closures, and generic values. Rejecting only a direct
`&T` result would leave a loophole. **B:** accept only when each result
carries a tracked dependency on `buffer` and cannot outlive it. Calling the
result or destroying its owner while the dependency is active must obey the
same access and lifetime rules as a direct view. Whether and how such types
express that dependency is open.

### 7. Nested Return and Reborrowing

```text
fun inner(buffer: &Buffer) -> &Bytes:
    return view(buffer)
fun outer(buffer: &Buffer) -> &Bytes:
    return inner(buffer)

buffer = create_buffer(...)
scope:
    bytes = outer(&buffer)
    buffer.clear()
    read(bytes)
```

**A:** reject the borrowed results at both function boundaries. **B:**
propagate the relationship through both calls and reject `clear()` until the
result's scope ends. Checking only the immediately preceding call would lose
the original owner. Reborrowing an exclusive view must also prevent use of
the parent permission where that would conflict with the child; the exact
rules for temporarily suspending and restoring that permission remain open.

### 8. Two Views and Access Modes

```text
buffer = create_buffer(...)
scope:
    first = header(&buffer)
    second = header(&buffer)
    read(first)
    read(second)
    mutable = header_mut(&mut buffer)
    write(mutable, ...)
```

**A:** reject both borrowed-returning interfaces. **B:** allow both shared
views, but reject creation of the exclusive view while either is active.
Similarly, two exclusive views of overlapping data cannot coexist. Disjoint
parts might permit more precise reasoning, but splitting permissions and
proving disjointness are separate open design choices; a conservative
whole-owner rule can reject them safely.

### 9. Escaping Through a Store or a Wider Scope

```text
outer_slot = empty_slot()
scope:
    buffer = create_buffer(...)
    outer_slot.store(header(&buffer))
use(outer_slot)
```

**A:** reject the borrowed return; the prohibition must also cover putting a
borrow into longer-lived state through other APIs. **B:** reject storing the
view where it can survive `buffer`. A compiler must track dependencies through
assignments and aggregate fields, not merely check the return statement.
Storing a view is not intrinsically invalid when the destination cannot
outlive its owner and all access restrictions hold; whether such storage is
supported is open.

### 10. Control Flow and Scope Exit

```text
buffer = create_buffer(...)
scope:
    bytes = header(&buffer)
    if runtime_choice():
        log("early exit")
        exit scope
    read(bytes)
buffer.clear()
```

**A:** reject the borrowed-returning interface. **B:** accept this ordering:
the borrow ends on every path that exits its scope, including an early exit,
and `clear` occurs afterward. Replacing `exit scope` with a jump to a use of
`bytes` outside the scope must be rejected. Return, error propagation, and
unwinding need the same scope-exit and destruction-order analysis; their
concrete control-flow and cleanup semantics are not decided here.

### 11. Callback as an Alternative Can Also Escape

```text
outer_slot = empty_slot()
with_header(&buffer, callback(view):
    outer_slot.store(view)
)
use(outer_slot)
```

**A:** a callback is a safe alternative only if its borrowed parameter
cannot escape the permitted scope; reject the store when it would outlive
the borrow or owner. **B:** apply the same escape and access checks; allowing
returned borrows does not make arbitrary escape safe. A scoped callback
therefore still needs compiler-enforced borrowing rules and is not free of
lifetime design.

### 12. Concurrency and Suspension

```text
buffer = create_buffer(...)
scope:
    bytes = header(&buffer)
    start_task_that_reads(bytes)
    buffer.clear()
    wait_for_task()
```

**A:** reject the borrowed-returning interface; task capture through other
borrow paths still requires checking. **B:** reject `clear()` while the
task may read the view, and do not allow the task to outlive the owner.
Accepting a scoped task additionally requires the task and value types to
meet the concurrency model's requirements, which remain open. A suspension
point likewise cannot silently end a live borrow; whether a borrow may cross
one depends on the eventual async model and proof rules.

### 13. Foreign Code and Untracked Aliases

```text
buffer = create_buffer(...)
scope:
    bytes = header(&buffer)
    foreign_call_with_pointer_to(buffer)  // foreign code mutates buffer
    read(bytes)
```

**A:** reject the borrowed-returning interface; this does not by itself make
foreign access safe. **B:** a safe FFI interface must not allow the foreign
mutation to bypass the live shared borrow or retain an untracked pointer.
Reject the call when its declared access conflicts, or use a boundary whose
contract ensures the foreign code cannot perform the conflicting access.
What can be verified statically and what trusted foreign contracts must
guarantee remain open; the compiler cannot inspect arbitrary foreign code.

### 14. Independent Copy and Implementation Strategies

```text
original = regular_value([1, 2])
copy = original
original.change_first(9)
observe(copy)                 // expected: [1, 2]
```

**A and B:** the result must be `[1, 2]` under both eager copying and a
copy-on-write implementation. This is a control scenario: returning an
independent copy is not returning a borrow. A strategy that shares storage
internally must preserve independent value semantics. The example does not
prove that copying is affordable for every regular value.

### 15. Resource-Backed Data and API Trade-offs

```text
resource = open_resource_with_buffer(...)
scope:
    record = view_record(&resource)
    inspect(record)
```

**A:** reject `view_record`; an API may instead copy the record's copyable
data, return positions to be used with the resource, or invoke a scoped
callback. These choices have different allocation, lifetime and usability
costs; copying the resource itself is not implied. **B:** the borrowed view
can be accepted if its owner and scope are proven, with conflicting access
to the resource restricted while it is live. The example exercises the
practical loss under A; it does not demonstrate that any particular
alternative is always cheap or that B is always preferable.

### Coverage Limits and Follow-up

| Concern | Distinguishing evidence | Status |
|---|---|---|
| Local owner ends before returned view | Scenario 1 requires rejection under both alternatives | Covered by a documented scenario |
| Owner invalidated during returned borrow | Scenario 2 requires rejection of the invalidating move under B | Covered by a documented scenario |
| Ambiguous input owner | Scenario 3 requires tracking every possible owner or proving a narrower relationship | Covered by a documented scenario; inference and annotation rules remain open |
| Exclusive returned view conflicts with access | Scenario 4 requires rejection during the scope and permits access after it | Covered by a documented scenario |
| Useful return and shared-access conflict | Scenario 5 accepts the safe return under B and rejects conflicting mutation | Covered by a documented scenario |
| Indirect escapes and nested calls | Scenarios 6–7 track dependencies through returned values and calls | Covered by documented scenarios; aggregate and reborrow rules remain open |
| Concurrent or overlapping views | Scenario 8 rejects conflicting views but allows shared reads | Covered by a documented scenario; disjointness remains open |
| Stored views, control flow, callbacks | Scenarios 9–11 detect scope escape and conflicting access | Covered by documented scenarios; cleanup and callback rules remain open |
| Tasks and suspension | Scenario 12 checks owner lifetime and overlapping access | Covered by a documented scenario; concurrency and async models remain open |
| External pointers and FFI | Scenario 13 exposes a foreign mutation that bypasses tracking | Covered by a documented scenario; foreign contracts remain open |
| Copy semantics across implementation strategies | Scenario 14 compares the result after mutation under eager copying and copy-on-write | Covered by a documented scenario; concrete implementations do not yet exist |
| Resource API expressiveness | Scenario 15 contrasts a view with copies, positions, and callbacks | Covered by a documented scenario; costs and usability are not measured |

The scenarios document the alternatives behind the working decision; they do
not prove that all violations are covered. Practical evaluation still needs concrete
APIs for resource-backed buffers, containers, and iterators, including the
cost and usability of copies, ranges, and scoped callbacks. Further cases
may arise when generics, error handling, concurrency, and FFI are specified;
no finite list proves complete detection. The distribution
of regular and resource types is a design expectation, not measured evidence.

## Decision History

*To be filled during Concept Design Review (Milestone 2).*
