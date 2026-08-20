# Positioning

> **Canonical block.** This document distills Orthon's strategic
> answers — what problem we solve, why it is the central one, what
> approach we chose, what we gave up, how we measure success, and what
> can go wrong — into one readable statement. It consolidates decisions
> already recorded in [`VISION.md`](VISION.md), [`GOALS.md`](GOALS.md),
> [`MANIFESTO.md`](MANIFESTO.md), and
> [`WORKING_BACKWARDS.md`](WORKING_BACKWARDS.md); the concrete actions
> live in [`ROADMAP.md`](../when/ROADMAP.md). It introduces no new
> design decisions.

## Strategy in One Paragraph

Programming languages have grown by accretion, programs ship as
incomplete artifacts, and the dominant code producers of the coming
decade were never a design consideration. Existing languages split
into a pendulum: Python bought expressiveness by surrendering control;
Rust and Zig bought control by raising the cognitive price of entry.
Orthon's strategy is the missing synthesis — a language architected
like software (SOLID, an orthogonal core, explicit semantics) that is
expressive through composition rather than sugar, controllable through
explicit semantics rather than ceremony, and designed from the start
for both humans and LLMs, producing fully-defined Execution Programs
that are reproducible, portable, and stable for decades.

---

## 1. The Problem

Three problems define Orthon. They are documented in full under
*The Pain of This Era* in [`VISION.md`](VISION.md).

**1. Languages grew by accretion, not by engineering.** Most mainstream
languages accumulated features, special cases, and implicit behaviors
over decades. They were never architected — they evolved. Programmers
spend more cognitive effort deciphering accidental complexity than
solving actual problems.

**2. Programs are incomplete artifacts.** A program today is just
source code. Its dependencies, runtime version, resource requirements,
and permissions live outside the artifact — in Dockerfiles, CI configs,
deployment scripts, and institutional knowledge. Reproducibility is a
pipeline problem, portability a configuration problem, and debugging
requires reconstructing the environment from scattered clues.

**3. Languages were not designed for the LLM era.** Every special case,
implicit conversion, and context-dependent rule expands the prediction
space for an LLM and multiplies the chance of generated errors.
Languages designed before LLMs existed were optimized for human
memorization, not for reliable machine generation.

## 2. Why This Is the Central Problem

The chain of languages — C, C++, Java, Go, Rust — ended in a pendulum.
Python bought expressiveness by surrendering control; Rust and Zig
bought control by raising the cognitive price of entry. Neither pole
reconciled the two. The pain of this era is the missing synthesis: a
language as expressive as Python and as controllable as Rust, redefined
for the LLM era.

This is the central problem because all three pains trace back to the
same root — a language never engineered as a coherent system:

- Accretion is a failure of *engineering*: the language was never
  architected, so special cases accumulated.
- Incomplete artifacts are a failure of *explicitness*: execution
  context is left implicit, outside the artifact.
- The LLM gap is a failure of *design-for-the-producer*: the dominant
  producers of the next decade were not a design input.

Address the root — engineer the language as a system, make semantics
explicit, and design for both humans and LLMs — and each pain is
answered by the same core. Orthon treats this synthesis as the
strategic center of gravity; every other decision is subordinate to it.

## 3. The Chosen Approach

Six commitments define how Orthon answers the problem. Each is derived
from the vision and manifesto and is elaborated in the referenced
documents.

**1. A language architected like software (SOLID).** The Core Language
and Standard Library define interfaces and contracts; the Implementation
Strategy fulfills them. Evolution happens through new strategies and
libraries — not through changes to the core. (Goal 1 in
[`GOALS.md`](GOALS.md).)

**2. An orthogonal minimal core.** One concept, one syntax; one symbol,
one responsibility; composition over special cases. There are no
context-dependent rules to memorize — what is learned in one part of
the language transfers to every other part. (Ten principles in
[`MANIFESTO.md`](MANIFESTO.md).)

**3. Explicit semantics.** Every behavior-changing operation is visible
in the syntax. There is no implicit behavior to discover — for the
programmer, and for the LLM, this eliminates the hallucination surface.

**4. Intent over implementation.** The language expresses *what* the
program does; the Implementation Strategy chooses *how* it executes.
Both are explicit, at different layers.

**5. The Execution Program.** A program is a fully-defined artifact —
source code plus language version, dependencies, permissions, and
resource requirements. Reproducibility and portability are properties
of how the program is defined, not a deployment-pipeline problem.

**6. LLM as a first-class user.** The same properties that make Orthon
comfortable for humans — a small rule surface, consistent rules,
explicit semantics — make it reliable for LLM code generation. Every
construct passes an LLM Generability Gate. (Goal 4 in
[`GOALS.md`](GOALS.md).)

## 4. What We Give Up

Strategy is as much about refusal as about commitment. Orthon
explicitly gives up the following.

**From the Vision (canonical block of [`VISION.md`](VISION.md)):**

- Being another C syntax
- Being the fastest language
- Replacing C++
- Novelty for its own sake
- Backward compatibility with any existing language
- REPL-first or notebook-first interaction

**From the Goals (Non-goals of [`GOALS.md`](GOALS.md)):**

- **Maximum performance.** Correctness, clarity, and stability come
  first; raw throughput is delegated to the Implementation Strategy and
  is never a design goal of the language itself.
- **Maximum ecosystem size.** Ecosystem growth follows design quality,
  not the reverse.
- **Zero-cost abstractions.** Efficiency matters, but never at the
  expense of explicit semantics or correctness guarantees; the
  programmer's mental model takes precedence over peak throughput.
- **Formal verification of semantics.** The v0.1 specification is
  prose-based; formalization in Coq, Lean, or similar proof assistants
  is deferred until an implementation and real-world usage justify the
  investment.

**Rejected from inherited languages (Goal 2 of
[`GOALS.md`](GOALS.md)):**

- From Python: significant whitespace, scope ambiguity, unchecked
  runtime type errors, a GIL-like concurrency model.
- From Java: ceremonial verbosity, the everything-is-an-object dogma,
  checked-exception fatigue, a stagnant standard library.

## 5. How We Measure Success

Success is not a single KPI; it is a set of acceptance criteria that
the design must satisfy, measured at three levels. Each level already
exists as a concrete artifact — this section only names them.

**At the level of the whole language** — the Success Criteria in the
canonical block of [`VISION.md`](VISION.md):

- **Stable semantics** — programs from v1.0 run correctly on v10.0.
- **Small core** — all features decompose to a minimal primitive set.
- **Predictable evolution** — extensions through libraries, not core
  changes.
- **LLM generability** — LLMs can reliably produce correct Orthon code.
- **Execution portability** — the same program runs identically in any
  conforming Engine.

**At the level of each goal** — the acceptance criteria attached to the
six goals in [`GOALS.md`](GOALS.md). Every design decision is judged
against them; a decision that advances a non-goal at the expense of a
goal is rejected.

**At the level of each construct and milestone** — the gates and exit
criteria:

- Each construct must pass the validation gates in
  [`DECISION_VALIDATION.md`](../how/gates/DECISION_VALIDATION.md),
  including the LLM Generability Gate.
- The design is protected against decay by the measurable checks in
  [`FITNESS_FUNCTIONS.md`](../how/architecture/FITNESS_FUNCTIONS.md).
- Each phase of the roadmap defines its own exit criteria; a phase is
  not complete until they are satisfied
  ([`ROADMAP.md`](../when/ROADMAP.md)).

There is no business-style kill criterion — this is a solo-authored,
documentation-only project. The equivalent of an exit condition is the
freeze audit: v0.1 is accepted only when the specification is
self-consistent, every cross-reference resolves, and the validation
suite passes (Phase 8 of [`ROADMAP.md`](../when/ROADMAP.md)).

## 6. Risks

What can go wrong, and what already exists to counter it.

- **Core creep.** The language accumulates features and special cases —
  the very accretion Orthon criticizes. *Control:* Design Principles are
  locked as constitutional and change only via EDR; Phase 8.2 enforces
  global minimality.
- **Layer mixing.** A "Why" argument smuggled into a "What" document, or
  roadmap detail pulled into the vision layer. *Control:* layer-integrity
  rules and the English-only gate in [`AGENTS.md`](../AGENTS.md), plus
  the repository-wide link audit.
- **A decorative LLM gate.** "LLM generability" is claimed but never
  actually verified, so the headline differentiator stays a slogan.
  *Control:* the LLM Generability Gate is mandatory for acceptance, and
  Phase 8.4 runs real LLM generation tests and a conformance suite.
- **Implementing before the design is frozen.** The pull to start the
  compiler before M1 completes, breaking the design contract. *Control:*
  the "Design before implement" principle and the documentation-only
  scope of this repository ([`ROADMAP.md`](../when/ROADMAP.md)).
- **Solo-author drift.** Without reviewers, decisions quietly drift from
  the principles. *Control:* self-imposed rigor — Concept Design Review,
  validation gates, and the mandatory human sign-off before consequential
  EDRs ([`AGENTS.md`](../AGENTS.md) §7.4).
- **An incoherent freeze.** The final specification has dangling
  cross-references or contradictions between concepts. *Control:* the
  Phase 8.5 freeze audit — full link audit and a self-consistency check
  before tagging v0.1.

## 7. Cross-References

- [`VISION.md`](VISION.md) — the full rationale: pain, pillars, and the
  LLM-era framing.
- [`GOALS.md`](GOALS.md) — six goals with acceptance criteria and the
  complete non-goals list.
- [`MANIFESTO.md`](MANIFESTO.md) — the ten principles the approach
  rests on.
- [`WORKING_BACKWARDS.md`](WORKING_BACKWARDS.md) — the programmer-facing
  statement of why Orthon exists.
- [`AGENTS.md`](../AGENTS.md) — contribution protocol and document map
  (English-only rule, §10.9).
- [`ROADMAP.md`](../when/ROADMAP.md) — the coherent actions: ordered
  phases and milestones that turn the strategy into deliverables.
