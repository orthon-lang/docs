# Ownership — Post-Acceptance Open Questions

> **Status:** Open tracking note. Accepted baseline: [EDR-013](../how/decision_records/architecture/EDR-013-semantic-model.md) and the [semantic model](../what/SEMANTIC_MODEL.md). Working review decisions and hypothesis links: [Ownership research map](../how/concepts/research/essential/OWNERSHIP.md).

| Question | Why it matters | Options and evidence | Provisional route | Status |
|---|---|---|---|---|
| How do markers and composition determine copying and resource responsibility? | Generic containers must not duplicate resources | [Type categories](../how/concepts/research/essential/OWNERSHIP_TYPE_CATEGORIES.md) | B; A if a genuinely new concept is proposed | Open |
| When and how is ownership transferred? | Prevent use after move and double cleanup | [Transfer](../how/concepts/research/essential/OWNERSHIP_TRANSFER.md) | B; syntax pipeline for spelling | Open |
| How is exclusive access proven? | `proc` and iteration must preserve shared XOR exclusive access | [Borrowing](../how/concepts/research/essential/OWNERSHIP_BORROWING_EXCLUSIVITY.md) | B; C if a conflict is demonstrated | Open |
| How are returned views linked to owners and scopes? | Views and iterators must not outlive data | [Borrow scopes](../how/concepts/research/essential/OWNERSHIP_BORROW_SCOPES.md) | B; C if a conflict is demonstrated | Working direction; details open |
| How are resource cleanup and foreign access managed? | Prevent invalid access and duplicate cleanup | [Lifecycle](../how/concepts/research/essential/OWNERSHIP_RESOURCE_LIFECYCLE.md) | B; C if a conflict is demonstrated | Open |

Routes are provisional: the scope of each actual resolution determines its tier. Type B uses the lightweight documented decision process; a semantic amendment needs the appropriate review and human sign-off before an EDR. Type C requires a concrete conflict between accepted concepts, not merely an open interaction; record any demonstrated conflict in the registry. Type A follows the full concept pipeline. No EDR or accepted specification is amended by this note.
