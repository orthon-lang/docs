---
quick_id: 261023-gx3
status: complete
date: 2026-10-23
---

# Summary 261023-gx3: purity of contract expressions

- CONTRACTS.md: new subsection "What a Condition May Call" (decision note, what a condition may contain, the check, the closed list, members of a contract, bound and dyn, examples); Open Question 7 resolved inline; Decision History entry.
- CORE_CONCEPTS.md: Resolved Questions field. EDR-098 item 4: one pointer sentence.
- Consequence to watch: a generic function cannot call members of its bound in conditions in v0.1 (example `show<Printable as T>` is an error); lifted by the effect model in v0.2.
- The closed list is part of the standard library boundary (LIBRARY_BOUNDARY.md); not edited there.
