# Agent Environment — Workstation & Research Notes

> Environment-specific notes for this workstation. They are **not** project rules — they
> record how the tooling actually behaves here so that work is not blocked.
>
> **Referenced from:** [`AGENTS.md`](../AGENTS.md) §12. Verified 2026-09-18 – 2026-09-27.

## macOS CLI

- BSD `cat` has no `-A` flag ("illegal option -- A"). Use `cat -et` (= `-vET`): tabs
  become `^I`, line ends `$`, non-printing characters become visible.
- Do not add the npx cache directory (`~/.npm/_npx/<hash>/node_modules/.bin`) to the
  permanent `PATH` — the path is ephemeral and may be cleaned. Symlink the required
  binary into `~/.local/bin` (already on `PATH`) instead.
- PDF text extraction: `pdftotext`, `mutool`, `gs`, and `qpdf` are not installed here.
  Use `pip install pypdf` and `pypdf.PdfReader(path).extract_text()`.

## Researching Paywalled Sources

- Substack: probe `https://<domain>/api/v1/posts/<slug>`. The JSON reports `audience`
  (`only_paid` / `everyone`), `truncated_body_text`, and `wordcount`, which reveals
  whether the text is truncated before an ingest plan is built.
- `web.archive.org` does not bypass a Substack paywall, and `r.jina.ai` frequently
  returns "Failed to extract meaningful content" on such pages — do not rely on it.
- To find mirrors, use `https://html.duckduckgo.com/html/?q=...` (the plain
  `duckduckgo.com/html` endpoint returns an interstitial).
- When an article is paid but its images are served openly, the poster often carries the
  payload: fetch originals from `<bucket>.s3.amazonaws.com/public/images/<uuid>_<WxH>.<ext>`.
  The `substackcdn.com` variant is a compressed progressive JPEG; the S3 original is
  lossless.
