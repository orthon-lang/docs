# Analogies

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

*Future analogies may be added here as the architecture develops.*
