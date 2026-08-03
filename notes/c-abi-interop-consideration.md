# C ABI / FFI Interoperability Consideration

> **Note:** Consideration captured from an external discussion (2026-08-03)
> about why new systems languages fail to replace C. Records the problems
> and proposed solutions so the discussion is not lost.
>
> **Status:** Deferred — action at M2 (Standard Library & FFI) / M4
> (Implementation). This is implementation-strategy and tooling territory,
> not semantic-core or syntax design.
>
> **See also:**
> [`../when/ROADMAP.md`](../when/ROADMAP.md) (§ M2, item 8.2 — FFI & Interoperability; § M4 — Implementation),
> [`../how/strategies/IMPLEMENTATION_STRATEGIES.md`](../how/strategies/IMPLEMENTATION_STRATEGIES.md),
> [`../how/strategies/DEFAULT_STRATEGY.md`](../how/strategies/DEFAULT_STRATEGY.md),
> [`../why/VISION.md`](../why/VISION.md) (§ Execution Program)

---

## Context

Source: an external discussion arguing that the gap in systems-language
design is a language which, from the point of view of any other language,
is indistinguishable from C — so that libraries written in it are usable
by everyone, not just its own users. The full argument was pasted into the
2026-08-03 conversation.

## Problem 1 — Bespoke ABI creates a one-way FFI (the island problem)

New systems languages typically define their own ABIs and calling
conventions. An "FFI" makes the language useful in one direction only:
you can call C libraries from your language, but nobody else can call your
functions from theirs. A library written in your language is a little
island, usable exclusively by users of your language.

Every relevant language already knows what a C function is, because it had
to implement an FFI to be useful. The platform C ABI is the common
denominator of the entire desktop/server/mobile stack. The analogies in the
source discussion are pointed: a website written in a language no browser
supports; a paper written in Esperanto.

## Problem 2 — FFI wrapper / binding burden

Even if a language uses the platform C ABI unchanged, calling an existing C
library requires translating its header definitions into the calling
language's syntax. With hundreds or thousands of usable C libraries already
installed, hand-written bindings are a dreadful use of time.

A C header is not plain code to read: it must be preprocessed, and
inlining and internal/external linkage must be handled. Doing this properly
means understanding not only the C ABI but the C language itself.

## Proposed Solutions

1. **Platform C ABI as the native ABI.** The language's ABI *is* the
   platform C ABI, so functions written in the language are callable with
   the existing FFI machinery of any other language — no magic extensions
   or wrapper libraries.
2. **Parse C headers natively.** The compiler parses C headers
   (including the C preprocessor and linkage rules) and generates bindings
   programmatically, preferably transparently, so calling a C library needs
   no separate binding-generation stage.
3. **Emit C headers for own exports.** A library written in the language
   ships a C header with compatible definitions, so C — and by extension
   every other language — can call it through the same mechanism already
   built for C interop.

## Implications for Orthon

- **Fits the Implementation Strategy model.** The ABI is a *how* decision,
  not a semantic one. A "platform C ABI" profile can be a default strategy
  without constraining the Core Language or its syntax.
- **Concrete mechanism: a new ABI Policy type.** In the strategy model
  (`how/strategies/IMPLEMENTATION_STRATEGIES.md`) each Policy type is a
  declarative choice within one domain-specific area. The catalogue
  currently has no ABI or representation policy (as of 2026-08-03), so
  this consideration maps to introducing one:

  ```
  ABI Policy (new) → PlatformCAbi | NativeOrthonAbi | WasmAbi
  ```

  "Platform C ABI as native ABI" becomes the value `PlatformCAbi` in a
  strategy profile (e.g. the Default Strategy). Semantics are unchanged;
  the strategy stays declarative and orthogonal. Explicit Semantics is
  preserved because crossing the boundary is a visible operation
  (an `extern`-style declaration or a build-time binding), and the
  Execution Program model is preserved because the ABI is part of the
  execution contract.
- **Boundary is a representation subset, not a semantic change.** The C
  ABI does not touch Identity/Ownership/Mutation/Evaluation/Visibility/
  Lifetime — it fixes how exported values are laid out in memory (struct
  layout, alignment, calling convention, linkage). Values with no C
  mapping (e.g. a lazy `Sequence`) cannot cross the boundary directly;
  they require materialisation or a callback/handle convention. Ownership
  has no ABI meaning, so the ownership contract at the boundary must be
  documented (e.g. in the emitted C header).
- **Natural home:** `when/ROADMAP.md` § M2 item 8.2 (FFI & Interoperability:
  C ABI, embedding API); compiler-side work belongs to M4 (Implementation).
  C-header parsing and emission are compiler/tooling capabilities, not
  language semantics.
- **Consistent with the vision:** the Execution Program model (a program as
  a fully-defined artifact), explicit semantics, and the "not an island"
  positioning in `why/VISION.md`.
- **Currently unrecorded:** no strategy document mentions ABI/FFI as of
  2026-08-03, and the strategy catalogue has no ABI Policy type. This note
  preserves the consideration so it is not lost between spec freeze (M1)
  and implementation (M2/M4).

## Deferral

**Deliberately deferred.** We are currently designing the semantic core and
syntax (M1, Phases 2-5). These problems concern compiler implementation and
Implementation Strategy. They must not pollute semantic-core or syntax
design, and no ABI decision should be introduced into
`DEFAULT_STRATEGY.md` or other strategy documents during M1.

Action points: review this note when M2 (Standard Library & FFI) planning
begins; resolve the open questions below at that point.

## Open Questions

1. Does the language itself need a minimal FFI / foreign-declaration
   surface (e.g., `extern` declarations, foreign types)? Or can C interop be
   handled entirely by build-time tooling, leaving the language with zero
   FFI syntax? — decide at M2; keep out of Phase 4/5 for now.
2. Which ABI Policy value does each Implementation Strategy use? Is
   `PlatformCAbi` the Default Strategy's ABI value (see the ABI Policy
   type proposed in Implications for Orthon)?
3. Is C-header emission a compiler feature or a separate tooling step?
4. Is C-header parsing a compiler capability or a pre-compilation tool?
   Transparent single-stage vs. separate binding-generation stage?

## Source

External discussion, captured verbatim in the 2026-08-03 conversation.
No URL available at capture time.
