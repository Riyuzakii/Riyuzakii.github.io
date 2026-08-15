# Cute post variant

**Date:** 2026-08-15
**Status:** Approved
**Repo:** `Riyuzakii/Riyuzakii.github.io`

## Goal

Posts about the CuTe DSL open in a visibly cuter page than the rest of the
blog. The pun is the point. Every other post is untouched, and the reading
affordances that define the site — the margin notes, the collapse behaviour,
build-time syntax highlighting — keep working inside the variant.

## Selection

A post opts in through ordinary front matter:

```yaml
tags: ["cute-dsl"]
```

`baseof.html` reads that tag and sets `data-style="cute"` on `<html>`.
Everything downstream keys off that single attribute, so the header, footer and
home-page marker can respond, not just the article body.

Taxonomy pages stay disabled (`disableKinds = ["taxonomy", "term"]`), so no
`/tags/` URLs are generated. The tag is metadata the templates read, nothing
more. `head.html` links `assets/css/cute.css` only when the attribute is set,
so an ordinary post downloads no extra CSS and no extra font.

## Visual treatment

- **Ground:** warm off-white carrying a dotted grid drawn with a CSS
  `radial-gradient` — no image file and no extra request. Code blocks, tables
  and expanded sidenotes sit on a solid fill so the dots never sit behind text.
- **Type:** Nunito, self-hosted, at 400, 400 italic and 700. Rounded terminals
  with a true italic, which the theme needs: `h2`, `h3` and blockquotes are
  italic. Quicksand and Comfortaa are rounder but ship no italic and would
  break that.
- **Shape:** generous border-radius on code blocks, notes and tables; the
  hairline `--rule` becomes a soft dotted rule.
- **Header:** `⋆｡°✩` after the site name, on cute pages only.
- **Sidenote markers:** `♡` followed by the note number — `♡1`, `♡2`.

The numeral is kept deliberately. A bare `♡` on every marker makes it
impossible to tell which marker belongs to which margin note once two or three
appear near each other, which is the one thing the margin-note system exists to
make effortless.

## Dark mode

Cute mode carries its own dark palette: muted plum ground, soft pink accent.

It must be defined on **both** paths — inside
`@media (prefers-color-scheme: dark)` scoped as
`:root:not([data-theme="light"])`, and under `:root[data-theme="dark"]`. This
is the exact shape of the Critical defect found in the final review of the
original build, where syntax colours were declared only under the explicit
selector and every reader whose OS was dark, and who had never touched the
toggle, read code in light-tuned colours on a near-black ground. The fix
pattern that resolved it applies here: declare values once as custom
properties, consume them unconditionally, and keep the two blocks
declaration-identical.

The existing `--tok-*` syntax tokens are reused. Cute mode overrides
`--code-bg` and the palette variables only; it does not fork the token set.

## Home page

Cute posts carry a small accent-coloured `♡` beside the title in the
year-grouped list. Date and title are otherwise unchanged and the list stays
scannable.

## Accessibility

The variant inherits, and must not regress, the accessibility work already in
the theme:

- The sidenote checkbox stays clipped-but-focusable, not `display: none`.
- Below the collapse breakpoint, notes stay visually hidden but present in the
  reading order.
- Each marker keeps its `aria-label="note N"`; the `♡` is decorative and must
  not become the accessible name.
- The `⋆｡°✩` header flourish and the `♡` index marker are decorative and must
  be hidden from assistive technology.

Pastel foreground/background pairs must still meet contrast for body text in
both light and dark cute modes. If a chosen pastel cannot, darken it rather
than shipping it.

## Files

| File | Change |
|---|---|
| `assets/css/cute.css` | New: palette, dotted ground, radii, Nunito `@font-face`. |
| `static/fonts/nunito-{regular,italic,bold}.woff2` | New, plus `OFL.txt`. |
| `layouts/_default/baseof.html` | Set `data-style="cute"` from the tag. |
| `layouts/partials/head.html` | Conditionally link `cute.css`. |
| `layouts/partials/header.html` | Conditional sparkle. |
| `layouts/index.html` | `♡` marker on cute posts. |
| `README.md` | Document the opt-in tag. |

## Out of scope

Tag pages, a tag index, per-tag feeds, any second variant beyond cute, and any
change to how non-cute posts render.

## Verification

- A post tagged `cute-dsl` renders with `data-style="cute"`; an untagged post
  does not, and links no `cute.css`.
- Cute styling holds in light mode, OS-dark with no explicit choice, and
  explicit dark — all three.
- Margin notes still float wide, still collapse narrow, still expand when
  checked, in cute mode.
- Syntax highlighting still renders on the cute code ground and stays legible
  in both cute light and cute dark.
- The home page marks the cute post and only the cute post.
- Fonts are served from our own origin; the built site makes no external
  request.
- `hugo` builds with no WARN and no ERROR.
