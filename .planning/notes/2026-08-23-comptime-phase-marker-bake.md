---
date: "2026-08-23 00:00"
promoted: false
---

## Comptime phase marker: `bake` at the call site

### The question

Can comptime be modelled as a separate execution context — a fifth policy
of the `execution_context` primitive alongside `defer`/`delegate`/`spawn`/
`fork`? Explored via `$gsd-explore`, starting from the accepted
`execution_context` primitive (EDR-085) and the accepted comptime concept
(EDR-031).

### Finding: comptime is an orthogonal phase axis, not a fifth context

Plan B confirmed. The "hand control to the compiler" analogy does not
hold: the compiler is not a runtime Execution Context (no state object, no
mailbox, no runtime lifetime, no materialisation vocabulary
`take`/`await`/`next`). Comptime answers *when* code runs (phase);
`execution_context` answers *how* it runs (policy). Re-confirms OQ9 in
`what/concepts/COMPILE_TIME_EXECUTION.md`.

### Phase marker: position and keyword

- **Position — the call site, not the declaration.** Functions are
  colourless: `TABLE = bake generate_table(256)`;
  `process(bake generate_table(64))`. Comptime safety (no IO/FS/network)
  is checked transitively at the call site.
- **Keyword — `bake` (candidate).** Symbols are blocked by Semantic
  Purity + loaded meanings: `#` (line comment + unhygienic macro escape,
  EDR-029), `@` (Metadata Protocol), `<>` (generics, EDR-086). `const`
  rejected (immutable-by-default makes it redundant; the expression-level
  marker is still needed for asserts/args). `consteval` is the fallback
  (exact semantics but C++ baggage). `static` is loaded (Allocation
  Policy); `macro` loaded (EDR-029).
- **`const TABLE = expr` rejected** — since the expression-level marker
  is required for the general case, no separate `const` binding concept.
  Resolves OQ3 in `what/concepts/DECLARATION_BY_ASSIGNMENT.md`.

### Tension flagged

EDR-031 Principle 5 ("marker at the definition site") conflicts with the
call-site position. Any change requires an EDR amendment with Human
Sign-off (AGENTS.md §7.4) — not changed here. All additions recorded as
draft, feeding Phase 5.

### Related

- [[apply-comptime-bake-decision]] — apply the decision (resolve OQs,
  draft the EDR-031 amendment with Human Sign-off).
- `what/concepts/COMPILE_TIME_EXECUTION.md` — draft notes on OQ7/OQ9,
  Synthesis, Decision History (2026-08-23).
