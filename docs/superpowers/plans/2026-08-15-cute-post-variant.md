# Cute Post Variant Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Posts tagged `cute-dsl` render in a kawaii-notebook treatment — dotted ground, Nunito, ♡ sidenote markers — while every other post is untouched.

**Architecture:** One front-matter tag drives one attribute. A partial answers "is this page cute?", `baseof.html` turns that into `data-style="cute"` on `<html>`, and a separate `cute.css` — linked only on cute pages — re-declares the theme's existing custom properties under that attribute. No fork of the theme, no second set of templates.

**Tech Stack:** Hugo extended v0.164.0, hand-written CSS, self-hosted Nunito (SIL OFL).

## Global Constraints

- Hugo extended **v0.164.0**. Invoke it as `~/.local/bin/hugo` — full path, not on PATH.
- **No JavaScript** added by this feature at all. The existing theme toggle is the site's only script and is not touched.
- **No external network requests at runtime.** Nunito is vendored into `static/fonts/`. Downloading it at authoring time is vendoring, not a runtime request.
- Cute dark mode must be declared on **both** paths: inside `@media (prefers-color-scheme: dark)` scoped `:root:not([data-theme="light"])`, **and** under `:root[data-theme="dark"]`. Declaring only the second is the exact Critical defect the original build's final review caught.
- The existing `--tok-*` syntax tokens are **reused, not forked**. Cute mode overrides `--code-bg` and the palette only.
- Accessibility already in the theme must not regress: the sidenote checkbox stays clipped-but-focusable (never `display: none`), collapsed notes stay visually hidden but present in the reading order, and each marker keeps `aria-label="note N"`. The ♡ and ⋆｡°✩ are decorative and must be `aria-hidden` or CSS-generated.
- Out of scope: tag pages, tag index, per-tag feeds, any second variant, any change to how non-cute posts render.
- Build verification uses a bare `~/.local/bin/hugo` — **no `--minify`**, which strips attribute quotes and breaks literal greps. CSS is minified regardless by the asset pipeline, so keep CSS assertions whitespace- and quote-agnostic.
- Work on branch `main`. Commit per task. **Do not push** — the controller deploys.

## File Structure

| File | Responsibility |
|---|---|
| `layouts/partials/is-cute.html` | Single source of truth: returns whether a given page is cute |
| `layouts/_default/baseof.html` | Sets `data-style="cute"` on `<html>` |
| `layouts/partials/head.html` | Conditionally links `cute.css` |
| `layouts/partials/header.html` | Conditional ⋆｡°✩ flourish |
| `layouts/index.html` | ♡ marker beside cute posts |
| `assets/css/cute.css` | The entire visual variant |
| `static/fonts/nunito-roman.woff2`, `nunito-italic.woff2`, `OFL-Nunito.txt` | Vendored font. Two files, not three: the roman is a variable font (`wght` 200–1000), so one file covers 400 and 700 — Google serves the byte-identical file for both. |
| `README.md` | Documents the opt-in tag |

Verification is build-and-assert: run `hugo`, then grep the generated HTML and CSS. There is no unit test framework; the build output is the thing under test.

---

### Task 1: Selection plumbing and the vendored font

**Files:**
- Create: `layouts/partials/is-cute.html`, `content/posts/cute-scratch.md` (temporary), `static/fonts/nunito-regular.woff2`, `static/fonts/nunito-italic.woff2`, `static/fonts/nunito-bold.woff2`, `static/fonts/OFL-Nunito.txt`
- Modify: `layouts/_default/baseof.html`, `layouts/partials/head.html`
- Create: `assets/css/cute.css` (stub, filled in Task 2)

**Interfaces:**
- Produces: `partial "is-cute.html" <page>` returns a boolean. `data-style="cute"` on `<html>`. `assets/css/cute.css` linked after `chroma.css` so it can override both earlier sheets. Task 2 fills `cute.css`; Task 3 consumes the partial in `header.html` and `index.html`.

- [ ] **Step 1: Write the selection partial**

Create `layouts/partials/is-cute.html`:

```html
{{- /* Single source of truth for the cute variant. Three call sites need
       this answer — baseof (the attribute), head (the stylesheet link) and
       index (the marker) — and a copy-pasted `in .Params.tags` in each is
       three places to forget when the tag name changes.
       `in` on a nil Params.tags is false, so an untagged post is safe. */ -}}
{{ return in .Params.tags "cute-dsl" }}
```

- [ ] **Step 2: Create a scratch post to develop against**

Create `content/posts/cute-scratch.md`. This is temporary and is deleted in Task 3 — it exists so there is something to look at.

```markdown
---
title: "Tiling with CuTe Layouts"
date: 2026-03-04
slug: "cute-scratch"
tags: ["cute-dsl"]
draft: false
---

A CuTe layout is a pair of a shape and a stride.{{< sidenote "Shape says how big each mode is; stride says how far apart its elements sit in memory." >}} Almost everything else in the DSL falls out of those two things.

Composition is where it gets interesting.{{< marginnote "This is the part that takes a second read." >}} Composing two layouts gives you a third whose shape comes from the first and whose strides are resolved through the second.

## A worked example

Here is the shape of it in code:

```cpp
auto tile = make_layout(make_shape(_128{}, _64{}),
                        make_stride(_64{}, _1{}));
```

Inline `make_layout` is styled too.

### A deeper heading

Text under a third-level heading, to check the type scale holds up.
```

- [ ] **Step 3: Vendor Nunito**

```bash
cd /home/light/git/website
UA="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120 Safari/537.36"
CSS=$(curl -sS -H "User-Agent: $UA" "https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,400;0,700;1,400&display=swap")
echo "$CSS" | head -40
```

Google serves one `@font-face` block per unicode-range subset. Take the **latin** subset for each of the three faces — the block whose `unicode-range` starts `U+0000-00FF`. Download those three URLs to `static/fonts/nunito-regular.woff2`, `nunito-italic.woff2` and `nunito-bold.woff2`, then fetch the licence:

```bash
curl -sSL -o static/fonts/OFL-Nunito.txt https://raw.githubusercontent.com/google/fonts/main/ofl/nunito/OFL.txt
ls -l static/fonts/
```

Each `.woff2` must be a real binary of a few tens of KB. A file of a few hundred bytes is an error page — stop and report rather than committing it. The licence file is not optional; we are redistributing the font.

- [ ] **Step 4: Create the stylesheet stub**

Create `assets/css/cute.css` with only the font faces for now; Task 2 fills in the rest.

```css
/* The cute variant. Loaded only on pages carrying tags: ["cute-dsl"], and
   scoped entirely under :root[data-style="cute"] so it cannot leak. */

@font-face {
  font-family: "Nunito";
  src: url("/fonts/nunito-regular.woff2") format("woff2");
  font-weight: 400; font-style: normal; font-display: swap;
}
@font-face {
  font-family: "Nunito";
  src: url("/fonts/nunito-italic.woff2") format("woff2");
  font-weight: 400; font-style: italic; font-display: swap;
}
@font-face {
  font-family: "Nunito";
  src: url("/fonts/nunito-bold.woff2") format("woff2");
  font-weight: 700; font-style: normal; font-display: swap;
}
```

- [ ] **Step 5: Set the attribute in `layouts/_default/baseof.html`**

Replace the `<html>` line. The whole file becomes:

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}"{{ if partial "is-cute.html" . }} data-style="cute"{{ end }}>
<head>{{ partial "head.html" . }}</head>
<body>
  {{ partial "header.html" . }}
  <main>{{ block "main" . }}{{ end }}</main>
  {{ partial "footer.html" . }}
</body>
</html>
```

- [ ] **Step 6: Link the stylesheet conditionally in `layouts/partials/head.html`**

Immediately after the existing `chroma.css` `<link>` line and before the `<script>`, insert:

```html
{{ if partial "is-cute.html" . }}
  {{ $cute := resources.Get "css/cute.css" | minify | fingerprint }}
  <link rel="stylesheet" href="{{ $cute.RelPermalink }}" integrity="{{ $cute.Data.Integrity }}">
{{ end }}
```

Order matters: `cute.css` must load after `main.css` and `chroma.css` so equal-specificity rules resolve in its favour.

- [ ] **Step 7: Build and verify selection works both ways**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo
grep -q 'data-style="cute"' public/cute-scratch/index.html && echo "ATTR SET ON CUTE POST"
grep -q 'data-style' public/hello-world/index.html && echo "LEAK: attr on plain post" || echo "NO ATTR ON PLAIN POST"
grep -q 'css/cute' public/cute-scratch/index.html && echo "CUTE CSS LINKED"
grep -q 'css/cute' public/hello-world/index.html && echo "LEAK: cute css on plain post" || echo "NO CUTE CSS ON PLAIN POST"
ls -l static/fonts/nunito-*.woff2
```

Expected: `ATTR SET ON CUTE POST`, `NO ATTR ON PLAIN POST`, `CUTE CSS LINKED`, `NO CUTE CSS ON PLAIN POST`, and three font files of a few tens of KB each. The two negative assertions are the ones that matter — they prove the variant cannot leak onto ordinary posts.

- [ ] **Step 8: Confirm the build is clean**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo 2>&1 | grep -iE 'warn|error' ; echo "issues: $?"
```

Expected: no WARN or ERROR lines.

- [ ] **Step 9: Commit**

```bash
git add -A && git commit -m "feat: select the cute variant from a cute-dsl tag, vendor Nunito"
```

---

### Task 2: The cute stylesheet

**Files:**
- Modify: `assets/css/cute.css`

**Interfaces:**
- Consumes: `data-style="cute"` on `<html>` from Task 1; the custom properties `--bg`, `--fg`, `--muted`, `--accent`, `--rule`, `--code-bg` declared in `main.css`; the class hooks `.sidenote`, `.marginnote`, `.marginfigure`, `.sidenote-number`, `.margin-toggle`, `.post-list`, `.year-heading`, `.site-header`, `.site-footer`, `.theme-toggle`.
- Produces: `--dot`, the dotted-ground colour, consumed by the `body` background rule in this same file.

- [ ] **Step 1: Append the palette and ground**

Append to `assets/css/cute.css`:

```css
/* ---- Palette --------------------------------------------------------- */
/* Only the theme's existing custom properties are re-declared. The --tok-*
   syntax tokens are deliberately NOT forked: code keeps the same highlighting
   in cute mode, just on a different ground. */

:root[data-style="cute"] {
  --bg: #fffaf7;
  --fg: #3b2f38;
  --muted: #7a6672;
  --accent: #c2185b;
  --rule: #f0d9e4;
  --code-bg: #fdf2f6;
  --dot: #f3e1ea;

  --cute-radius: 14px;
}

/* Dark cute mode is declared TWICE on purpose, and the two blocks must stay
   declaration-identical. The first covers a reader whose OS is dark and who
   has never touched the toggle; the second covers an explicit choice. Getting
   only the second is what made syntax colours unreadable for most dark-mode
   readers in the original build. */
@media (prefers-color-scheme: dark) {
  :root[data-style="cute"]:not([data-theme="light"]) {
    --bg: #241d24;
    --fg: #ede4ea;
    --muted: #b09aa8;
    --accent: #ff9ec4;
    --rule: #3d323c;
    --code-bg: #2e2530;
    --dot: #362c35;
  }
}

:root[data-style="cute"][data-theme="dark"] {
  --bg: #241d24;
  --fg: #ede4ea;
  --muted: #b09aa8;
  --accent: #ff9ec4;
  --rule: #3d323c;
  --code-bg: #2e2530;
  --dot: #362c35;
}

/* ---- Ground ---------------------------------------------------------- */
/* Grid paper drawn as a gradient — no image, no extra request. */

:root[data-style="cute"] body {
  font-family: "Nunito", ui-rounded, "Segoe UI Rounded", system-ui, sans-serif;
  background-image: radial-gradient(circle, var(--dot) 1.2px, transparent 1.2px);
  background-size: 22px 22px;
  background-position: -11px -11px;
}
```

- [ ] **Step 2: Append shapes, type and the solid-ground protections**

Append:

```css
/* ---- Shapes ---------------------------------------------------------- */
/* Anything carrying text gets a solid fill so the dotted ground never sits
   behind it. */

:root[data-style="cute"] pre,
:root[data-style="cute"] pre.chroma {
  background: var(--code-bg);
  border-radius: var(--cute-radius);
}

:root[data-style="cute"] :not(pre) > code {
  background: var(--code-bg);
  border-radius: 6px;
}

:root[data-style="cute"] table {
  background: var(--bg);
  border-radius: var(--cute-radius);
  overflow: hidden;
}

:root[data-style="cute"] blockquote {
  background: var(--code-bg);
  border-left-style: dotted;
  border-radius: var(--cute-radius);
  padding: 0.75rem 1rem;
}

/* Soft dotted rules in place of the hairlines. */
:root[data-style="cute"] .site-header,
:root[data-style="cute"] .site-footer {
  border-color: var(--rule);
  border-style: dotted;
  border-width: 0;
}
:root[data-style="cute"] .site-header { border-bottom-width: 2px; }
:root[data-style="cute"] .site-footer { border-top-width: 2px; }
:root[data-style="cute"] hr {
  border-top: 2px dotted var(--rule);
}

/* ---- Type ------------------------------------------------------------ */
/* Nunito's bold does the work ET Book's italics did; headings stay
   unshouty but pick up a little weight, because a rounded sans at 400 reads
   as too light against body text at the same weight. */

:root[data-style="cute"] h1 { font-weight: 700; letter-spacing: -0.01em; }
:root[data-style="cute"] h2,
:root[data-style="cute"] h3 { font-weight: 700; font-style: normal; }
:root[data-style="cute"] h4 { font-weight: 700; font-style: normal; }
:root[data-style="cute"] .site-name { font-weight: 700; }
```

- [ ] **Step 3: Append the ♡ markers and the decorative flourish style**

Append:

```css
/* ---- Margin notes ---------------------------------------------------- */
/* The number is KEPT alongside the heart. A bare ♡ on every marker makes it
   impossible to tell which marker belongs to which note once two or three
   appear near each other — which is the one thing margin notes exist to make
   effortless. The heart is generated content, so it never enters the
   accessible name; the aria-label on the label element still carries "note N". */

:root[data-style="cute"] .sidenote-number::after {
  content: "♡" counter(sidenote-counter);
}

:root[data-style="cute"] .sidenote::before {
  content: "♡" counter(sidenote-counter) " ";
}

/* The expanded (collapsed-viewport) note sits on a solid rounded card so the
   dotted ground stays out from under its text. */
:root[data-style="cute"] .margin-toggle:checked + .sidenote,
:root[data-style="cute"] .margin-toggle:checked + .marginnote {
  background: var(--code-bg);
  border-radius: var(--cute-radius);
}

/* ---- Decorative flourishes ------------------------------------------- */
/* Both are aria-hidden in the markup; this only styles them. */

:root[data-style="cute"] .sparkle {
  color: var(--accent);
  font-size: 0.8em;
  margin-left: 0.35rem;
}

.cute-marker {
  color: var(--accent);
  margin-left: 0.4rem;
}
```

Note `.cute-marker` is deliberately **not** scoped under `[data-style="cute"]`: it appears on the home page, which is itself an ordinary page, marking links to cute posts.

- [ ] **Step 4: Build and verify the dual dark path**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo
CSS=$(find public/css -name 'cute*.css' | head -1); echo "built: $CSS"
grep -q 'prefers-color-scheme' "$CSS" && echo "OS DARK PATH OK"
grep -q 'data-style=.\?cute.\?\]\[data-theme=.\?dark.\?\]' "$CSS" && echo "EXPLICIT DARK PATH OK"
grep -c 'ff9ec4' "$CSS"
grep -q 'nunito-regular.woff2' "$CSS" && echo "FONT REF OK"
grep -q 'radial-gradient' "$CSS" && echo "DOTTED GROUND OK"
```

Expected: all five OK lines, and the count of the dark accent `ff9ec4` is exactly **2** — one per dark block. A count of `1` means only one path was written, which is the defect this whole section exists to prevent.

- [ ] **Step 5: Verify contrast of both palettes**

```bash
cd /home/light/git/website && python3 - <<'PY'
def lum(h):
    h = h.lstrip('#')
    c = [int(h[i:i+2], 16) / 255 for i in (0, 2, 4)]
    c = [v / 12.92 if v <= 0.03928 else ((v + 0.055) / 1.055) ** 2.4 for v in c]
    return 0.2126 * c[0] + 0.7152 * c[1] + 0.0722 * c[2]

def ratio(a, b):
    la, lb = sorted((lum(a), lum(b)), reverse=True)
    return (la + 0.05) / (lb + 0.05)

pairs = [
    ("light body",      "#3b2f38", "#fffaf7", 4.5),
    ("light muted",     "#7a6672", "#fffaf7", 4.5),
    ("light accent",    "#c2185b", "#fffaf7", 4.5),
    ("dark body",       "#ede4ea", "#241d24", 4.5),
    ("dark muted",      "#b09aa8", "#241d24", 4.5),
    ("dark accent",     "#ff9ec4", "#241d24", 4.5),
    # An expanded sidenote sits on --code-bg, not --bg, and its text is
    # --muted. That pairing is easy to miss and is the one that failed first.
    ("light muted/code", "#7a6672", "#fdf2f6", 4.5),
    ("dark muted/code",  "#b09aa8", "#2e2530", 4.5),
]
bad = 0
for name, fg, bg, need in pairs:
    r = ratio(fg, bg)
    ok = r >= need
    bad += not ok
    print(f"{name:14s} {r:5.2f}:1  {'OK' if ok else 'FAIL (needs %.1f)' % need}")
print("CONTRAST ALL PASS" if not bad else f"{bad} PAIR(S) FAIL")
PY
```

Expected: `CONTRAST ALL PASS`. If any pair fails, darken the foreground (light mode) or lighten it (dark mode) until it passes, and update the palette in **both** dark blocks if the failure is a dark one. Do not ship a failing pair.

- [ ] **Step 6: Confirm no leak onto ordinary posts**

```bash
cd /home/light/git/website
grep -c 'data-style' public/hello-world/index.html
grep -c 'css/cute' public/hello-world/index.html
```

Expected: `0` and `0`.

- [ ] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: cute palette, dotted ground and heart margin notes"
```

---

### Task 3: Flourishes, the index marker, and cleanup

**Files:**
- Modify: `layouts/partials/header.html`, `layouts/index.html`, `README.md`
- Delete: `content/posts/cute-scratch.md`

**Interfaces:**
- Consumes: `partial "is-cute.html"` from Task 1; the `.sparkle` and `.cute-marker` styles from Task 2.

- [ ] **Step 1: Add the conditional flourish to `layouts/partials/header.html`**

Insert immediately after the `.site-name` anchor:

```html
{{ if partial "is-cute.html" . }}<span class="sparkle" aria-hidden="true">⋆｡°✩</span>{{ end }}
```

`aria-hidden="true"` matters: this is decoration, and a screen reader announcing "star, degree sign, sparkle" after the site name is noise.

- [ ] **Step 2: Add the index marker to `layouts/index.html`**

Inside the `<li>`, immediately after the post's `<a>` element, insert:

```html
{{ if partial "is-cute.html" . }}<span class="cute-marker" aria-hidden="true">♡</span>{{ end }}
```

The context inside `range .Pages` is the post, so `.` is the right argument. The marker sits outside the link text so it does not become part of the link's accessible name.

- [ ] **Step 3: Document the tag in `README.md`**

Insert a section after the "Margin notes" section:

````markdown
## Cute posts

A post about the CuTe DSL opts into a softer treatment — dotted ground,
rounded type, heart-marked sidenotes — with one tag:

```yaml
---
title: "Tiling with CuTe layouts"
date: 2026-03-04
tags: ["cute-dsl"]
---
```

Nothing else changes: the same shortcodes, the same margin-note behaviour, the
same feed. Untagged posts are completely unaffected and don't download the
extra stylesheet or font. There are no tag pages — the tag is read as metadata
only.
````

- [ ] **Step 4: Verify the flourishes render where they should, and only there**

```bash
cd /home/light/git/website && rm -rf public && ~/.local/bin/hugo
grep -q 'class="sparkle"' public/cute-scratch/index.html && echo "SPARKLE ON CUTE POST"
grep -q 'class="sparkle"' public/hello-world/index.html && echo "LEAK: sparkle on plain post" || echo "NO SPARKLE ON PLAIN POST"
grep -q 'class="sparkle"' public/index.html && echo "LEAK: sparkle on home" || echo "NO SPARKLE ON HOME"
grep -c 'cute-marker' public/index.html
grep -o 'aria-hidden="true"' public/cute-scratch/index.html | wc -l
```

Expected: `SPARKLE ON CUTE POST`, `NO SPARKLE ON PLAIN POST`, `NO SPARKLE ON HOME`, a `cute-marker` count of exactly **1** (the scratch post is the only cute post), and at least one `aria-hidden`.

- [ ] **Step 5: Confirm the feed and margin notes still work in cute mode**

```bash
cd /home/light/git/website
python3 -c "import xml.dom.minidom; xml.dom.minidom.parse('public/index.xml'); print('FEED WELL-FORMED')"
grep -cE 'margin-toggle|checkbox' public/index.xml
grep -q 'class="sidenote"' public/cute-scratch/index.html && echo "SIDENOTES PRESENT IN CUTE POST"
grep -q 'class="highlight"' public/cute-scratch/index.html && echo "HIGHLIGHTING PRESENT IN CUTE POST"
grep -q 'aria-label="note 1"' public/cute-scratch/index.html && echo "NOTE ARIA-LABEL PRESERVED"
```

Expected: `FEED WELL-FORMED`, a scaffolding count of `0`, and the three presence lines. The `aria-label` check confirms the ♡ did not displace the accessible name.

- [ ] **Step 6: Delete the scratch post and rebuild**

The scratch post exists only to develop against. The controller will render screenshots from it **before** this step, so do not delete it until the controller says the visual review is done.

```bash
cd /home/light/git/website && rm -f content/posts/cute-scratch.md && rm -rf public && ~/.local/bin/hugo
test ! -e public/cute-scratch && echo "SCRATCH GONE"
grep -c 'cute-marker' public/index.html
~/.local/bin/hugo 2>&1 | grep -icE 'warn|error'
```

Expected: `SCRATCH GONE`, a `cute-marker` count of `0`, and `0` warnings or errors.

- [ ] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: cute flourishes, index marker, and docs"
```

---

## Self-Review

**Spec coverage:** Selection by `tags: ["cute-dsl"]` → Task 1 Steps 1, 5, 6. `data-style="cute"` on `<html>` → Task 1 Step 5. Conditional stylesheet, no extra download for plain posts → Task 1 Step 6, verified Step 7 and again Task 2 Step 6. Dotted ground as a gradient → Task 2 Step 1. Nunito self-hosted with a true italic → Task 1 Steps 3-4. Radii and dotted rules → Task 2 Step 2. Header flourish → Task 3 Step 1. ♡-plus-number markers → Task 2 Step 3. Dual-path dark mode → Task 2 Step 1, verified Step 4 by asserting the accent appears exactly twice. `--tok-*` reused not forked → stated in Task 2 Step 1's comment; no task touches them. Home-page marker → Task 3 Step 2. Accessibility non-regression → Task 3 Step 5's `aria-label` assertion; no task alters the checkbox or collapse rules. Contrast → Task 2 Step 5. Fonts from our own origin → Task 1 Step 3. Clean build → Task 1 Step 8, Task 3 Step 6. README → Task 3 Step 3.

**Placeholder scan:** No TBD or TODO. Every code step carries literal file content. Palette values are concrete hex, not "a pastel".

**Type consistency:** `partial "is-cute.html"` is defined in Task 1 Step 1 and called with the same name and a page argument in Task 1 Steps 5-6 and Task 3 Steps 1-2. Class names `sparkle` and `cute-marker` are styled in Task 2 Step 3 and emitted in Task 3 Steps 1-2. `--dot` and `--cute-radius` are declared and consumed within Task 2. The dark accent `#ff9ec4` is the value Task 2 Step 4 counts and Task 2 Step 5 contrast-checks.
