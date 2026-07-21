# Analogies

> **Last updated:** 2026-07-20

> Architectural analogies that map Orthon concepts to familiar domains.
> Each analogy illuminates one aspect of the architecture — not as a
> decorative comparison, but as a reasoning tool that reveals constraints,
> relationships, and invariants that might otherwise be invisible.
>
> Analogies live here, not in the primary documents, to keep those
> documents focused on the architecture itself rather than on the
> explanatory devices used to communicate it.

## Semantic ISA

Orthon's evolution model mirrors the role of Instruction Set
Architectures in computing.

Assembly language is a stable interface between the program and the
processor. The ISA does not change when a new microarchitecture is
introduced — the same instructions run on different hardware
implementations.

Orthon Core is designed as a **Semantic ISA**: a stable semantic
interface between the program and any implementation. Just as a
compiler translates assembly into micro-ops for a specific CPU, the
Orthon compiler translates Core semantics into a concrete
implementation guided by the active Implementation Strategy.

| ISA Concept | Orthon Analogy |
|---|---|
| Instruction set | Semantic primitives (Core Language) |
| Microarchitecture | Implementation Strategy (Policies) |
| System library | Standard Library contracts |
| Silicon / hardware | Platform implementation |

A RISC-like philosophy applies: a small, stable set of semantic
primitives, with nearly all evolution happening in the strategy and
library layers around them.

### What the analogy reveals

- **Stability at the interface.** Just as x86-64 programs from decades
  ago still run on modern chips, Orthon programs should compile and run
  correctly across any future implementation strategy, provided they
  target the same Core semantics.

- **Evolution behind the interface.** Microarchitectures improve
  dramatically without changing the ISA. Similarly, Orthon
  implementations improve through new strategies — better allocators,
  smarter evaluation policies, novel concurrency models — without
  changing the language the programmer sees.

- **The cost of abstraction.** ISA abstractions hide complexity but
  also hide performance — programmers cannot always predict how the
  microarchitecture will realise their instructions. Orthon's semantic
  abstraction carries the same trade-off: the programmer codes to
  semantics, not to strategy, and must accept that performance
  characteristics are strategy-dependent.

### Boundaries of the analogy

- **No single die.** A processor ISA maps to one physical
  implementation at a time. Orthon can have multiple strategies active
  in different parts of a program, or even compose strategies within a
  single compilation unit.

- **Softer interface.** Processor ISAs are frozen by hardware
  fabrication timelines. Orthon's Semantic ISA is designed for
  stability but can evolve through the language design process — it is
  a discipline, not a physical constraint.

---

## RISC vs CISC: Core Design Philosophy

While the Semantic ISA analogy focuses on **interface stability**
(the Core as a stable contract between program and implementation),
the RISC vs CISC analogy focuses on **core design strategy** — what
belongs in the instruction set and what should be composed from it.

### The comparison

| Processor Architecture | Orthon Design Philosophy |
|---|---|
| **RISC**: small set of simple, general-purpose instructions. Complex operations are composed from sequences of simple ones. | **Orthon**: a minimal set of powerful, orthogonal Primitive Operations (Level 1). Library functions, Language Patterns (Level 2), and higher-level abstractions are composed from these primitives. |
| **CISC**: rich set of specialised, high-level instructions. Complex operations are built directly into the instruction set. | **Many languages today**: specialised constructs added directly to the core — list comprehensions, pattern matching, async/await, decorators, context managers, enums — each of which could, in Orthon's design, be expressed as a Pattern or Library abstraction. |

### What the analogy reveals

- **The RISC bet.** RISC processors win not because individual
  instructions are faster, but because a small, regular instruction set
  enables better compiler optimisation, simpler pipelining, and more
  efficient implementations. Orthon makes the same bet: a minimal,
  regular core enables better implementation strategies, simpler
  tooling (especially LLM tooling), and a language that evolves
  through its library and pattern layers rather than through core
  expansion.

- **The CISC trap.** CISC processors accumulated instructions because
  each seemed useful in isolation. Languages do the same: a feature
  that enters the core "just this once" becomes permanent, sets a
  precedent for the next addition, and gradually erodes the core's
  minimality. The Core Inclusion Filter (see
  [`CORE_INCLUSION_FILTER.md`](./CORE_INCLUSION_FILTER.md)) is
  Orthon's defence against this trap.

- **Composition as the alternative.** RISC's power comes from
  composing simple instructions into complex operations. Orthon's
  Semantic Dependency Architecture formalises this: Language Patterns
  (Level 2) are compositions of Primitive Operations (Level 1), and
  the Standard Library (Level 3) is built on Patterns. The language
  rewards composition rather than core expansion.

### Boundaries of the analogy

- **ISA vs. library.** A processor has only one instruction set.
  Orthon has a Spectrum: Library → Pattern → Primitive Operation.
  Most new features should land in the Library or Pattern layer, not
  the core — the processor analogy obscures this spectrum.

- **Performance assumptions.** RISC vs CISC was historically about
  hardware trade-offs (code density, pipeline complexity, power).
  Orthon's version of the comparison is purely about **design
  philosophy** — the concern is conceptual minimality, not hardware
  efficiency.

- **Compiler-as-microarchitecture.** In the processor analogy, the
  compiler translates ISA to micro-ops. In Orthon, the *same* compiler
  also translates Patterns to Primitives. The analogy does not capture
  this dual role — the compiler both *implements patterns* (core
  expansion) and *implements semantics* (strategy resolution).

### Related concepts

- [`CORE_INCLUSION_FILTER.md`](./CORE_INCLUSION_FILTER.md) — the
  procedural decision rule that implements this philosophy
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) § Semantic Dependency
  Architecture — the 6-level pyramid this analogy describes
- [`DESIGN_PRINCIPLES.md`](../DESIGN_PRINCIPLES.md) § Minimal Core —
  the principle formalised in this analogy
