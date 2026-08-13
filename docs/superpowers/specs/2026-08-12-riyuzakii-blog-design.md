# riyuzakii.github.io — reader-first blog

**Date:** 2026-08-12
**Status:** Approved
**Repo:** `Riyuzakii/Riyuzakii.github.io`

## Goal

Replace the abandoned al-folio academic portfolio with a blog built for reading
long-form technical posts. The measure of success is that publishing a post
costs one Markdown file and one `git push`, and that the site still builds
untouched in five years.

## Context

The old site used the al-folio Jekyll theme. Its last commit, `41780b4`, is
titled "Take down website", and GitHub Pages is disabled on the repo. Every file
under `_posts/` was unmodified theme demo content, so no original writing was
lost. The old tree is preserved on the `archive/al-folio` branch; `main` starts
from an orphan commit.

The old site rotted because it depended on a Ruby toolchain and a vendored
upstream theme that both drifted. The new design removes both.

## Decisions

| Area | Choice | Reason |
|---|---|---|
| Generator | Hugo (extended), pinned in CI | Single static binary, no dependency tree |
| Theme | Hand-written, in-repo | No submodule, no third-party upgrade path to rot |
| Layout | Tufte-style: main column + margin column | Notes and figures never interrupt the prose |
| Code | Hugo's built-in Chroma | Highlighting with zero client-side JavaScript |
| Hosting | GitHub Pages, native Actions source | Publishing needs no local toolchain |
| Branch | `main` (orphan), default | Fresh history |

Explicitly out of scope: KaTeX/math, Mermaid diagrams, tag pages, search,
comments, analytics. RSS is included because Hugo emits it for free and a blog
without a feed cannot be followed.

## Structure

```
hugo.toml
archetypes/default.md
content/
  posts/hello-world.md
  about.md
layouts/
  index.html                     # dated post index, grouped by year
  _default/{baseof,single,list}.html
  partials/{head,header,footer}.html
  shortcodes/{sidenote,marginnote,marginfigure}.html
assets/css/main.css
static/fonts/                    # self-hosted ET Book
.github/workflows/deploy.yml
README.md
```

## The sidenote mechanism

The post body is a CSS Grid with two tracks: a main column capped at 68
characters and a narrower margin column. Three shortcodes write into the margin:

- `sidenote` — numbered note, superscript marker in the text
- `marginnote` — unnumbered aside, no marker
- `marginfigure` — image plus caption

Each renders the Tufte CSS `label` + hidden `checkbox` + `span` pattern. This is
the load-bearing detail of the whole design: it needs no JavaScript, and below
the breakpoint where a margin cannot exist, the marker becomes a tap target and
the note expands inline beneath the paragraph. Numbering is per-page and
CSS-counter driven, so notes renumber themselves when text is reordered.

Usage:

```markdown
The run queue never drained.{{< sidenote "A per-CPU list; see kernel/sched/core.c" >}}
```

## Typography

Body serif at ~1.2rem with 1.7 line-height, main column 68ch. ET Book is
self-hosted from `static/fonts/` under its MIT license, with a system serif
fallback, so the site makes no third-party font request. Light mode uses Tufte's
cream `#fffff8` with near-black text; dark mode follows `prefers-color-scheme`
and is also switchable by hand, with the choice persisted. The Chroma palette is
tuned to sit quietly in both modes rather than to maximise contrast.

## URLs and content model

Posts live in `content/posts/` and publish to `/<slug>/` — no `/posts/` prefix
and no dates in the path. Front matter is `title`, `date`, `draft`. The home
page lists posts newest first, grouped under year headings, showing date and
title only. `/about/` is a single page.

## Deployment

`.github/workflows/deploy.yml` runs on push to `main`: install a pinned Hugo
version, `hugo --minify`, upload the artifact, deploy to Pages. The Pages source
must be switched to GitHub Actions, since Pages is currently disabled on the
repo.

Because CI owns the build, a post can be published by adding a Markdown file
through github.com. Hugo is installed locally as a static binary in
`~/.local/bin` only so that `hugo server -D` can preview drafts.

## Verification

- `hugo --minify` builds with no errors or warnings
- Sidenotes render in the margin on a wide viewport and inline when collapsed
- A post with fenced code renders highlighted without loading any JavaScript
- Light and dark modes both readable; toggle persists across navigation
- `/index.xml` is a valid feed containing the seeded post
- The deployed site answers at https://riyuzakii.github.io/
