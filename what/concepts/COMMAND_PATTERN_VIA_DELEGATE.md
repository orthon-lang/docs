# Command Pattern via Delegate

## Issue (Why)

The GoF Command pattern encapsulates a request as an object. It requires a separate class per command. For trivial operations — a single function call with captured parameters — the ceremony-to-meaning ratio is punishing. In a codebase of any size, this produces class explosion and indirection obscuring actual program logic.

Languages with first-class functions (delegates, closures, lambdas) collapse all command-like patterns into a single concept: **a callable value**.

## Principles

1. **First-class functions subsume the Command pattern** — Anywhere a Command object would be used, a delegate/lambda suffices.
2. **No separate Command abstraction** — The delegate model (EDR-036) provides `() -> void`, `T -> void`, `T -> R` and all arities.
3. **Undo is a pair of delegates** — Not a command interface with `execute` and `undo` methods.

## Policy Footprint

| Policy Type | Role in the concept |
|---|---|
| N/A — concept is obsoleted by existing primitives | |

## Model (What)

The Command pattern is obsoleted by the delegate model (EDR-036). Every use case is covered by an existing primitive:

```orthon
# UI event handler — delegate, not Command
button.on_click = fn() -> print("clicked")

# Task queue — list of delegates, not Command objects
tasks: List[() -> void] = []
tasks.push(fn() -> log("start"))
tasks.push(fn() -> process(data))

# Undo — pair of delegates, not Command interface
let operation = (
    execute: fn() -> insert_char(c),
    undo: fn() -> delete_char()
)
```

### Pattern Isomorphism

| OOP pattern | Delegate form |
|---|---|
| `Command` | `T -> void` delegate |
| `Runnable` | `() -> void` delegate |
| `Callable<V>` | `() -> V` delegate |
| `ActionListener` | `Event -> void` delegate |
| `Comparator<T>` | `(T, T) -> Int` delegate |
| `Consumer<T>` | `T -> void` delegate |
| `Function<T,R>` | `T -> R` delegate |
| `Predicate<T>` | `T -> Bool` delegate |

All are the same concept — deferred invocation — differing only in arity.

## Default Strategy

The Command pattern is a **StdLib** documentation concept. The delegate model (EDR-036) obsoletes the need for a Command abstraction. The StdLib documents this as "Command? Use a delegate" guidance rather than providing any Command-specific types or macros.

## Alternative Strategies

| Strategy | Description |
|---|---|
| Traditional Command interface | Class per command — ceremony overhead. |
| Enum-based commands | Tagged union of command variants — exhaustive but requires variant declaration per command. |
| Function reference | Simplest — just pass the function reference. |

## Decision History

- **EDR-071:** Command Pattern via Delegate accepted as StdLib documentation — the command pattern is obsoleted by delegate (EDR-036). The StdLib documents this as a guidance note rather than introducing any Command-specific constructs.
- **Classification per D-03:** StdLib (documentation-only). The concept exists only to document that first-class functions subsume the pattern. No new semantics, types, or macros required.
- **Cross-reference:** Delegate (EDR-036) — the execution delegate provides first-class functions that eliminate the Command pattern. PATTERN_MATCHING_DISPATCH (EDR-026) — dispatch-based approaches provide alternative command routing.

---

### Affected Documents

- [x] `what/CORE_CONCEPTS.md`
- [x] `what/GLOSSARY.md`
- [ ] `how/process/DECISION_PIPELINE.md`
