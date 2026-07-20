# Technology Stack

**Analysis Date:** 2026-07-20

## Languages

**Primary:**
- Markdown - All documentation content (91 `.md` files, 2.8M total)

**Metadata/Config:**
- YAML (implicit in `.gitignore` patterns) - Version control configuration

## Runtime

**Environment:**
- Git-based documentation repository — no runtime execution

**Version Control:**
- Git — local and remote (`origin` → GitHub)

## Frameworks

**Documentation:**
- None — pure Markdown files without build framework (no mkdocs, Sphinx, Hugo, or similar)

**Build/Dev:**
- None — documentation is static Markdown intended for direct GitHub browsing or manual processing

## Key Dependencies

**None detected** — This is a design-documentation-only repository with no software dependencies, package managers, or external libraries.

## Configuration

**Version Control:**
- `.git/` — Standard Git repository
- `.gitignore` — Python-focused patterns (legacy; no Python tooling present in repo)

**Hosting:**
- GitHub remote: `git@github.com:orthon-lang/docs.git`

## Documentation Structure

**Organization Method:**
- Why → How → What (Golden Circle framework)
- Directories: `why/`, `what/`, `how/`, `when/`, `notes/`, `.planning/`
- See `AGENTS.md` for complete directory mapping

**Key Entry Points:**
- `README.md` — Project overview and getting started
- `AGENTS.md` — AI agent contribution protocol and document map
- `why/VISION.md` — Core philosophy and pillars
- `how/decision_records/INDEX.md` — Engineering Decision Records journal

## Platform Requirements

**Development:**
- Text editor with Markdown support
- Git client (any version)
- No build tools, compilers, or runtimes required

**Content Consumption:**
- GitHub web interface (for browsing)
- Standard Markdown renderer
- Optional: local git clone for offline access

## Notes

- **No build artifacts** — Repository contains only source Markdown files
- **No external tooling dependencies** — Documentation is self-contained
- **Purpose:** Design documentation for the Orthon programming language (language design only; no compiler/runtime code present)

---

*Stack analysis: 2026-07-20*
