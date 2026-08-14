# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this repo is

A [Quarto](https://quarto.org) website: the student-facing template for a Sonoma State
University accounting analytics course. Students clone it, personalize the identity
placeholders, complete the assignments in `posts/`, and deploy to Netlify.

**The posts are assignments in progress, not finished writing.** They are deliberately
incomplete and deliberately non-running. Read the next section before editing anything.

## Critical rules

### 1. Never fill in a `_____` blank

Assignments are blanked templates. The marker is a five-underscore token `_____`, and
there are several hundred across `posts/`. The student fills them in; we do not — unless
the user explicitly asks for help with a specific blank, in which case explain the
reasoning rather than silently editing the file.

The hint mechanism is a trailing comment:

```r
kmeans_spec <- k_means(num_clusters = _____) |>  # Use 3 clusters
  set_engine("_____") |>                          # Which engine?
```

Blanks are not always standalone tokens. They appear inside string literals and regex
quantifiers (`str_detect(account_code, "^\\d{_____}$")`) and spliced into identifiers
(`geom______()` is `geom_` + `_____`). A blind find/replace, autoformatter, or linter pass
will corrupt them.

### 2. Never make the code run

Every post sets `execute: eval: false`. Code that does not execute is the expected state,
not a bug. Do not flip `eval` to `true`, do not "repair" incomplete pipelines, and do not
report non-running chunks as defects.

### 3. Never move a data URL off the current term directory

Remote data is term-scoped. The current directory is `https://estanny.org/data/2026-fall/`.
Re-derive it rather than trusting this line after a term rollover:

```bash
grep -rho 'estanny.org/data/[^/]*/' posts/ | sort -u   # expect exactly one directory
```

This matters beyond correctness: the course's copy detector treats an `estanny.org` data
URL *without* the current term directory as a copying signal, because that is what a
copied prior-term post looks like. Rewriting a URL to an unversioned or older path can
flag an honest student for academic misconduct.

### 4. Posts are generated artifacts

Everything under `posts/` is regenerated each term from `build/<term>/templates/` in a
separate course repo (see commit `65cd0bb`). Answering a student's question about one post
is fine. Structural or term-wide changes are not — they belong upstream and should be
regenerated, or the two copies silently diverge.

### 5. Never commit drafts or solutions

`.gitignore` excludes `posts/_*.qmd` and `posts/*sol*.qmd` on purpose. Solution files
legitimately sit untracked in the working tree. Do not `git add -A` without checking, and
do not "helpfully" untrack them from `.gitignore`.

### 6. Leave the placeholder identity text alone

`"Your Name - Accounting Analytics"` in `_quarto.yml`, `youraddress@sonoma.edu`,
`linkedin.com/in/yourprofile`, and about.qmd's "Your name is a student at Sonoma State
University" are intentional. This is a template; the student personalizes it, not us.

## Layout

```
_quarto.yml          Site config: title, navbar, html format (cosmo theme, toc)
index.qmd            Home page + post listing (sort: date desc, categories: true)
about.qmd            Bio page (jolla template, profile.jpg)
styles.css           Empty stub — the place for custom CSS
posts/
  _metadata.yml      Applies to ALL posts: freeze: true, title-block-banner: true
  NN-topic.qmd       Assignment templates, currently 02–12
```

There is no README, no `data/` directory, and no build tooling. What you see is the whole
repo.

Posts are numbered `NN-topic.qmd`; the roster rotates each term, so read the directory
rather than trusting a list here. Two groups behave differently:

- **02–09** — regular HTML posts.
- **10–12** — Quarto dashboards (`format: dashboard`).

Filenames mix hyphens and underscores (`04_data_visualization.qmd` vs
`05-data-governance.qmd`). This is untidy but harmless; grading resolves topics by
substring, e.g. topic 6 by `"clustering"`. Do not mass-rename to normalize.

## Post conventions

### Front matter

Canonical block, used by 9 of 11 posts:

```yaml
---
title: "N: Title Case Topic"
date: today
execute:
  eval: false
  message: false
  warning: false
---
```

Dashboards add `format: dashboard`. Post 12 nests it to enable code folding:

```yaml
format:
  dashboard:
    code-fold: true
```

No post declares `categories:`, `image:`, `toc:`, or `description:`. Only post 12 has an
`author:`. `toc` and the theme come from `_quarto.yml`; `freeze` and the banner come from
`posts/_metadata.yml`.

### Setup chunk

Every post opens with one. This example encodes the whole convention — semantic label, one
`library()` per line with an aligned trailing comment, then display preferences and a seed:

```r
#| label: setup
# Load packages we need - matching the slides
library(tidyverse)    # For data manipulation
library(tidymodels)   # For clustering
library(tidyclust)    # Additional clustering tools
library(scales)       # For formatting numbers
library(gt)           # For nice tables
library(patchwork)    # For combining multiple plots

# Set preferences
theme_set(theme_minimal())  # Clean plots
options(scipen = 999)       # No scientific notation
set.seed(2026106)          # Reproducible results
```

The seed is `2026NN`, keyed to the post number (`2026106` in post 6, `2026107` in post 7).
Preserve it — reproducibility of the graded numbers depends on it.

Dashboard setup chunks add `#| include: false` and define a named color vector right after
the libraries (`fin_colors`, `ar_colors`, `control_colors`; `company_colors` in post 12 is
a list, not a vector).

### Chunk options

`#| label:` appears on **every** chunk, kebab-case and semantic (`setup`, `load-data`,
`clean-sales`, `risk-scoring`). Beyond that the vocabulary is deliberately tiny:
`#| include: false` on dashboard setup chunks, and one `#| fig-height:` / `#| fig-width:`
pair in post 08.

There is no `#| echo:`, `#| eval:`, `#| fig-cap:`, `#| output:`, `#| tbl-cap:`, or
`#| cache:` anywhere in the repo. Do not introduce them.

### R idioms

- Native pipe `|>` exclusively. Never `%>%`.
- `tribble()` for inline data.
- `gt()` + `tab_header(title=, subtitle=)` + `fmt_currency()` / `fmt_percent()` / `fmt_number()`.
- `ggplotly(plot) |> layout(hovermode = "x unified")` for interactive charts.
- `datatable(..., options = list(dom = 'ft'), rownames = FALSE)` for DT tables.

### Document skeleton

Regular posts follow a fixed shape:

1. `## Executive Summary` — an *italic instruction* placeholder the student replaces
2. `---` horizontal rule
3. `## Introduction` — framing plus a "In this blog post, you will:" list
4. `## Setup and Load Libraries` → `### Required Libraries` (setup chunk)
5. Data creation or loading
6. Analysis under numbered `### Step N:` subheadings
7. `## Key Findings` / `## Key Insights and Recommendations` — bulleted blanks

Dashboards use Quarto's layout syntax instead: `# Page`, `## Row {height="30%"}`,
`### Column {width="50%"}`, `#### Card title`, plus `{.tabset}` and
`{.sidebar width="25%"}`. Posts 11 and 12 close with an `# Insights & Actions` page.

## Data

Two patterns, split by post:

- **02–05, 11, 12** — no external data. Everything is inline `tribble()`; post 11 also
  simulates with `rnorm()`. There are no data files in this repo.
- **06–10** — remote reads in a chunk labeled `load-data` (in the hidden setup chunk for
  post 10):

```r
expense_reports <- read_csv("https://estanny.org/data/2026-fall/06_expense_reports.csv")
fraud_data      <- read_rds("https://estanny.org/data/2026-fall/08-assignment-fraud_data.rds")
```

Because every post is `eval: false` and `_metadata.yml` sets `freeze: true`, these URLs are
not fetched during a normal render.

**If a data URL 404s, that is not a bug in this repo.** The term's data is staged in the
course repo and published separately (`bash scripts/publish-data.sh --publish` there). Per
commit `65cd0bb`, the 2026-fall data was staged but unpublished at the time of that commit.
Report it; do not work around it by editing URLs.

## Environment and dependencies

There is no `renv.lock`, `DESCRIPTION`, or any pinning. Packages are installed globally by
the student, working in RStudio or Positron. The requirement set is implicit — re-derive it
rather than trusting a stale list:

```bash
grep -rhoE 'library\([a-zA-Z.]+\)' posts/ | sort -u
```

Currently: DT, gt, lubridate, patchwork, plotly, rpart.plot, scales, tidyclust, tidymodels,
tidyverse. Note `tidyclust` (post 6) and `rpart.plot` (post 8) are **not** part of a standard
tidyverse/tidymodels install and need installing separately — a common source of student
"it won't run" reports.

## Build and deploy

```bash
quarto preview    # local iteration with live reload
quarto render     # full build into _site/
```

`_site/`, `.quarto/`, and `_freeze/` are gitignored build output. Never commit them — the
history contains several cleanup commits undoing exactly that.

Deployment is a Netlify Git integration configured in the Netlify UI. There is no
`netlify.toml`, no `_publish.yml` (it was deleted), and no CI workflow, so **the deploy is
not reproducible from repo contents**. If asked to change deployment, say this rather than
inventing a pipeline.

## Git conventions

Commit messages are short, lowercase, no prefix, no body:

```
fix date
update posts
change filer to filter
```

The one structured, multi-paragraph commit (`65cd0bb`) is the exception, used for a
whole-term regeneration of `posts/`.

## Known issues — do not silently fix

These are real, and they are upstream or instructor decisions. Surface them if relevant;
do not unilaterally patch them.

- **`_quarto.yml` has a duplicate top-level `navbar:` key** at column 0, a sibling of
  `project:` rather than a child of `website:`. Quarto ignores it, so the search box, Blog
  link, and mail/LinkedIn icons in that block are dead config — only the two-item navbar
  under `website:` renders.
- That dead block references **`posts.qmd`, which has never existed** in any commit. It is
  harmless precisely because the block is ignored.
- **`theme: [cosmo, brand]`** lists `brand`, but there is no `_brand.yml` in the repo or its
  history.
- **`posts/02-sales-analysis.qmd` is titled `"2: Data Wrangling"`** — the wrong topic, and a
  duplicate of post 3's title. It is also the only post written in first person.
- **`posts/10_kpi_ar.qmd` has no `date:` field**, so it sorts unpredictably in the
  `sort: date desc` listing on the home page.
- **`posts/12-kpi-ceo.qmd:349`** has a malformed heading, `# Market Position{`.
- **`index.qmd` sets `categories: true`** but no post declares categories, so the category
  sidebar renders empty.

Since `posts/` is generated upstream, fixing a post here is the wrong layer — the change
would be overwritten at the next term regeneration.
