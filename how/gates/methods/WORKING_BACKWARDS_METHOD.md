# Working Backwards

> Start from the desired user experience and reason backwards to
> what the language must provide. If you cannot articulate who
> benefits and how, the feature does not yet deserve a specification.

---

## Origin

Developed at Amazon as a product-development discipline. The core
practice is to write a fictional **press release** and **FAQ**
before any design work begins — forcing the team to clarify *who*
the feature is for and *why* it matters before considering *how* to
build it.

---

## Application to Language Design

When evaluating a proposed language construct, apply Working Backwards
as a user-value gate: it tests whether the proposal solves a real
programmer problem or is merely technically interesting.

### 1. Write the User Story

```
As a [programmer role],
I want to [use this construct]
so that [concrete benefit].
```

If the story feels contrived or the benefit is vague, the proposal
needs more work before it can be evaluated.

### 2. Write the Press Release

Draft a one-paragraph announcement of this feature. It should answer:

- What is the feature called?
- What problem does it solve?
- Why should a programmer care?
- How does their daily code change?

If the press release reads like an implementation note rather than
a user-facing benefit, the proposal fails this gate.

### 3. Write the FAQ

Anticipate the questions a working programmer would ask:

- "How is this different from [existing construct]?"
- "When would I use this instead of [alternative]?"
- "What do I lose by not using it?"
- "Is there a migration path from current code?"

Unexplained or dismissed questions reveal gaps in the proposal.

### 4. Derive Requirements

From the story, press release, and FAQ, extract what the language
must provide. If the requirements list is empty or duplicates
existing constructs, the feature is not needed.

---

## Perspective

This method serves the **USER_VALUE_GATE** — the gate that asks
whether a proposed language feature solves a real problem justified
by the project's Vision. It is the first gate a proposal should
pass, because a feature nobody needs cannot be rescued by technical
elegance.

**Used by:** `USER_VALUE_GATE`
