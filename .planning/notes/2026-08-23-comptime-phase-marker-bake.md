---
date: "2026-08-23 00:00"
promoted: false
---

## Compile time phase marker: `bake` at the call site

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

### Considered and rejected: `@bake` (2026-08-23)

Proposal: use `@bake` as the phase marker by analogy with `@macro`
(EDR-029). Rejected — three reasons:

1. **Different granularity and position.** `@macro` is a declaration-site
   annotation on a function (`@macro fun generate(...)`); `bake` is a
   call-site/expression marker (`TABLE = bake generate_table(256)`). Per
   the three-axis thesis, `@macro` is the specialised case of a comptime
   function; the general phase marker must cover expressions and blocks,
   so the analogy does not transfer.
2. **Axis conflation.** `@` is the Metadata Protocol (axis 2 — *what
   structure*); the phase marker is axis 3 (*when it runs*). `@bake`
   would give `@` two meanings depending on the following token
   (`@typeInfo` vs `@bake`) — exactly what Semantic Purity forbids, and
   the axis conflation the thesis dissolves. `@` remains blocked for the
   phase marker (OQ7).
3. **The block form.** The irreducible comptime form is a block; `@bake
   { ... }` does not fit the established `@` grammar (`@name(args)` or
   `expr@op`). One keyword `bake` covers both `bake expr` and
   `bake: block` with a single grammar; `@bake` would require two marker
   styles for one concept.

Consistent alternative acknowledged: a *declaration-site* marker for a
general non-macro comptime function could be `@`-annotated like `@macro`
(e.g. `@bake fun f(...)`). But per the thesis that is sugar (function
comptime = all parameters comptime + body comptime), and the call-site
model makes such a marker unnecessary — the colourless function's phase is
visible at the call site.

**Conclusion:** `bake` stays a keyword at the call site (expression +
block), not `@bake`.

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
