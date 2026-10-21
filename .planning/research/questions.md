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

## Per-mechanism evidence base for core constructs (2026-10-18)

**Q:** For each of Orthon's core mechanisms (algebraic data types, traits,
error handling, ownership/mutability), which Programming Language Theory
works and which documented language experience (specifications, RFCs, design
documents, retrospectives) exist, and does any of it include empirical
evidence rather than theory alone?

**Why:** `notes/evidence-based-software-engineering.md` ("Where They Diverge")
states that no external empirical studies of Orthon's constructs exist. That
holds for Orthon as a whole, but whether it holds for the individual
mechanisms is unconfirmed: the existence of such studies was asserted from
model memory only (unresolved: unverifiable, not checked against a primary
source). The answer determines how realistic the Evidence Survey step (see
[[2026-10-18-evidence-survey-step-design]]) is and what its sufficiency
criterion should be.

**Acceptance:** A table per mechanism listing candidate sources, each tagged
by class (theory / language experience), evidence kind (formal result /
empirical / experience report) and verification status (admitted with source /
unresolved with reason); plus an explicit "no evidence found" entry where
nothing was located; plus a recommendation on whether to revise the "Where
They Diverge" claim. Pilot on one concept first.

## Choosing between a down-shift and a sideways shift when a concept is too complex (2026-10-21)

**Q:** When the implementation complexity of a concept signals that it must
be reconsidered, what criterion decides between moving down (extract a common
kernel from which the constructs derive) and moving sideways (give the
complexity to another carrier: compiler, library, another layer)?

**Why:** The 2026-10-21 exploration (`notes/2026-10-21-complexity-as-redesign-signal.md`)
settled that a complexity signal triggers redesign in another plane and that
both kinds of shift are allowed, "by the situation". Without a criterion, two
authors facing the same concept can reach different designs. Existing
instances to learn from: the control-transfer gap (down), named arguments via
macros versus a compiler built-in (sideways), `comptime T: type` versus `<>`
(sideways, one mechanism kept).

**Acceptance:** A short decision rule or checklist that classifies each
existing instance above correctly, tested on at least three further cases from
the 58 accepted concepts, plus an explicit statement of when the rule does not
decide.
