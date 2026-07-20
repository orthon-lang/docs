# Phase 1: Concerns Remediation — Discussion Log (Assumptions Mode)

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions captured in CONTEXT.md — this log preserves the analysis.

**Date:** 2026-07-20
**Phase:** 01-concerns-remediation
**Mode:** assumptions
**Areas analyzed:** Technical Approach (Content Depth, Fitness Functions), Scope Boundaries (Template Structure), Sequencing/Priority (Dependency Order), Quality Bar (Concept Depth)

## Assumptions Presented

### Technical Approach — Content Depth for Empty Architecture Specs
| Assumption | Confidence | Evidence |
|------------|-----------|----------|
| Fill arch specs with design specifications (interface contracts, key decisions), not implementation-level detail | Likely | `ARCHITECTURE.md` § Core Language, `EDR-010`, zero compiler code in repo |

### Technical Approach — Fitness Functions Catalogue Scope
| Assumption | Confidence | Evidence |
|------------|-----------|----------|
| Source fitness functions from existing latent patterns in codebase (DESIGN_PRINCIPLES.md's 27 principles, gates, EDR-005, EDR-004) rather than inventing from scratch | Confident | `EDR-005-fitness-functions.md`, `_language-design.md`, `DESIGN_PRINCIPLES.md`, `EDR-004` |

### Scope Boundaries — Template Structure (7 vs 8 Sections)
| Assumption | Confidence | Evidence |
|------------|-----------|----------|
| Use actual 8-section template; correct "7-section" errors in ROADMAP.md and CONCERNS.md | Confident | `docs/how/templates/_concept.md` (8 sections), `.claude/CLAUDE.md` (8 sections) |

### Sequencing/Priority — Dependency Order Among DEBT Items
| Assumption | Confidence | Evidence |
|------------|-----------|----------|
| Three-wave sequence: Wave A (content), Wave B (cleanup), Wave C (audits/process) | Likely | DEBT-11/14 depend on template knowledge (Wave A). DEBT-13 should catch DEBT-06 fixes |

### Quality Bar — Under-Developed Concepts
| Assumption | Confidence | Evidence |
|------------|-----------|----------|
| Bring to structurally complete drafts (8 sections with content), defer full design to Phase 3 | Likely | ROADMAP success criterion #6 ("or remediation plan"), ALLOCATION.md partial content |

## Corrections Made

No corrections — all assumptions confirmed.

## External Research

No external research required — codebase provided sufficient evidence for all assumptions.
