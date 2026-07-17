# Working Backwards — Why Orthon?

> Applying the Working Backwards method to derive Orthon's purpose
> from the desired programmer experience.

---

## User Story

```
As a working programmer who values clarity and control,
I want a language that is small enough to hold in my head
yet expressive enough to build real systems,
so that I spend less time deciphering accidental complexity
and more time solving actual problems.
```

---

## Press Release

**For Immediate Release**

**Orthon: A Language Designed Like Software Should Be**

Programming languages have grown fat on legacy — accumulating special
cases, implicit behaviors, and context-dependent syntax that rewards
memorization over understanding. Orthon takes a different path.

Orthon is a small, orthogonal language where every construct has one
job and combines freely with others. What you learn in one part of the
language transfers directly to every other part. There are no surprises,
no hidden conversions, no special cases waiting to ambush you at 2 AM.

Built on SOLID architecture principles, Orthon separates the Core
Language from the Standard Library and Implementation Strategy —
meaning the language evolves without breaking your code, and different
backends (default, embedded, high-performance) are interchangeable
without changing a single line of your program.

Inspired by what Python and Java got right, designed to fix what they
got wrong. Explicit where Python is implicit. Concise where Java is
verbose. Consistent where both have accumulated exceptions.

Orthon is not another language to learn. It is the language you already
know how to use — because it follows the rules you already expect.

---

## FAQ

### Who is Orthon for?

Working programmers who build and maintain real systems. Orthon is not
a research language, a DSL, or a teaching toy. It is designed for the
same audience as Go, Rust, and Kotlin — developers who care about
correctness, readability, and long-term maintainability.

### How is Orthon different from Rust?

Rust solves memory safety without a garbage collector through its
borrow checker — a notoriously difficult mental model. Orthon shares
Rust's commitment to correctness but aims for a smaller conceptual
surface. Where Rust requires the programmer to internalize lifetimes,
regions, and ownership rules as a core skill, Orthon makes ownership
explicit but optional — you opt into the level of control you need.

### How is Orthon different from Go?

Go values simplicity through minimalism but often sacrifices
expressiveness — no generics (historically), no sum types, no
meaningful abstraction mechanism. Orthon's minimal core is not minimal
features; it is minimal *concepts* that compose into rich behavior.
Orthon provides generics, traits, pattern matching, and a proper type
system without compromising its orthogonal foundation.

### How is Orthon different from Kotlin?

Kotlin is pragmatic evolution of Java — it keeps the JVM ecosystem
while fixing Java's verbosity. Orthon starts from a clean sheet,
unconstrained by backward compatibility with any existing platform.
This allows Orthon to achieve a level of consistency and orthogonality
that a JVM-hosted language cannot match.

### When would I choose Orthon over these alternatives?

When you want the safety and expressiveness of modern language design
*without* the complexity tax. Orthon is for projects where readability
and long-term maintainability matter more than raw performance (choose
Rust) or ecosystem size (choose Kotlin/Go). Orthon targets the sweet
spot: systems programming without the systems programming overhead.

### What do I lose by not using Orthon?

You lose a language that is designed to be learned once and applied
everywhere. You lose the confidence that comes from a language where
constructs compose predictably. You keep whatever ecosystem and tooling
you already have — which, for many projects, is the right choice.

### Is there a migration path?

Orthon is a new language, not a replacement for an existing one. There
is no migration path from Python, Java, or any other language. The
migration is personal: when you start a new project and want a language
that stays out of your way, Orthon is ready.

---

## Derived Requirements

From the story, press release, and FAQ, Orthon's Why documents must
satisfy these requirements:

| # | Requirement | Satisfied By |
|---|---|---|
| R1 | Articulate the core beliefs that make Orthon different | `MANIFESTO.md` |
| R2 | State the vision — what world does Orthon create? | `VISION.md` |
| R3 | Capture the spirit in memorable aphorisms | `ZEN.md` |
| R4 | Ground everything in programmer experience, not implementation | `WORKING_BACKWARDS.md` |
| R5 | Explain what we keep from predecessors and what we fix | `VISION.md` — "Learn from What Came Before" |
| R6 | Define comfort and predictability as design goals | `VISION.md` — "Comfortable by Design" |

### Traceability

Each Why document answers a specific question that a programmer
encountering Orthon would ask:

> **"Why should I care?"** → `WORKING_BACKWARDS.md`

> **"What do you believe?"** → `MANIFESTO.md`

> **"Where are you going?"** → `VISION.md`

> **"What's the essence?"** → `ZEN.md`
