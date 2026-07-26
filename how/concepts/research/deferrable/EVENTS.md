# Events

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-22
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Issue (Why)

How should a language provide a first-class, type-safe publisher/subscriber pattern?

Java has no dedicated event keyword; observer patterns are built with interfaces, listeners, and ad hoc callback registration. C# adds the `event` keyword together with delegates to make multicast observer patterns a language-supported construct. This reduces boilerplate and prevents accidental misuse of the event delegate.

The core problem is: should Orthon offer a built-in event mechanism, or should observer-style interaction remain a library-level construct?

## Examples

| Language | Event abstraction | Type safety | Multicast | Caller syntax |
|---|---|---|---|---|
| **Java** | Listener interfaces + registration methods | Yes, manually enforced | Yes, by container code | `button.addActionListener(...)` |
| **C#** | `event` + delegate types | Yes, built into language | Yes, multicast with `+=`/`-=` | `button.Click += handler` |
| **JavaScript** | Event emitter / DOM API | Weakly typed | Yes | `on('click', handler)` |
| **Python** | Callbacks / signals libraries | No | Yes | `signal.connect(handler)` |
| **Rust** | `Fn` trait + channels | Yes | Yes with containers | `sender.send(msg)` |

### C# Example

```csharp
public delegate void ClickHandler(object sender, EventArgs e);

public class Button {
    public event ClickHandler Click;

    protected virtual void OnClick(EventArgs e) {
        Click?.Invoke(this, e);
    }
}
```

## Implications for Orthon

1. **Dedicated event declarations** — Orthon can support a first-class `event` construct that wraps a delegate type and restricts how subscribers can add or remove handlers.
2. **Multicast semantics** — Events should support multiple subscribers naturally, without requiring library-defined multicast containers.
3. **Encapsulation of invocation rights** — The event publisher can raise the event, while external code can only subscribe or unsubscribe.
4. **Observer syntax ergonomics** — `+=` / `-=` or a similarly concise syntax can make event subscription easier than manual listener registration.
5. **Compiler checks on event use** — The compiler should prevent misuse such as direct assignment or invocation from outside the declaring type.

## Open Questions

1. Should events be separate from delegates, or should they be syntactic sugar around function-typed properties?
2. How does Orthon express event payload types and optional sender semantics?
3. Should events support removal of individual subscribers, or only clear-all semantics?
4. What are the default dispatch semantics for events: synchronous, asynchronous, or configurable?

## Decision History

Initial research — no decisions recorded yet.

---

### Cross-References

- [DELEGATION.md](DELEGATION.md)
- [FUNCTIONS.md](FUNCTIONS.md)
- [VISIBILITY_AND_ENCAPSULATION.md](VISIBILITY_AND_ENCAPSULATION.md)
