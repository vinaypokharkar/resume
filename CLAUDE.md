# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Personal career-document repo, not an application. No source code, no tests, no package manager. LaTeX resume + tailored application documents + YAML config consumed by an external job-search workflow.

## Build

```bash
pdflatex resume.tex               # local build (repo root, MiKTeX on this machine)
```

CI ([.github/workflows/build-resume.yml](.github/workflows/build-resume.yml)) runs on every push to `main`: `latexmk -pdf resume.tex`, moves the result to `docs/Vinay_Pokharkar_Resume.pdf`, and auto-commits it as `github-actions`. So `docs/` is CI-owned — don't hand-edit it. The root `resume.pdf` is separately tracked and is what a local `pdflatex` run overwrites; commit it alongside `resume.tex` so the two don't drift before CI catches up.

`*.aux` / `*.log` are gitignored, as is `temp/` (scratch: saved job-posting HTML, signed offer PDFs).

## Page-count is a hard constraint

`resume.tex` must stay **2 pages**. Adding or lengthening any entry can silently spill a near-empty third page. After every content change, read the page count off the build output:

```bash
pdflatex -interaction=nonstopmode resume.tex 2>&1 | grep -oE "Output written.*pages"
```

If it spills, the fix is content-level (tighten wording, fold a duplicated fact into an existing entry, or drop the weakest entry) — the layout has no slack left to tune. The Honours and Awards section was removed for this reason; the award now lives in the SeaGuard project bullet and the Summary.

## LaTeX structure

`resume.tex` and `internship_application.tex` are RenderCV-generated documents sharing an identical ~150-line preamble. Author content with the custom environments, not raw `itemize`/`tabular`:

- `twocolentry{RIGHT}` — entry title on the left, right-aligned second column. Used for dates (experience, education) and for links in Projects / Open Source (`\href{...}{Github}`, `{Live}`, `{Merged PRs}`).
- `onecolentry` + `highlights` — the bullet list under an entry. `\vspace{0.2 cm}` separates sibling entries.
- `threecolentry`, `highlightsforbulletentries`, `header` — also available, currently unused in `resume.tex`.

Escaping that matters here: `\%` inside URLs (the GitHub PR-search links use `%3A`), `\textasciitilde` for `~`, `\&` in prose.

`\placelastupdatedtext` is defined but never called in either file — the "Last updated in <Month Year>" string does not render. Editing it changes nothing visible.

Both `.tex` files carry large commented-out blocks (past internships at Connectwise / Cartman Labs / EaseworkAI, Publications, Copyright, Positions of Responsibility). These are a reservoir for tailoring a variant to a specific application — leave them in place.

## Content lives in four places — keep them in sync

The same facts are duplicated across:

| File | Role |
|---|---|
| [resume.tex](resume.tex) | The canonical 2-page resume |
| [cv.md](cv.md) | Longer Markdown CV — superset, includes entries cut from the resume for space |
| [profile.yml](profile.yml) | Structured profile: `narrative.proof_points`, `superpowers`, target roles, compensation |
| [internship_application.tex](internship_application.tex) | Variant tailored to one application (HENNGE), different section order |

When a fact changes — GitHub star count, a repo link, an end date, a new merged PR — it usually needs updating in more than one of these. `cv.md` and `profile.yml` drift most easily because nothing builds them.

## Verifying external claims

The resume states hard numbers (star counts, PR counts) that go stale. Check them against GitHub rather than editing by hand:

```bash
gh api repos/OWNER/REPO --jq .stargazers_count
gh search prs --author vinaypokharkar --merged --limit 40 --json number,title,url,repository
```

Note `gh search prs` uses `--merged`, not `--state merged`.

## Career-ops config

[profile.yml](profile.yml) and [portals.yml](portals.yml) are consumed by an external `/career-ops` workflow, not by anything in this repo. `portals.yml` holds `title_filter` (positive/negative title keywords — entry-level words like "Intern", "New Grad" are *positive*), `search_queries`, and `tracked_companies` (each entry requires a `careers_url`, branded URL preferred over the ATS-hosted one).
