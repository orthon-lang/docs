# Smart Cast

## Issue (Why)

After checking that a value is of a specific type, the programmer should not need to cast it again — the compiler should know. In languages without smart casts, a type check proves a value is a specific type, yet the programmer must cast it again. The compiler has lost the information that the check provided.

Smart casting solves this: **the compiler tracks type-narrowing information through control flow constructs** (`if`, `when`, `&&`, `||`, `return`, `throw`) and automatically narrows the type of a variable within the relevant scope.

## Principles

1. **Automatic type narrowing after type checks** — After `if value is String`, the compiler narrows the type of `value` to `String` within the true branch.
2. **Null check narrowing** — After checking `value isnt None`, the compiler unwraps `Option[T]` to `T` in the relevant scope.
3. **`when` branch narrowing** — Each branch of a `when` expression narrows the scrutinee's type for that branch (see PATTERN_MATCHING, EDR-025).
4. **Immutability prerequisite** — Smart cast only applies to effectively-immutable variables (`val` or non-reassigned `let`). If the variable could change between the check and the use, the cast is unsafe.
5. **No cast in compound expressions** — In `value is String && value.length > 0`, the smart cast applies to the second operand because short-circuit evaluation guarantees the check passed.
6. **Explicit cast escape hatch** — `value as Type` performs an unchecked cast when the programmer knows more than the compiler can prove.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| Type Narrowing Policy | Determines the flow-sensitive type tracking rules — which control flow constructs trigger narrowing |
| Smart Cast Safety Policy | Governs when narrowing is safe — immutability requirement, reassignment reset |
| Explicit Cast Policy | Controls the `as Type` escape hatch behaviour |

## Model (What)

### Type Check Narrowing

```orthon
if value is String:
    print(value.length)      # no cast needed — compiler knows it's String
```

### Null Check Narrowing

```orthon
if user.email isnt None:
    send_email(user.email)   # compiler knows email is not None
```

### `when` Branch Narrowing

```orthon
when value:
    case String:   print(value.length)
    case Int:      print(value + 1)
    case _:        print("unknown")
```

## Default Strategy

Flow-sensitive type narrowing after type checks (`is`, `isnt`), null checks, and `when` branch patterns. Per-variable tracking; resets on reassignment. Conservative — if unsure, stays at the wider type.

## Alternative Strategies

| Strategy | Description |
|---|---|
| No smart cast | Programmer must manually cast after every type check (Java pre-16). Verbose but no magic. |
| Explicit narrow | Narrowing requires explicit syntax (e.g., `value.narrow<String>()`). Visible but verbose. |
| Aggressive narrowing | Smart cast applies to `var` variables if the compiler can prove they are not modified between check and use. More ergonomic but more complex analysis. |

## Open Questions

1. Should smart casts apply to mutable variables if the compiler can prove they are not modified between check and use (escape analysis)?
2. How does smart cast interact with extension functions — can the compiler narrow to a specific interface to enable extension function calls?
3. Should function return types participate in narrowing?

## Decision History

- **EDR-060:** Smart Cast accepted as Language feature — type narrowing after type check requires compiler-level control flow analysis.
- **Classification per D-03:** Language. Flow-sensitive type narrowing requires the compiler to track type information across control flow edges — a compiler-level analysis not expressible via primitive composition.
- **Relationship with PATTERN_MATCHING (EDR-025):** Smart cast is partially subsumed by pattern matching. In `when` branches with explicit type patterns, the narrowing is inherent to the match arm. Smart cast handles non-pattern scenarios: `if` checks, short-circuit operators, and explicit type queries outside `match` expressions.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/DESIGN_PRINCIPLES.md`
- [ ] `how/process/DECISION_PIPELINE.md`
