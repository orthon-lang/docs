# Einstein's Method

> If you cannot explain a concept simply enough that it seems
> obvious, you do not understand it well enough to enshrine it in a
> language. Radical simplification is the most honest test of
> conceptual maturity.

---

## Origin

Attributed to Albert Einstein: "If you can't explain it simply, you
don't understand it well enough." Whether or not the quote is
authentic, the principle stands — the mark of a well-designed concept
is that it feels inevitable once understood.

---

## Application to Language Design

When evaluating long-term maintainability, Einstein's Method tests
whether a concept is simple enough to survive decades of use and
evolution.

### 1. Explain in One Sentence

Write exactly one sentence:

> "This concept lets the programmer [do X]."

If the sentence contains "and", "but", "except", "however", or
"unless", the concept is not simple enough. Break it down.

### 2. Explain to a Non-Expert

Could a programmer familiar with Python or Java understand the
concept from the one-sentence explanation alone? If they must ask
"What does that mean?", the concept still carries accidental
complexity.

### 3. Remove One Thing

What is the minimal viable subset of the proposal that still delivers
the core benefit?

- If you remove the most complex rule, does the concept still work?
- If you remove the most common use case, is the concept still worth
  having?
- If the subset is empty, the full proposal may be too complex for
  its own good.

### 4. Check for Obviousness

After the explanation, does the listener's reaction feel like "of
course, that's the natural way"? If the reaction is "interesting, I
never thought of it that way" or "clever", the concept still has
novelty masquerading as insight. A well-designed language concept
should feel like it was always missing.

---

## Perspective

This method serves the **LONG_TERM_MAINTAINABILITY_GATE** — the gate
that asks whether a decision will age well and whether the concept
carries conceptual debt. A concept that cannot be explained simply
will be misunderstood, misused, and regretted.

**Used by:** `LONG_TERM_MAINTAINABILITY_GATE`
