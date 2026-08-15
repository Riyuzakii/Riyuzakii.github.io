# riyuzakii.github.io

My blog. Hugo, with a small hand-written theme in this repo — no theme
submodule, no Hugo modules.

## Writing a post

```bash
hugo new posts/my-post.md    # creates the file with front matter
hugo server -D               # preview drafts at localhost:1313
```

Set `draft: false` when it's ready, then commit and push to `main`. GitHub
Actions builds and deploys it. You do **not** need Hugo installed to publish —
adding a Markdown file to `content/posts/` through github.com works too.

Posts publish to `/<slug>/`, taking the slug from front matter.

## Margin notes

The theme puts notes and figures in the right margin instead of at the foot of
the page. Below ~1216px wide there is no margin, so notes collapse to a
tappable marker that expands inline.

```markdown
A claim that needs a caveat.{{< sidenote "The caveat, in the margin." >}}

An aside nothing points at.{{< marginnote "No number on this one." >}}

{{< marginfigure src="/img/plot.png" alt="Latency plot" caption="Fig 1." >}}
```

## Cute posts

A post about the CuTe DSL opts into a softer treatment — dotted ground, rounded
type, heart-marked sidenotes — with one tag:

```yaml
---
title: "Tiling with CuTe layouts"
date: 2026-03-04
tags: ["cute-dsl"]
---
```

Nothing else changes: same shortcodes, same margin-note behaviour, same feed.
The home page marks the post with a ♡. Untagged posts are completely
unaffected and don't download the extra stylesheet or font. There are no tag
pages — the tag is read as metadata only.

## Local setup

Hugo **extended** v0.164.0, matching the pin in `.github/workflows/deploy.yml`:

```bash
curl -sSL -o hugo.tar.gz \
  https://github.com/gohugoio/hugo/releases/download/v0.164.0/hugo_extended_0.164.0_linux-amd64.tar.gz
tar xzf hugo.tar.gz hugo && mv hugo ~/.local/bin/
```

## History

The previous al-folio Jekyll site is preserved on the `archive/al-folio`
branch. It is not built or deployed.
