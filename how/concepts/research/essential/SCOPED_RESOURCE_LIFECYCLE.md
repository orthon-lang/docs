# Scoped Resource Lifecycle

> **⚠️ DRAFT — This document is a preliminary hypothesis derived from imperative crutch analysis.**
> It has not passed Concept Design Review.
>
> **Source:** `imperative-crutch-resource-management.md`

## Issue (Why)

Manually opening and closing files, connections, and handles forces the
programmer to write `try/finally`-style cleanup code that easily misses
`close()` in case of an exception, leading to resource leaks. Every
manual `close()` call is a potential leak site.

Orthon should eliminate resource leaks by providing a built-in scoped
resource management construct that guarantees release at scope exit.

## Examples

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `f = open(...); try: ... finally: f.close()` | `with open(...) as f:` |
| Java | `br = null; try { ... } catch {...} finally { if(br != null) br.close(); }` | `try (BufferedReader br = ...)` |

## Hypothesis

Orthon can eliminate `try/finally`-style cleanup by providing a built-in
scoped resource management construct (RAII via ownership, context-manager
blocks, or `defer` statements) that guarantees release at scope exit.

## Implications for Orthon

- RAII / context manager / defer — the language must provide a declarative
  mechanism for guaranteed resource release.
- The compiler should warn or forbid using a resource without an explicit
  lifetime scope.
- The mechanism must be built-in, not library-based, so that everyone uses
  it, not just "those who know."
- Common pattern: acquire → use → guaranteed release.
- Related concepts: `OWNERSHIP.md`, `EXECUTION_PROGRAM.md`

## Open Questions

- Should RAII (ownership-based) or explicit scope blocks (`using`) be
  primary? Both?
- How to handle resources that outlive their scope intentionally (e.g.,
  returning a file handle from a function)?

## Decision History

Initial hypothesis derived from `imperative-crutch-resource-management.md` — no decisions recorded yet.

## Affected Documents

- [ ] `what/CORE_CONCEPTS.md`
- [ ] `essential/OWNERSHIP.md`
- [ ] `essential/EXECUTION_PROGRAM.md`
- [ ] `what/GLOSSARY.md`
