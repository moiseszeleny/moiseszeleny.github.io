# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Quarto website/blog** ("Physics boosted with LLMs") — a bilingual (English/Spanish)
personal research log documenting the use of LLMs in theoretical/particle-physics (BSM
phenomenology) work. Audience is fellow researchers. There is no application code; the repo
is content (`.qmd`), a custom theme (`theme.scss`), and site config (`_quarto.yml`).

## Commands

```bash
quarto preview              # live local server with hot reload — use while editing
quarto render               # build the whole site into docs/
quarto render about.qmd     # build a single file (faster when iterating on one page)
quarto check jupyter        # verify the Python engine is wired up (needed for code cells)
```

There is no separate build/lint/test step — `quarto render` is the build and the check.
Deployment is automatic (see below); **do not** run a manual publish command.

## Deploy pipeline — how the site goes live

- **`_quarto.yml` sets `output-dir: docs`**, but **`docs/` is git-ignored and NOT tracked.**
  Do not commit `docs/`. It exists locally only for previews.
- Pushing to `main` triggers `.github/workflows/publish.yml` ("Publish site"), which runs
  `quarto render` on a runner and deploys via the **native GitHub Pages artifact**
  (`actions/deploy-pages` — Pages "build type" is `workflow`, source = GitHub Actions, not a
  branch). Live at https://moiseszeleny.github.io/.
- Everyday workflow: edit `.qmd` → `git commit` → `git push`. The Action renders and
  deploys. Verify a run with `gh run watch <id> --exit-status`.

## The freeze contract (critical for any executed code cell)

The CI runner has **only Quarto — no Python/Jupyter/matplotlib.** Any page with an executed
code cell (e.g. a `{python}` block) must therefore ship **pre-computed, frozen output**:

- Posts inherit `freeze: true` from `posts/_metadata.yml`. Root pages that execute code
  (e.g. `about.qmd`) set `execute: freeze: auto` in their own front matter.
- Frozen output lives in **`_freeze/`** (tracked, not ignored). It is (re)generated when you
  `quarto render` a page **locally** (where Python/matplotlib/Jupyter are available).
- **When you change an executed cell, you MUST re-render locally and commit `about.qmd`/the
  `.qmd` *and* its updated `_freeze/` files together.** If you forget, the CI render fails
  (it tries to execute and has no Python) — a useful early warning, not a silent stale figure.
- Because of this, figures are styled to the site palette in-code (slate accent `#3a5a7a`,
  paper bg `#fcfcfa`, serif) — see the `{python}` cell in `about.qmd` for the pattern.

## Content conventions

- **Posts** live in `posts/<slug>/index.qmd`. Shared post defaults are in
  `posts/_metadata.yml` (freeze, reading-time, no title banner, per-post TOC).
- **Bilingual rule:** one primary language per post, with a short abstract in the other
  language at the top wrapped in a `::: {.abstract-alt}` div (styled in `theme.scss`).
  `posts/why-this-log-exists/index.qmd` is the reference example.
- **Language badge:** the post's language is its **first `categories:` entry** — literally
  `English` or `Español` — followed by topic tags. The listing (`index.qmd`) shows these as
  pills; `_includes/language-badges.html` (a JS `include-after-body`) tags the
  language pills with a `.lang-badge` class that `theme.scss` styles distinctly from topic
  tags. Keep the language as the first category or the badge won't highlight.
- **Unpublished-results policy:** details tied to unpublished results are withheld until the
  corresponding preprint is public — document method/workflow, not the specific result.
- **Math:** LaTeX via MathJax (`html-math-method: mathjax`). Display equations get extra
  spacing + horizontal scroll from `theme.scss` (`mjx-container[display="true"]`).

## Theme (`theme.scss`)

A custom academic look split into Quarto's `/*-- scss:defaults --*/` (fonts, the color
palette variables — `$accent: #3a5a7a`, `$paper`, `$ink`, `$rule`, code surfaces) and
`/*-- scss:rules --*/` (component styling). Body is a system **serif** stack; UI/headings a
sans stack; no web-font fetch (offline/reproducible). Wired in via `format: html: theme:` in
`_quarto.yml`. Edit palette variables in the defaults block rather than hard-coding colors in
rules.

## Gotchas

- Pinned Quarto version in CI is `1.9.38` (`.github/workflows/publish.yml`) — match locally
  to avoid render drift.
- The About page uses the `trestles` layout without a profile image (intentional); several
  contact links in `about.qmd` are still `TODO` placeholders the author fills in.
