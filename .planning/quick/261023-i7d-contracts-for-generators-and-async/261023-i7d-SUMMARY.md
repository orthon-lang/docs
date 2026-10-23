---
quick_id: 261023-i7d
status: complete
date: 2026-10-23
---

# Summary 261023-i7d: contracts for generators and async

- CONTRACTS.md: new subsection "Generators and Async Functions" (decision note, five rules, examples with errors, stage table); first bullet of Where Contracts Apply names the exception; Open Question 8 resolved inline; Decision History entry.
- CORE_CONCEPTS.md: Resolved Questions. EDR-098 item 4: one pointer sentence.
- Interpretation to confirm: a type that declares an invariant cannot declare an async proc or a generator proc in v0.1; async fun and async new stay allowed with requires only.
