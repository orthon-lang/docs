# Goals

> Concrete aims derived from the project's vision.
> Each goal is traceable to a section of [`VISION.md`](VISION.md) and
> provides criteria against which design decisions are measured.

---

## Goal 1: Architectural Integrity

**Derived from:** VISION.md — *Language Designed Like Software*

**Statement:** The language itself must be a well-engineered system.
The Core Language and Standard Library define interfaces and contracts;
the Implementation Strategy fulfills them. Evolution happens through
new strategies and libraries — not through changes to the language core.

**Rationale:** A language that follows SOLID principles is itself
maintainable, testable, and extensible. This architectural foundation
makes all other goals achievable.

**Criteria:**
- [ ] The Core Language is closed for modification; all extensions go
      through the Standard Library or Implementation Strategy.
- [ ] Implementation Strategies are interchangeable without changing
      program semantics (Liskov Substitution).
- [ ] A new capability (e.g. GPU computing, new GC algorithm) can be
      added through a new Strategy or Library without modifying the
      Core Language.
- [ ] Every language component has a single, well-defined responsibility.

---

## Goal 2: Inherited Wisdom

**Derived from:** VISION.md — *Learn from What Came Before*

**Statement:** Orthon acknowledges its intellectual debt to Python and
Java — preserving what worked, fixing what didn't, and designing for the
lessons only hindsight provides.

**Rationale:** Starting from a clean sheet is a privilege. The project
must exercise it responsibly by learning from proven languages rather
than repeating their mistakes.

**Criteria:**
- [ ] Python-inherited: readability and approachable syntax are
      preserved.
- [ ] Python-rejected: no significant whitespace, no scope ambiguity,
      no unchecked runtime type errors, no GIL-like concurrency model.
- [ ] Java-inherited: explicit semantics, static type safety, and
      disciplined architecture are preserved.
- [ ] Java-rejected: no ceremonial verbosity, no everything-is-an-object
      dogma, no checked exception fatigue, no stagnant standard library.
- [ ] A programmer familiar with Python or Java can read idiomatic
      Orthon code with minimal learning overhead.

---

## Goal 3: Comfortable by Construction

**Derived from:** VISION.md — *Comfortable by Design*

**Statement:** Reading Orthon code should require less effort than
writing it. The Principle of Least Astonishment governs every feature:
the behavior of a construct must match what a competent programmer
intuitively expects. Orthogonality ensures that what is learned in one
part of the language transfers directly to every other part.

**Rationale:** A language that surprises its users erodes trust.
Predictable, orthogonal design reduces cognitive load and makes the
language accessible to a wide range of programmers.

**Criteria:**
- [ ] Every language construct has exactly one responsibility and
      combines freely with others (orthogonality).
- [ ] There are no special cases — no context-dependent syntax, no
      rules that apply only in specific circumstances.
- [ ] A competent programmer can correctly predict the behavior of any
      construct on first encounter (Principle of Least Astonishment).
- [ ] No two constructs overlap in responsibility.

---

## Goal 4: LLM Readiness

**Derived from:** VISION.md — *Designed for the LLM Era*

**Statement:** Orthon is designed for an era where code is increasingly
written alongside LLMs. The same properties that make Orthon comfortable
for humans — orthogonality, consistent rules, explicit semantics — make
it uniquely suited for AI-generated code.

**Rationale:** LLMs predict code probabilistically. Every special case,
exception, and context-dependent rule expands the prediction space and
increases the chance of error. Orthon minimizes this surface by
construction.

**Criteria:**
- [ ] Every language construct passes an **LLM Generability Gate** —
      validated that LLMs can reliably produce correct code using it.
- [ ] No implicit behaviors exist — every behavior-changing operation
      is visible in the syntax, eliminating hallucination surface.
- [ ] The language's rule surface is small enough that an LLM can
      internalise it within its context window.
- [ ] Extensibility through libraries and strategies means the semantic
      core remains stable; LLMs do not need to learn a changing language.
- [ ] The Execution Program model provides a natural artifact boundary
      for LLM code generation — the LLM produces intent, the Engine
      handles execution.

---

## Goal 5: Execution Integrity

**Derived from:** VISION.md — *Execution Program*

**Statement:** A program should be fully defined — not just its source
code, but the context required to execute it. Language version, runtime,
dependencies, permissions, and resource requirements are part of the
program's definition, not external infrastructure bolted on after the
fact.

**Rationale:** A program that declares what it needs is reproducible by
construction, portable across environments without modification, and
debuggable without guessing about its surroundings.

**Criteria:**
- [ ] The same Execution Program runs identically in any conforming
      Engine without source-level changes.
- [ ] All execution context (language version, dependencies,
      permissions, resource requirements) is declared within the
      program artifact.
- [ ] Different targets (development, testing, production) fulfill the
      same contract through different Engine configurations — the
      program artifact does not change.
- [ ] Portability is a property of how the program is defined, not a
      configuration problem for deployment pipelines.

---

## Goal 6: Long-term Stability

**Derived from:** VISION.md — *Goal* (the existing single goal)

**Statement:** Keep the language stable for decades while allowing
implementations to continuously improve without breaking existing
programs.

**Rationale:** The ultimate measure of a language's design is not how
many features it has at launch, but how well those features hold up
over decades of use. Stability is the capstone goal — it is what makes
Orthon a responsible choice for long-lived systems.

**Criteria:**
- [ ] Programs written for Orthon v1.0 run correctly (produce the same
      semantics) on Orthon v10.0.
- [ ] The Core Language changes only when new semantics cannot be
      expressed through composition of existing Core primitives.
- [ ] Improvements to performance, memory usage, or code generation
      never alter observable program behavior.
- [ ] The language specification grows primarily through the Standard
      Library and Implementation Strategies, not through core keyword
      or syntax additions.

---

## Non-goals

The following are explicitly **not** goals of the Orthon language design.
Decisions that advance these non-goals at the expense of the stated goals
are rejected.

- **Maximum performance.** Orthon targets correctness, clarity, and
  stability first. Raw throughput is delegated to the Implementation
  Strategy and is never a design goal of the language itself.
- **Maximum ecosystem size.** Orthon is a new language with no
  compatibility obligation to any existing platform. Ecosystem growth
  follows design quality, not the reverse.
- **Novelty for its own sake.** Orthon does not seek to be innovative
  in language research. It seeks to combine known, proven ideas into a
  coherent whole.
- **Backward compatibility with any existing language.** Orthon starts
  from a clean sheet. Compatibility with Python, Java, or any other
  language is not a goal — only intellectual honesty about what is kept
  and what is rejected.
- **Zero-cost abstractions.** While efficiency matters, Orthon will not
  compromise explicit semantics or correctness guarantees for the sake
  of eliminating abstraction overhead. The programmer's mental model
  takes precedence over peak throughput.
- **REPL-first or notebook-first interaction.** Orthon is designed for
  program-scale development with the Execution Program as its primary
  artifact. REPL and notebook interfaces may be built on top, but they
  are not design drivers.

---

## Goal Interaction Map

Some goals pull in the same direction; others require explicit balance.

| Goals | Relationship |
|-------|-------------|
| **G1** (Architectural Integrity) ↔ **G6** (Long-term Stability) | Strongly aligned. SOLID architecture directly enables long-term stability. |
| **G3** (Comfortable) ↔ **G4** (LLM Readiness) | Strongly aligned. The same orthogonal design serves both humans and LLMs. |
| **G5** (Execution Integrity) ↔ **G6** (Long-term Stability) | Aligned. Self-contained programs are stable by construction across environments. |
| **G2** (Inherited Wisdom) ↔ all others | Informative. Provides guardrails for all design decisions. |
| **G1** ↔ **Non-goal: Zero-cost abstractions** | Tension. Architectural layers may impose abstraction cost; documented trade-off accepted. |

---

## Traceability

Each EDR and design proposal throughout the project should reference
which goal(s) it serves. This creates a traceable chain from vision to
decision:

```
VISION.md → GOALS.md → DESIGN_PRINCIPLES.md → EDR → Language Specification
    ↑                                                    ↑
    └──── why ────────────── how ────────────── what ────┘
```

When a new EDR is created, include a `**Goals served:**` line listing
the goal numbers this decision advances. When a design review raises
concerns, frame them relative to the goals.
