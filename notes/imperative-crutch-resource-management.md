# Imperative Crutch: Resource Management

## Problem

Manually opening and closing files, connections, and handles — code easily
misses `close()` in case of an exception, leading to resource leaks.

## Examples

### Opening and closing a file

| Language | Crutch | Modern |
|----------|--------|--------|
| Python | `f = open(...); try: ... finally: f.close()` | `with open(...) as f:` (context manager) |
| Java | `br = null; try { ... } catch {...} finally { if(br != null) br.close(); }` | `try (BufferedReader br = ...)` (try-with-resources) |

## Implications for Orthon

- RAII / context manager / defer — the language must provide a declarative
  mechanism for guaranteed resource release.
- The compiler should warn or forbid using a resource without an explicit
  lifetime scope.
- The mechanism must be built-in, not library-based, so that everyone uses
  it, not just "those who know."
- Common pattern: acquire → use → guaranteed release.

**See also:** concepts `OWNERSHIP.md`, `EXECUTION_PROGRAM.md`
