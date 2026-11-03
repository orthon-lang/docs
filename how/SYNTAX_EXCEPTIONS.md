# Syntax Exceptions Registry

> **Established by:** [EDR-100](decision_records/process/EDR-100-syntax-exceptions-registry.md) (2026-11-02).
> **See also:** [`SYNTAX_PIPELINE.md`](SYNTAX_PIPELINE.md), [`what/SYNTAX.md`](../what/SYNTAX.md), and [`how/syntax/README.md`](syntax/README.md).

## Purpose and Authority

This registry makes departures from Syntax Principle 2 — **one syntactic
element, one responsibility** — explicit and reviewable. It is a process-control
index, not a second syntax specification. Canonical meaning remains in linked
`what/syntax/` or `what/concepts/` records and their deciding EDRs.

An unregistered exception is a **design defect**, not an implicit allowance.
Stage 3 of the [Syntax Pipeline](SYNTAX_PIPELINE.md#stage-3--coupling--overload-check--syntax-exceptions-registry-check-syntax-specific)
must find an entry before syntax with a context-dependent role is accepted.

## Statuses

| Status | Meaning | Acceptance effect |
|--------|---------|-------------------|
| **ACCEPTED EXCEPTION** | A deliberately retained mirrored or context-sensitive role with a bounded rule. | May be used only within its recorded boundary. |
| **OPEN CONFLICT** | A collision or parsing ambiguity remains unresolved. | Blocks acceptance of the proposed additional role until resolved, rejected, or explicitly deferred. |
| **RESOLVED** | A prior collision was eliminated or reduced to one responsibility. | Retained as anti-memory; the withdrawn role is unavailable. |
| **REJECTED** | A proposed reuse was declined before acceptance. | Retained as anti-memory; the proposed role is unavailable. |

## Entry Contract

Every entry records the syntactic element, roles or contention, status,
permitted scope or decision rule, rationale, canonical sources, deciding EDR,
and review or sunset condition. When an Architecture EDR accepts, changes,
resolves, rejects, or deprecates an entry, it and the applicable accepted
syntax record must link back here; this entry must link to them in return.

## Active Entries

| Element | Roles or contention | Status | Permitted scope or decision rule | Canonical sources | Deciding EDR | Review / sunset condition |
|---------|---------------------|--------|----------------------------------|-------------------|--------------|---------------------------|
| Prefix `*` | Collects a remainder in receiving positions; spreads values in sending positions; may later contend with numeric multiplication. | **ACCEPTED EXCEPTION** | The two roles are permitted only as mirror-image construct/destruct forms. Numeric multiplication remains undecided. | [`DESIGN_PRINCIPLES.md`](DESIGN_PRINCIPLES.md#pola-principle-of-least-astonishment), [`UNPACKING.md`](../what/concepts/UNPACKING.md) | [EDR-096](decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) | Revisit when arithmetic operators are specified. |
| `\|` | Separates variant constructors in an ADT; combines existing types in a union type. | **ACCEPTED EXCEPTION** | Each role is limited to its declared type form; no third role without review. | [`UNION_INTERSECTION_TYPES.md`](../what/concepts/UNION_INTERSECTION_TYPES.md) | [EDR-045](decision_records/architecture/EDR-045-union-intersection-types.md) | Revisit if either type form changes. |
| `\|\|` | Used for short-circuit logical OR in smart-cast analysis; proposed as a distinct ADT variant separator to remove the `\|` overload. | **OPEN CONFLICT** | Logical OR remains the documented role; ADT separation with `\|\|` is a hypothesis only and must not be treated as accepted syntax. | [`SMART_CAST.md`](../what/concepts/SMART_CAST.md), [`ADT_VARIANT_SEPARATOR_SYNTAX.md`](syntax/ADT_VARIANT_SEPARATOR_SYNTAX.md) | [EDR-060](decision_records/architecture/EDR-060-smart-cast.md) documents logical OR; no EDR decides the proposed ADT role. | Resolve the separator and logical-OR coupling through the [syntax hypothesis](syntax/ADT_VARIANT_SEPARATOR_SYNTAX.md) before changing the accepted `\|` exception. |
| `as` | Names a generic type parameter from a bound; attaches an explicit label to a value or pattern. | **ACCEPTED EXCEPTION** | Both forms name the item on the right; casts and conformance words must not reuse `as`. | [`GENERICS_SYNTAX.md`](../what/syntax/GENERICS_SYNTAX.md), [`UNPACKING.md`](../what/concepts/UNPACKING.md) | [EDR-086](decision_records/architecture/EDR-086-generics-syntax-revision.md), [EDR-096](decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) | Revisit before accepting another `as` form. |
| `!` | Error-union type constructor; forced unwrap of `Option`; proposed negative bounds. | **OPEN CONFLICT** | No additional role may be accepted. Negative bounds remain unaccepted. | [`ERROR_UNION.md`](../what/concepts/ERROR_UNION.md), [`NULL_SAFETY.md`](../what/concepts/NULL_SAFETY.md), [`ERROR_REPRESENTATION_UNIFICATION.md`](concepts/research/essential/ERROR_REPRESENTATION_UNIFICATION.md) | [EDR-023](decision_records/architecture/EDR-023-error-union.md) | Resolve with error-representation and negative-bound syntax decisions. |
| `[]` | Collection-literal candidate; indexing and slicing syntax; custom indexer candidate. | **OPEN CONFLICT** | Indexing and slicing are accepted. Literal and custom-indexer spellings require a Stage 3 decision before final grammar. | [`INDEXING.md`](../what/concepts/INDEXING.md), [`COLLECTION_LITERAL_SYNTAX.md`](../what/concepts/COLLECTION_LITERAL_SYNTAX.md), [`INDEXERS.md`](concepts/research/deferrable/INDEXERS.md) | [EDR-041](decision_records/architecture/EDR-041-collection-literal-syntax.md), [EDR-082](decision_records/architecture/EDR-082-1-based-indexing.md) | Resolve during collection-literal and indexer syntax decisions. |
| `->` | Function return annotation; function type; lambda body; rejected generator-expression ordering. | **OPEN CONFLICT** | No new arrow role before coupled function-type and lambda syntax questions are resolved. | [`FUNCTION_RETURNING_FUNCTION.md`](syntax/FUNCTION_RETURNING_FUNCTION.md), [`GENERATOR_EXPRESSION_SYNTAX.md`](../what/syntax/GENERATOR_EXPRESSION_SYNTAX.md) | [EDR-092](decision_records/architecture/EDR-092-generator-expression-syntax.md) | Resolve in the coupled function-return, function-type, and lambda decisions. |
| `:` | Type-annotation candidate; former range-step, generic-bound, call-label, and block-opener uses; potential map-literal use. | **OPEN CONFLICT** | Withdrawn roles stay unavailable. Map-literal and final annotation grammar remain subject to Stage 3. | [`TYPE_ANNOTATION_SYNTAX.md`](syntax/TYPE_ANNOTATION_SYNTAX.md), [`RANGE_SYNTAX.md`](../what/syntax/RANGE_SYNTAX.md), [`COLLECTION_LITERAL_SYNTAX.md`](../what/concepts/COLLECTION_LITERAL_SYNTAX.md) | [EDR-083](decision_records/architecture/EDR-083-range.md), [EDR-086](decision_records/architecture/EDR-086-generics-syntax-revision.md), [EDR-096](decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) | Resolve when annotations and map literals receive final grammar. |
| `{}` | Explicit scope block; map or set literal candidates. | **OPEN CONFLICT** | Literal grammar must explicitly disambiguate or select another form. | [`COLLECTION_LITERAL_SYNTAX.md`](../what/concepts/COLLECTION_LITERAL_SYNTAX.md), [EDR-076](decision_records/architecture/EDR-076-reject-significant-whitespace.md) | [EDR-041](decision_records/architecture/EDR-041-collection-literal-syntax.md) | Resolve during collection-literal syntax acceptance. |
| `<>` | Generic parameter and bound delimiters; lexical contention with `<` and `>` comparisons. | **OPEN CONFLICT** | Generics use `<>`; parser disambiguation from comparisons must be specified before grammar freeze. | [`GENERICS_SYNTAX.md`](../what/syntax/GENERICS_SYNTAX.md) | [EDR-086](decision_records/architecture/EDR-086-generics-syntax-revision.md) | Resolve in the concrete parser grammar. |

## Historical Negative Decisions

| Element | Withdrawn role | Status | Decision rule | Canonical sources | Deciding EDR | Retention reason |
|---------|----------------|--------|---------------|-------------------|--------------|------------------|
| `..` | Dotted rest syntax (`..rest` and brace-rest forms). | **REJECTED** | `..` remains the range operator only; rest uses prefix `*`. | [`UNPACKING.md`](../what/concepts/UNPACKING.md), [`RANGE_SYNTAX.md`](../what/syntax/RANGE_SYNTAX.md) | [EDR-096](decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) | Prevents a future rest proposal from reusing range syntax. |
| `?` | Ignore-pattern marker. | **REJECTED** | `_` is the one-position ignore pattern; `?` is unavailable for that role. | [`UNPACKING.md`](../what/concepts/UNPACKING.md), [`NULL_SAFETY.md`](../what/concepts/NULL_SAFETY.md) | [EDR-096](decision_records/architecture/EDR-096-redesigned-unpacking-syntax.md) | Preserves the recorded rejection of a fifth question-mark role. |

## Maintenance

The registry is consulted at Stage 3 for every syntax decision. A registry
entry is updated in the same change that changes its decision or canonical
source. Phase 5 review must leave no **OPEN CONFLICT** unclassified: each is
resolved, rejected, or explicitly deferred with a review trigger.
