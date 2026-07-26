# Hypothesis: `act` as Active Object with Mailbox and Delegate Semantics

> **⚠️ DRAFT — This document is a preliminary draft.**
> It was created as exploratory research material for the Concept Design Review
> process (Milestone 2). A concept is registered only after
> acceptance via EDR (Architecture category).
>
> **Last updated:** 2026-07-25
>
> **⚠️ SUPERSEDED by `DELEGATE.md` (2026-07-26).**
> The `act`-as-active-object hypothesis has been replaced. Actor is no longer
> a language concept. Delegated execution (`delegate`) provides the same
> semantics without introducing `act` as a language-level construct.
> See `DELEGATE.md` for the current hypothesis.
>
> **⚠️ Syntax note:** Code examples use abstract syntax. Final syntax is subject
> to language-wide agreement and will be specified in Phase 5 (Syntax).

## Problem

Conventional objects conflate independent concerns:

```orthon
class BankAccount:
    int balance = 0

    deposit(amount):
        balance += amount
```

State is accessible from any location holding a reference. This raises
unanswered questions:

- Who is allowed to mutate state?
- When does the operation execute relative to the caller?
- What happens under concurrent access?
- Who owns synchronisation?
- Who manages the object's lifecycle?

A conventional object answers:

> "I am just data plus methods."

Real systems require:

> "I own state and accept requests for mutation."

---

## Hypothesis

Introduce `act`:

```orthon
act BankAccount:

    int balance = 0

    proc deposit(amount):
        balance += amount
```

`act` means:

> an object with its own execution sequence and internal state.

---

### Key Difference

`class`:

```text
caller
 |
 v
method()
 |
 v
mutate state
```

`act`:

```text
caller
 |
 |
 delegate message
 |
 v

 mailbox

 v

actor execution

 v

mutate state
```

The call is no longer a direct function invocation — it is the sending of **an
intention**.

---

### `proc` Inside `act`

Inside an `act`, `proc` is used — not a separate `handle` keyword.

```orthon
proc deposit(amount):
```

The meaning comes from the enclosing `act` context and the caller's choice of
operator (`<-` vs `.`):

- `ba.deposit(10)` — direct invocation (as with any `class`)
- `ba <- deposit(10)` — delegate via mailbox

The same `proc` declaration serves both: `act` changes the *calling semantics*,
not the declaration syntax.

```
message
    |
    v
mailbox
    |
    v
proc execution
```

---

### `delegate` (`<-`)

Interaction uses the delegate operator `<-`:

```orthon
ba <- deposit(10)
```

Reads as:

> delegate execution responsibility to the owner of the object.

Not:

```
call deposit
```

but:

```
delegate deposit request to ba
```

#### Direct call (`->`)

```orthon
ba.deposit(10)
```

Meaning:

> I invoke the object's code.

#### Delegate (`<-`)

```orthon
ba <- deposit(10)
```

Meaning:

> I hand over responsibility for execution to the object.

This maps naturally onto the actor model.

---

### Lifecycle via Context Manager

```orthon
with BankAccount() as ba:

    ba <- deposit(10)
    ba <- deposit(20)
```

Semantics:

```
enter

create actor
create mailbox
start mailbox processor

---

work

---

exit

close mailbox
drain pending messages
release resources
```

`with` becomes a universal mechanism for:

- ownership
- lifetime
- cleanup

Not just for files.

---

## Synchrony: Two Variants

### Variant A — Fire and Forget

```orthon
ba <- deposit(10)
```

Meaning:

```
enqueue message
return immediately
```

Analogue: Erlang's `Pid ! Message`.

### Variant B — Await Result

```orthon
result = await ba <- withdraw(10)
```

Meaning:

```
delegate
+
await response
```

An alternative syntax without `await`:

```orthon
result = ba <- (withdraw(10))
```

---

## Where Is `coroutine`?

It disappears as a separate concept.

| Concept    | Mechanism                         |
| ---------- | --------------------------------- |
| Coroutine  | suspend / resume               |
| Act        | mailbox / receive / proc / next |

Coroutine is an **execution mechanism**.

Act is an **interaction model**.

If `act` exists, `coroutine` is no longer a primary abstraction — it becomes a
derived concept (a special case of an active object with a single-entry
mailbox).

---

## Comparison with Existing Models

| Model             | State    | Transfer     | Lifecycle         |
| ----------------- | -------- | ------------ | ----------------- |
| Python class      | external | call         | manual            |
| Python generator  | internal | yield/send   | poorly expressed  |
| async function    | internal | await        | scheduler         |
| Actor             | internal | message      | actor system      |
| Orthon `act`      | internal | delegate     | context manager   |

---

## Open Questions

### 1. Message Ordering

```orthon
ba <- deposit(10)
ba <- deposit(20)
```

Is it guaranteed that `deposit(10)` is handled before `deposit(20)`?

**Must be yes.** The model breaks without this guarantee. Sequential
delegates from the same caller preserve order (FIFO mailbox).

### 2. Error Propagation

What happens when a `proc` inside an `act` throws?

```orthon
proc withdraw(amount):
    if balance < amount:
        throw InsufficientFunds
```

Options:

```orthon
future = ba <- (withdraw(100))
await future        # error surfaces here
```

or implicitly surface on the delegate call if awaiting. Pure fire-and-forget
delegates that throw need a policy (crash actor? route to caller?).

### 3. Drain on Exit

```orthon
with BankAccount() as ba:
    ba <- deposit(10)
```

What if the message is still in the mailbox at `exit`?

Policy options:

- **Drain** — process all pending messages, then stop.
- **Cancel** — discard pending messages.
- **Block** — wait until the mailbox is empty.

### 4. Actor References and Ownership

Can an `act` reference be shared? If so, how does the language prevent
unsynchronised access to the delegate channel? See
[`OWNERSHIP.md`](OWNERSHIP.md) for the ownership model.

### 5. Composition with `emit`

An `act` proc could `emit` intermediate results:

```orthon
act Worker:
    proc process(items):
        for item in items:
            emit transform(item)
        return summary
```

How does the consumer subscribe to these emits? Does the delegate
operator propagate `emit` back to the caller? This interacts with the
hypothesis in [`EMIT_AS_INTERMEDIATE_RESULT.md`](EMIT_AS_INTERMEDIATE_RESULT.md).

### 6. Relationship to `ACTORS.md`

The existing [`ACTORS.md`](ACTORS.md) surveys actor models in Swift, Erlang,
and Pony. This hypothesis proposes that Orthon adopt `act` as a built-in
construct — not as a library — where the actor model replaces `coroutine`
entirely, and the delegate operator `<-` provides syntactic distinction from
plain method calls.

---

## Summary

If this hypothesis holds, Orthon gains a single unified mechanism for:

- coroutine
- actor
- async object
- state machine
- reactive component
- background worker

Rather than adding "yet another async" model, it changes the fundamental
unit of execution: **from function to autonomous object that owns state and
accepts intentions**. This aligns naturally with the emerging `emit`,
`delegate`, `proc`, and `with` semantics already under consideration.

---

## See Also

- [`ACTORS.md`](ACTORS.md) — original actor survey (superseded)
- [`DELEGATE.md`](DELEGATE.md) — current hypothesis
- [`OWNERSHIP.md`](OWNERSHIP.md) — ownership model
- [`EMIT_AS_INTERMEDIATE_RESULT.md`](EMIT_AS_INTERMEDIATE_RESULT.md) — emit/delegate interaction
- [../../../notes/actor-implementation-taxonomy.md](../../../notes/actor-implementation-taxonomy.md) — taxonomy of 12 actor implementation approaches
