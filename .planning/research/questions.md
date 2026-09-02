# Research Questions

## Symbol-role inventory (2026-08-15)

**Q:** Which symbols are currently overloaded (`:`, `*`, `{}`, and any
others), what is each symbol's single semantic role, and is there already an
EDR lineage for Semantic Purity's wording?

**Why:** Before drafting the Tier 1 EDR that amends Semantic Purity to "one
symbol = one semantic role", the complete inventory of symbols and their
roles is needed so the amendment can enumerate roles explicitly.

**Acceptance:** A table listing every symbol, its role(s) as documented
today, and any conflicts (e.g. `*` = pack/unpack vs POLA's prohibition of
context-dependent role reversal).

## QTT and the semantics/mechanism boundary (2026-09-02)

**Q:** Does the semantics/mechanism boundary — "single owner is
guaranteed; how it is verified is a Strategy decision" — undermine
Orthon's bet that LLMs can generate correct code without a borrow
checker? If an LLM is not required to prove `1`-multiplicity at
generation time, where is an ownership violation caught at all — and does
the default CoW strategy eliminate the error class entirely (plain data
copies; only the resource subset is governed by ownership), making
Orthon's answer stronger than QTT-level typing rather than weaker?

**Why:** QTT (quantitative typing) is the formal counterpart of the
single-owner invariant, but Orthon grounds ownership in Separation Logic
and defers enforcement to Strategy (EDR-061 accepts CoW + `shared`; no
borrow checker). Before deciding whether QTT has any role (soundness
model, `HIGH_PERFORMANCE_STRATEGY` Lifetime Policy, or nothing), the
boundary itself needs scrutiny: is the ownership-violation error class
removed by construction, or merely relocated to the resource subset?

**Acceptance:** A reasoned position on (a) whether the CoW+`shared`
default genuinely removes the ownership-violation error class rather than
moving it, and (b) whether QTT adds anything over the Separation Logic
framing as a soundness argument — with a recommendation for
`how/strategies/` vs `how/concepts/research/`.
