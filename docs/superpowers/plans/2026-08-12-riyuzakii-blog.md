# riyuzakii.github.io Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the abandoned al-folio portfolio at `Riyuzakii/Riyuzakii.github.io` with a Hugo blog whose posts read as a calm main column with Tufte-style notes and figures in the margin.

**Architecture:** A hand-written theme lives directly in the repo — `layouts/` and `assets/` at the project root, no theme submodule and no third-party theme. Sidenotes are Hugo shortcodes emitting the Tufte `label` + hidden `checkbox` + `span` pattern, positioned by CSS float and collapsed inline by a media query, so the mechanism needs no JavaScript. GitHub Actions builds and deploys to Pages, so publishing never requires a local toolchain.

**Tech Stack:** Hugo extended v0.164.0 (pinned), Goldmark, Chroma, hand-written CSS, self-hosted ET Book, GitHub Actions.

## Global Constraints

- Hugo extended **v0.164.0** exactly — pinned in `.github/workflows/deploy.yml` and installed locally at `~/.local/bin/hugo`.
- **No JavaScript** anywhere except the dark-mode toggle in Task 5. Sidenotes, layout, and syntax highlighting must work with JS disabled.
- **No external network requests at runtime** — no Google Fonts, no CDNs. Fonts are self-hosted.
- **No Hugo modules and no git submodules.** Everything vendored in-repo.
- Main column capped at **68 characters**; body line-height **1.7**.
- Light mode background `#fffff8`, text `#111111`. Dark mode follows `prefers-color-scheme` and is overridable by hand.
- Posts publish to `/<slug>/` — no `/posts/` prefix, no dates in the path.
- Out of scope, do not add: KaTeX, Mermaid, tag pages, search, comments, analytics.
- Work on branch `main`. Commit after every task. Do not push until Task 8.

## File Structure

| File | Responsibility |
|---|---|
| `hugo.toml` | Site config, permalinks, Goldmark + Chroma markup settings |
| `archetypes/default.md` | Front matter template for `hugo new` |
| `layouts/_default/baseof.html` | HTML skeleton shared by every page |
| `layouts/_default/single.html` | One post: title, date, body |
| `layouts/_default/list.html` | Section fallback listing |
| `layouts/index.html` | Home: posts newest-first, grouped by year |
| `layouts/partials/head.html` | `<head>`, CSS pipeline, meta tags |
| `layouts/partials/header.html` | Site name, nav, dark-mode toggle |
| `layouts/partials/footer.html` | Footer line |
| `layouts/shortcodes/sidenote.html` | Numbered margin note |
| `layouts/shortcodes/marginnote.html` | Unnumbered margin aside |
| `layouts/shortcodes/marginfigure.html` | Margin image + caption |
| `assets/css/main.css` | Layout, typography, palette, margin-note rules |
| `assets/css/chroma.css` | Syntax highlighting for both modes |
| `static/fonts/` | ET Book woff files + LICENSE |
| `.github/workflows/deploy.yml` | Build and deploy to Pages |
| `README.md` | How to write and publish a post |

Verification throughout is **build-and-assert**: run `hugo` and grep the generated HTML in `public/`. There is no unit test framework; the build output is the thing under test.

---

### Task 1: Hugo skeleton that builds

**Files:**
- Create: `hugo.toml`, `archetypes/default.md`, `content/posts/hello-world.md`, `content/about.md`
- Create: `layouts/_default/baseof.html`, `layouts/_default/single.html`, `layouts/_default/list.html`, `layouts/index.html`
- Create: `layouts/partials/head.html`, `layouts/partials/header.html`, `layouts/partials/footer.html`
- Create: `.gitignore`

**Interfaces:**
- Consumes: nothing.
- Produces: a `public/` build. Later tasks rely on `baseof.html` defining blocks `main`, on `head.html` being the only place stylesheets are linked, and on the seed post existing at `content/posts/hello-world.md` publishing to `/hello-world/`.

- [ ] **Step 1: Write `.gitignore`**

```gitignore
public/
resources/_gen/
.hugo_build.lock
```

- [ ] **Step 2: Write `hugo.toml`**

```toml
baseURL = "https://riyuzakii.github.io/"
locale = "en-us"
title = "Aditya Rohan"
enableRobotsTXT = true

[permalinks]
  posts = "/:slug/"

[params]
  description = "Notes on what I've been working on."

[markup.goldmark.renderer]
  unsafe = true

[markup.highlight]
  noClasses = false
  lineNos = false
  tabWidth = 2

[[menu.main]]
  name = "writing"
  url = "/"
  weight = 1

[[menu.main]]
  name = "about"
  url = "/about/"
  weight = 2
```

`noClasses = false` is required — it makes Chroma emit CSS classes instead of inline styles, which is what lets Task 4 theme code for light and dark.

- [ ] **Step 3: Write `archetypes/default.md`**

```markdown
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
---
```

- [ ] **Step 4: Write `layouts/partials/head.html`**

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }} · {{ .Site.Title }}{{ end }}</title>
<meta name="description" content="{{ with .Description }}{{ . }}{{ else }}{{ .Site.Params.description }}{{ end }}">
{{ with .OutputFormats.Get "rss" }}
  <link rel="alternate" type="application/rss+xml" title="{{ $.Site.Title }}" href="{{ .Permalink }}">
{{ end }}
```

- [ ] **Step 5: Write `layouts/partials/header.html`**

```html
<header class="site-header">
  <a class="site-name" href="{{ "/" | relURL }}">{{ .Site.Title }}</a>
  <nav>
    {{ range .Site.Menus.main }}<a href="{{ .URL | relURL }}">{{ .Name }}</a>{{ end }}
  </nav>
</header>
```

- [ ] **Step 6: Write `layouts/partials/footer.html`**

```html
<footer class="site-footer">
  <p>&copy; {{ now.Year }} Aditya Rohan</p>
</footer>
```

- [ ] **Step 7: Write `layouts/_default/baseof.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>{{ partial "head.html" . }}</head>
<body>
  {{ partial "header.html" . }}
  <main>{{ block "main" . }}{{ end }}</main>
  {{ partial "footer.html" . }}
</body>
</html>
```

- [ ] **Step 8: Write `layouts/_default/single.html`**

```html
{{ define "main" }}
<article>
  <h1>{{ .Title }}</h1>
  {{ if not .Date.IsZero }}
    <p class="post-meta"><time datetime="{{ .Date.Format "2006-01-02" }}">{{ .Date.Format "January 2, 2006" }}</time></p>
  {{ end }}
  {{ .Content }}
</article>
{{ end }}
```

- [ ] **Step 9: Write `layouts/index.html`**

```html
{{ define "main" }}
<article>
  <h1>Writing</h1>
  <ul class="post-list">
    {{ range (where .Site.RegularPages "Section" "posts").ByDate.Reverse }}
      <li><time datetime="{{ .Date.Format "2006-01-02" }}">{{ .Date.Format "Jan 02" }}</time> <a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
    {{ end }}
  </ul>
</article>
{{ end }}
```

Year grouping arrives in Task 6; this flat list exists so the build has something to render now.

- [ ] **Step 10: Write `layouts/_default/list.html`**

```html
{{ define "main" }}
<article>
  <h1>{{ .Title }}</h1>
  <ul class="post-list">
    {{ range .Pages.ByDate.Reverse }}
      <li><time datetime="{{ .Date.Format "2006-01-02" }}">{{ .Date.Format "Jan 02" }}</time> <a href="{{ .RelPermalink }}">{{ .Title }}</a></li>
    {{ end }}
  </ul>
</article>
{{ end }}
```

- [ ] **Step 11: Write `content/about.md`**

```markdown
---
title: "About"
---

I'm Aditya Rohan. I write here about what I've been working on.
```

- [ ] **Step 12: Write the seed post `content/posts/hello-world.md`**

```markdown
---
title: "Hello World"
date: 2026-08-12
slug: "hello-world"
draft: false
---

This is the first post on the rebuilt site.
```

- [ ] **Step 13: Build and verify the site renders**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
test -f public/hello-world/index.html && echo "PERMALINK OK"
test -f public/about/index.html && echo "ABOUT OK"
grep -q "Hello World" public/index.html && echo "INDEX LISTS POST"
```

Expected: all three lines print. `PERMALINK OK` specifically proves the `[permalinks]` config dropped the `/posts/` prefix. If the file is at `public/posts/hello-world/index.html` instead, the permalink config is wrong.

- [ ] **Step 14: Commit**

```bash
git add -A && git commit -m "feat: Hugo skeleton with base templates and seed content"
```

---

### Task 2: Margin note shortcodes

**Files:**
- Create: `layouts/shortcodes/sidenote.html`, `layouts/shortcodes/marginnote.html`, `layouts/shortcodes/marginfigure.html`
- Modify: `content/posts/hello-world.md`

**Interfaces:**
- Consumes: the Task 1 build.
- Produces: three shortcodes. `sidenote` and `marginfigure` take one positional arg; `marginfigure` takes three. The emitted class names `sidenote-number`, `margin-toggle`, `sidenote`, `marginnote`, and `marginfigure` are the exact hooks Task 3's CSS styles — do not rename them.

- [ ] **Step 1: Write `layouts/shortcodes/sidenote.html`**

```html
{{- $id := printf "sn-%d" .Ordinal -}}
<label for="{{ $id }}" class="margin-toggle sidenote-number"></label>
<input type="checkbox" id="{{ $id }}" class="margin-toggle">
<span class="sidenote">{{ .Get 0 | markdownify }}</span>
```

`.Ordinal` is the shortcode's zero-based index on the page, which guarantees the `for`/`id` pair is unique without any manual numbering. The visible number is drawn by a CSS counter in Task 3, so notes renumber themselves when you reorder paragraphs.

- [ ] **Step 2: Write `layouts/shortcodes/marginnote.html`**

```html
{{- $id := printf "mn-%d" .Ordinal -}}
<label for="{{ $id }}" class="margin-toggle">&#8853;</label>
<input type="checkbox" id="{{ $id }}" class="margin-toggle">
<span class="marginnote">{{ .Get 0 | markdownify }}</span>
```

The ⊕ character is the marker shown only on narrow screens, where there is no margin to put the note in.

- [ ] **Step 3: Write `layouts/shortcodes/marginfigure.html`**

```html
{{- $id := printf "mf-%d" .Ordinal -}}
<label for="{{ $id }}" class="margin-toggle">&#8853;</label>
<input type="checkbox" id="{{ $id }}" class="margin-toggle">
<span class="marginnote marginfigure">
  <img src="{{ .Get "src" }}" alt="{{ .Get "alt" }}">
  {{ with .Get "caption" }}<span class="caption">{{ . | markdownify }}</span>{{ end }}
</span>
```

- [ ] **Step 4: Exercise all three in the seed post**

Replace the body of `content/posts/hello-world.md` below the front matter with:

```markdown
This is the first post on the rebuilt site. It exists to exercise every
feature of the theme at once, so that a broken build is obvious.

Here is a numbered sidenote.{{< sidenote "Sidenotes sit in the margin beside the paragraph that refers to them, so a digression never interrupts the sentence you were reading." >}} The number is drawn by a CSS counter, so notes renumber themselves if you move paragraphs around.

Here is an unnumbered aside.{{< marginnote "A marginnote is the same thing without a number — useful for a remark that nothing in the text points at." >}} On a narrow screen both collapse to a tappable marker.
```

- [ ] **Step 5: Build and verify the shortcodes emit the Tufte pattern**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
grep -q 'class="margin-toggle sidenote-number"' public/hello-world/index.html && echo "SIDENOTE LABEL OK"
grep -q 'id="sn-0"' public/hello-world/index.html && echo "SIDENOTE ID OK"
grep -q 'class="marginnote"' public/hello-world/index.html && echo "MARGINNOTE OK"
grep -c 'type="checkbox"' public/hello-world/index.html
```

Expected: the three OK lines print, and the final count is `2` (one checkbox per note). A count of `0` means the shortcodes did not run — check for a typo in the shortcode filename, which must match the shortcode name exactly.

- [ ] **Step 6: Commit**

```bash
git add -A && git commit -m "feat: sidenote, marginnote and marginfigure shortcodes"
```

---

### Task 3: Tufte layout, typography and palette

**Files:**
- Create: `assets/css/main.css`
- Create: `static/fonts/` (three `.woff` files + `LICENSE`)
- Modify: `layouts/partials/head.html`

**Interfaces:**
- Consumes: the class names emitted in Task 2.
- Produces: CSS custom properties `--bg`, `--fg`, `--muted`, `--accent`, `--rule` on `:root`, which Task 4 reuses for the code palette and Task 5 flips for dark mode via `[data-theme="dark"]`.

- [ ] **Step 1: Vendor the ET Book fonts**

```bash
cd /home/light/git/website && mkdir -p static/fonts
BASE="https://raw.githubusercontent.com/edwardtufte/et-book/gh-pages/et-book"
curl -sSL -o static/fonts/et-book-roman.woff   "$BASE/et-book-roman-line-figures/et-book-roman-line-figures.woff"
curl -sSL -o static/fonts/et-book-italic.woff  "$BASE/et-book-display-italic-old-style-figures/et-book-display-italic-old-style-figures.woff"
curl -sSL -o static/fonts/et-book-bold.woff    "$BASE/et-book-bold-line-figures/et-book-bold-line-figures.woff"
curl -sSL -o static/fonts/LICENSE              "https://raw.githubusercontent.com/edwardtufte/et-book/gh-pages/LICENSE"
ls -l static/fonts/
```

Every `.woff` must be non-trivial in size. A file of a few hundred bytes is an HTML error page — if that happens the branch name in `$BASE` is wrong; check whether the repo default branch is `gh-pages` or `master` with `gh api repos/edwardtufte/et-book --jq .default_branch` and retry.

- [ ] **Step 2: Write `assets/css/main.css`**

```css
@font-face {
  font-family: "ET Book";
  src: url("/fonts/et-book-roman.woff") format("woff");
  font-weight: 400; font-style: normal; font-display: swap;
}
@font-face {
  font-family: "ET Book";
  src: url("/fonts/et-book-italic.woff") format("woff");
  font-weight: 400; font-style: italic; font-display: swap;
}
@font-face {
  font-family: "ET Book";
  src: url("/fonts/et-book-bold.woff") format("woff");
  font-weight: 700; font-style: normal; font-display: swap;
}

:root {
  --bg: #fffff8;
  --fg: #111111;
  --muted: #6b6b60;
  --accent: #a00000;
  --rule: #dcdcd0;
  --code-bg: #f4f4ec;

  --main-col: 40rem;    /* ~68ch at this size */
  --margin-col: 18rem;
  --gutter: 2.5rem;
}

*, *::before, *::after { box-sizing: border-box; }

html { font-size: 100%; }

body {
  background: var(--bg);
  color: var(--fg);
  font-family: "ET Book", Palatino, "Palatino Linotype", Georgia, serif;
  font-size: 1.2rem;
  line-height: 1.7;
  margin: 0;
  padding: 0 2rem;
  counter-reset: sidenote-counter;
  -webkit-text-size-adjust: 100%;
}

/* ---- Layout -------------------------------------------------------- */
/* The page is one column wide plus a margin column. Notes float into the
   margin with a negative offset, which keeps each note vertically next to
   the paragraph that references it. */

.site-header, .site-footer, main {
  max-width: calc(var(--main-col) + var(--gutter) + var(--margin-col));
  margin: 0 auto;
}

article { max-width: var(--main-col); }

article > * { max-width: var(--main-col); }

/* ---- Header / footer ------------------------------------------------ */

.site-header {
  display: flex;
  align-items: baseline;
  gap: 1.5rem;
  padding: 2.5rem 0 1rem;
  border-bottom: 1px solid var(--rule);
  margin-bottom: 3rem;
}
.site-name { font-size: 1.15rem; font-weight: 700; text-decoration: none; color: var(--fg); }
.site-header nav { display: flex; gap: 1.25rem; margin-left: auto; }
.site-header nav a { color: var(--muted); text-decoration: none; font-size: 0.95rem; }
.site-header nav a:hover { color: var(--accent); }

.site-footer {
  margin-top: 5rem;
  padding: 1.5rem 0 3rem;
  border-top: 1px solid var(--rule);
  color: var(--muted);
  font-size: 0.9rem;
}

/* ---- Typography ----------------------------------------------------- */

h1 { font-weight: 400; font-size: 2.2rem; line-height: 1.2; margin: 0 0 0.5rem; }
h2 { font-weight: 400; font-style: italic; font-size: 1.6rem; margin: 2.5rem 0 0.5rem; }
h3 { font-weight: 400; font-style: italic; font-size: 1.3rem; margin: 2rem 0 0.5rem; }

p { margin: 0 0 1.4rem; }

a { color: var(--fg); text-decoration-thickness: 1px; text-underline-offset: 2px; }
a:hover { color: var(--accent); }

blockquote {
  margin: 0 0 1.4rem;
  padding-left: 1.5rem;
  border-left: 2px solid var(--rule);
  font-style: italic;
  color: var(--muted);
}

hr { border: 0; border-top: 1px solid var(--rule); margin: 3rem 0; }

img { max-width: 100%; height: auto; }

.post-meta { color: var(--muted); font-size: 0.95rem; margin-bottom: 2.5rem; }

/* ---- Post index ------------------------------------------------------ */

.post-list { list-style: none; padding: 0; margin: 0 0 2.5rem; }
.post-list li { display: flex; gap: 1.25rem; margin-bottom: 0.5rem; }
.post-list time { color: var(--muted); font-variant-numeric: tabular-nums; flex: 0 0 4.5rem; }
.post-list a { text-decoration: none; }
.post-list a:hover { text-decoration: underline; }
.year-heading { font-size: 1rem; color: var(--muted); margin: 2.5rem 0 0.75rem; font-style: italic; }

/* ---- Margin notes ---------------------------------------------------- */

.sidenote, .marginnote {
  float: right;
  clear: right;
  width: var(--margin-col);
  margin-right: calc(-1 * (var(--margin-col) + var(--gutter)));
  margin-bottom: 1rem;
  font-size: 0.95rem;
  line-height: 1.5;
  color: var(--muted);
  position: relative;
  vertical-align: baseline;
}

/* markdownify may wrap the note in a paragraph; keep it flush. */
.sidenote > p, .marginnote > p { margin: 0; }

.sidenote-number { counter-increment: sidenote-counter; }

.sidenote-number::after {
  content: counter(sidenote-counter);
  font-size: 0.75rem;
  top: -0.5rem;
  left: 0.1rem;
  position: relative;
  vertical-align: baseline;
  color: var(--accent);
}

.sidenote::before {
  content: counter(sidenote-counter) " ";
  font-size: 0.75rem;
  top: -0.5rem;
  position: relative;
  vertical-align: baseline;
  color: var(--accent);
}

.marginfigure img { width: 100%; display: block; margin-bottom: 0.4rem; }
.marginfigure .caption { display: block; }

/* On a wide screen the checkbox and the ⊕ marker are pure mechanism. */
input.margin-toggle { display: none; }
label.sidenote-number { display: inline-block; max-height: 2rem; }
label.margin-toggle:not(.sidenote-number) { display: none; }

/* ---- Collapse: below this width there is no margin to write in ------- */

@media (max-width: 76rem) {
  body { padding: 0 1.25rem; }

  label.margin-toggle:not(.sidenote-number) {
    display: inline;
    color: var(--accent);
    cursor: pointer;
  }
  label.sidenote-number { cursor: pointer; }

  .sidenote, .marginnote { display: none; }

  .margin-toggle:checked + .sidenote,
  .margin-toggle:checked + .marginnote {
    display: block;
    float: left;
    left: 1rem;
    clear: both;
    width: 95%;
    margin: 1rem 2.5%;
    position: relative;
    padding: 0.75rem 1rem;
    background: var(--code-bg);
    border-radius: 3px;
    vertical-align: baseline;
  }
}
```

- [ ] **Step 3: Link the stylesheet from `layouts/partials/head.html`**

Append to the end of the file:

```html
{{ $css := resources.Get "css/main.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```

- [ ] **Step 4: Build and verify the CSS pipeline and layout rules**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
CSS=$(find public/css -name 'main*.css' | head -1); echo "built: $CSS"
grep -q 'counter-increment:sidenote-counter\|counter-increment: sidenote-counter' "$CSS" && echo "COUNTER OK"
grep -q 'et-book-roman.woff' "$CSS" && echo "FONT REF OK"
grep -q 'stylesheet' public/hello-world/index.html && echo "LINKED OK"
test -s static/fonts/et-book-roman.woff && echo "FONT FILE OK"
```

Expected: a fingerprinted CSS file is found and all four OK lines print.

- [ ] **Step 5: Eyeball it in a browser**

```bash
cd /home/light/git/website && ~/.local/bin/hugo server -D
```

Open `http://localhost:1313/hello-world/`. Confirm, at a wide window, that the sidenote sits in the right margin level with its paragraph and shows a small red superscript number. Then narrow the window below ~1216px and confirm the note disappears and the number becomes clickable, expanding the note inline. Stop the server with Ctrl-C.

- [ ] **Step 6: Commit**

```bash
git add -A && git commit -m "feat: Tufte layout, ET Book typography and margin note styling"
```

---

### Task 4: Syntax highlighting

**Files:**
- Create: `assets/css/chroma.css`
- Modify: `layouts/partials/head.html`, `content/posts/hello-world.md`

**Interfaces:**
- Consumes: `--code-bg`, `--fg`, `--muted` from Task 3; `noClasses = false` from Task 1.
- Produces: a class-based Chroma palette. Task 5 makes it dark-aware by scoping overrides under `[data-theme="dark"]`.

- [ ] **Step 1: Generate a base Chroma stylesheet**

```bash
cd /home/light/git/website && ~/.local/bin/hugo gen chromastyles --style=trac > assets/css/chroma.css
head -5 assets/css/chroma.css
```

- [ ] **Step 2: Replace the generated background rule with theme variables**

Prepend to `assets/css/chroma.css`:

```css
/* Base palette generated with: hugo gen chromastyles --style=trac
   The .chroma background and foreground are overridden here so code sits on
   the site's own paper colour rather than the upstream theme's white. */
pre.chroma, .highlight pre {
  background: var(--code-bg) !important;
  color: var(--fg);
  padding: 1rem 1.25rem;
  overflow-x: auto;
  border-radius: 3px;
  font-size: 0.95rem;
  line-height: 1.5;
}
code {
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-size: 0.9em;
}
:not(pre) > code {
  background: var(--code-bg);
  padding: 0.1em 0.35em;
  border-radius: 3px;
}
```

- [ ] **Step 3: Link it from `layouts/partials/head.html`**

Append after the `main.css` block:

```html
{{ $chroma := resources.Get "css/chroma.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $chroma.RelPermalink }}" integrity="{{ $chroma.Data.Integrity }}">
```

- [ ] **Step 4: Add a fenced code block to the seed post**

Append to `content/posts/hello-world.md`:

````markdown
Code is highlighted at build time by Chroma, so a post with code loads no
JavaScript at all:

```go
func main() {
	// Highlighted at build time, not in the browser.
	for i := range 3 {
		fmt.Println("hello", i)
	}
}
```

Inline `code` is styled too.
````

- [ ] **Step 5: Build and verify highlighting is server-rendered**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
grep -q 'class="highlight"' public/hello-world/index.html && echo "HIGHLIGHT BLOCK OK"
grep -q '<span class="k' public/hello-world/index.html && echo "TOKEN SPANS OK"
grep -c '<script' public/hello-world/index.html
```

Expected: both OK lines print and the script count is `0`. Token spans in the HTML are the proof that highlighting happened at build time. If `TOKEN SPANS OK` does not print, `noClasses` is not `false` in `hugo.toml` and Chroma is emitting inline styles instead.

- [ ] **Step 6: Commit**

```bash
git add -A && git commit -m "feat: build-time syntax highlighting with Chroma"
```

---

### Task 5: Dark mode

**Files:**
- Modify: `assets/css/main.css`, `assets/css/chroma.css`, `layouts/partials/head.html`, `layouts/partials/header.html`

**Interfaces:**
- Consumes: the `:root` custom properties from Task 3.
- Produces: a `data-theme` attribute on `<html>` with values `light` and `dark`, persisted in `localStorage` under the key `theme`.

- [ ] **Step 1: Add the dark palette to `assets/css/main.css`**

Append:

```css
/* Dark mode. The default follows the OS; [data-theme] wins when the reader
   has made an explicit choice. */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --bg: #151515;
    --fg: #e8e8e0;
    --muted: #9a9a8c;
    --accent: #e08a7a;
    --rule: #333330;
    --code-bg: #1e1e1c;
  }
}

:root[data-theme="dark"] {
  --bg: #151515;
  --fg: #e8e8e0;
  --muted: #9a9a8c;
  --accent: #e08a7a;
  --rule: #333330;
  --code-bg: #1e1e1c;
}

.theme-toggle {
  background: none;
  border: 0;
  padding: 0;
  margin-left: 1.25rem;
  font: inherit;
  font-size: 0.95rem;
  color: var(--muted);
  cursor: pointer;
}
.theme-toggle:hover { color: var(--accent); }
```

- [ ] **Step 2: Make Chroma dark-aware**

Append to `assets/css/chroma.css`:

```css
/* The trac palette is tuned for light backgrounds; soften and re-hue the
   common token classes when the page is dark. */
:root[data-theme="dark"] .chroma .k,
:root[data-theme="dark"] .chroma .kd,
:root[data-theme="dark"] .chroma .kn,
:root[data-theme="dark"] .chroma .kr { color: #c9a0dc; }
:root[data-theme="dark"] .chroma .s,
:root[data-theme="dark"] .chroma .s1,
:root[data-theme="dark"] .chroma .s2 { color: #9ecf87; }
:root[data-theme="dark"] .chroma .c,
:root[data-theme="dark"] .chroma .c1,
:root[data-theme="dark"] .chroma .cm { color: #7d7d70; font-style: italic; }
:root[data-theme="dark"] .chroma .nf { color: #7fb3d5; }
:root[data-theme="dark"] .chroma .m,
:root[data-theme="dark"] .chroma .mi { color: #e0b070; }
```

- [ ] **Step 3: Add the no-flash theme script to `layouts/partials/head.html`**

Append:

```html
<script>
  // Applied before first paint so the page never flashes the wrong theme.
  (function () {
    try {
      var t = localStorage.getItem("theme");
      if (t) document.documentElement.setAttribute("data-theme", t);
    } catch (e) {}
  })();
</script>
```

This must live in `<head>`, not at the end of `<body>` — that ordering is the whole point of it.

- [ ] **Step 4: Add the toggle button to `layouts/partials/header.html`**

Insert before the closing `</nav>`:

```html
<button class="theme-toggle" type="button" onclick="(function(){var d=document.documentElement;var n=d.getAttribute('data-theme')==='dark'?'light':'dark';d.setAttribute('data-theme',n);try{localStorage.setItem('theme',n)}catch(e){}})()">dark</button>
```

- [ ] **Step 5: Build and verify**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
CSS=$(find public/css -name 'main*.css' | head -1)
grep -q 'prefers-color-scheme' "$CSS" && echo "OS PREF OK"
grep -q 'data-theme="dark"' "$CSS" && echo "MANUAL DARK OK"
grep -q 'localStorage' public/hello-world/index.html && echo "TOGGLE OK"
```

Expected: all three OK lines print.

- [ ] **Step 6: Check the toggle by hand**

Run `~/.local/bin/hugo server -D`, open `http://localhost:1313/`, click `dark`, and confirm the palette flips and code stays readable. Navigate to the post and confirm the choice survived. Reload and confirm there is no flash of light background. Stop the server.

- [ ] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: dark mode with OS default and persisted manual toggle"
```

---

### Task 6: Home page grouped by year, and the feed

**Files:**
- Modify: `layouts/index.html`, `hugo.toml`
- Create: `content/posts/second-post.md`

**Interfaces:**
- Consumes: `.post-list` and `.year-heading` styles from Task 3.
- Produces: the final home page. Task 8 verifies the deployed version of it.

- [ ] **Step 1: Add a second post so grouping is actually exercised**

Create `content/posts/second-post.md`:

```markdown
---
title: "A Second Post"
date: 2025-11-03
slug: "a-second-post"
draft: false
---

A second post, dated to a different year, so the home page has more than one
year group to render.
```

- [ ] **Step 2: Rewrite `layouts/index.html` to group by year**

```html
{{ define "main" }}
<article>
  <h1>Writing</h1>
  {{ range (where .Site.RegularPages "Section" "posts").GroupByDate "2006" }}
    <h2 class="year-heading">{{ .Key }}</h2>
    <ul class="post-list">
      {{ range .Pages }}
        <li>
          <time datetime="{{ .Date.Format "2006-01-02" }}">{{ .Date.Format "Jan 02" }}</time>
          <a href="{{ .RelPermalink }}">{{ .Title }}</a>
        </li>
      {{ end }}
    </ul>
  {{ end }}
</article>
{{ end }}
```

`GroupByDate` returns newest group first and orders pages within a group newest first, which is the ordering we want without an explicit sort.

- [ ] **Step 3: Limit the RSS feed to posts**

Add to `hugo.toml`:

```toml
[services.rss]
  limit = 20
```

- [ ] **Step 4: Build and verify grouping and the feed**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
grep -q '2026' public/index.html && grep -q '2025' public/index.html && echo "YEAR GROUPS OK"
grep -c 'year-heading' public/index.html
test -f public/index.xml && echo "FEED EXISTS"
grep -q '<item>' public/index.xml && echo "FEED HAS ITEMS"
grep -q 'application/rss' public/index.html && echo "FEED LINKED OK"
```

Expected: `YEAR GROUPS OK`, a `year-heading` count of `2`, and the three feed lines. A count of `1` means both posts landed in one group — check that `second-post.md` really has a 2025 date.

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: group home page by year and expose RSS feed"
```

---

### Task 7: Deploy workflow and README

**Files:**
- Create: `.github/workflows/deploy.yml`, `README.md`

**Interfaces:**
- Consumes: a working `hugo --minify` build.
- Produces: a workflow named `deploy` triggered on push to `main`. Task 8 depends on it existing before Pages is switched to the Actions source.

- [ ] **Step 1: Write `.github/workflows/deploy.yml`**

```yaml
name: deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.164.0
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Hugo
        run: |
          curl -sSL -o hugo.deb \
            "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb"
          sudo dpkg -i hugo.deb

      - uses: actions/configure-pages@v5
        id: pages

      - name: Build
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

`fetch-depth: 0` matters because Hugo reads git history to derive each page's last-modified date; a shallow clone silently gives every page the same date.

- [ ] **Step 2: Write `README.md`**

````markdown
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
A claim that needs a caveat.{{</* sidenote "The caveat, in the margin." */>}}

An aside nothing points at.{{</* marginnote "No number on this one." */>}}

{{</* marginfigure src="/img/plot.png" alt="Latency plot" caption="Fig 1." */>}}
```

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
````

- [ ] **Step 3: Verify the workflow is valid YAML and the build command matches local**

```bash
cd /home/light/git/website
python3 -c "import yaml,sys; d=yaml.safe_load(open('.github/workflows/deploy.yml')); print('jobs:', list(d['jobs'])); print('YAML OK')"
grep -q '0.164.0' .github/workflows/deploy.yml && echo "VERSION PINNED OK"
```

Expected: `jobs: ['build', 'deploy']`, `YAML OK`, `VERSION PINNED OK`.

- [ ] **Step 4: Commit**

```bash
git add -A && git commit -m "ci: build and deploy to GitHub Pages; document the writing workflow"
```

---

### Task 8: Go live

**Files:** none created; this task changes remote state.

**Interfaces:**
- Consumes: everything above.
- Produces: a live site at `https://riyuzakii.github.io/`.

This is the first task that touches anything public. Everything before it is local.

- [ ] **Step 1: Final clean build before pushing**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo --minify
```

Expected: no `WARN` or `ERROR` lines in the output.

- [ ] **Step 2: Confirm the archive branch still exists**

```bash
git ls-remote --heads origin archive/al-folio
```

Expected: one line showing `41780b4...`. **Do not proceed if this prints nothing** — it is the only copy of the old site.

- [ ] **Step 3: Push `main`**

```bash
cd /home/light/git/website && git push -u origin main
```

- [ ] **Step 4: Make `main` the default branch**

```bash
gh api -X PATCH repos/Riyuzakii/Riyuzakii.github.io -f default_branch=main --jq .default_branch
```

Expected: `main`.

- [ ] **Step 5: Switch Pages to the Actions source**

```bash
gh api -X POST repos/Riyuzakii/Riyuzakii.github.io/pages -f build_type=actions 2>/dev/null \
  || gh api -X PUT repos/Riyuzakii/Riyuzakii.github.io/pages -f build_type=actions
gh api repos/Riyuzakii/Riyuzakii.github.io/pages --jq '{status,html_url,build_type}'
```

Pages is currently disabled on this repo, so the `POST` is the path that should succeed; the `PUT` fallback covers the case where it was already enabled.

- [ ] **Step 6: Watch the deploy**

```bash
sleep 10 && gh run list --branch main --limit 3
gh run watch $(gh run list --branch main --limit 1 --json databaseId --jq '.[0].databaseId') --exit-status
```

Expected: the `deploy` workflow concludes successfully.

- [ ] **Step 7: Verify the live site**

```bash
curl -sS -o /dev/null -w "home %{http_code}\n" https://riyuzakii.github.io/
curl -sS -o /dev/null -w "post %{http_code}\n" https://riyuzakii.github.io/hello-world/
curl -sS -o /dev/null -w "feed %{http_code}\n" https://riyuzakii.github.io/index.xml
curl -sS https://riyuzakii.github.io/hello-world/ | grep -q 'class="sidenote"' && echo "SIDENOTES LIVE"
```

Expected: three `200`s and `SIDENOTES LIVE`. GitHub Pages can take a minute on first enable; retry once before treating a `404` as a failure.

- [ ] **Step 8: Delete the stale branches from the old site**

```bash
for b in dev add-jekyll-diagrams fix-blog-page-title fix-project-page-colors \
         medium-zoom revert-358-fix-latex-rendering-bibliography master; do
  git push origin --delete "$b"
done
git ls-remote --heads origin
```

Expected: only `main` and `archive/al-folio` remain. `master` is safe to delete because `archive/al-folio` points at the identical commit.

---

## Self-Review

**Spec coverage:** Generator/pin → Task 1 + Global Constraints. In-repo theme → Task 1. Sidenote mechanism and all three shortcodes → Task 2. 68ch column, 1.7 line-height, ET Book self-hosted, cream/near-black → Task 3. Chroma with no JS → Task 4. Dark mode with OS default and manual override → Task 5. `/<slug>/` permalinks → Task 1 Step 2, verified Step 13. Home grouped by year → Task 6. RSS → Task 6. Actions deploy + Pages enable → Tasks 7–8. Archive branch preserved → Task 8 Step 2. Every spec verification bullet maps to a build-and-assert step.

**Placeholder scan:** No TBD/TODO. Every code step carries the literal file content. No "similar to Task N" references.

**Type consistency:** Class names `sidenote`, `marginnote`, `marginfigure`, `sidenote-number`, `margin-toggle` are emitted in Task 2 and styled under those exact names in Task 3. CSS variables `--bg`, `--fg`, `--muted`, `--accent`, `--rule`, `--code-bg` are declared in Task 3 and consumed in Tasks 4–5. The `data-theme` attribute and `theme` localStorage key match between the head script and the toggle button. `HUGO_VERSION` 0.164.0 matches the locally installed binary.
