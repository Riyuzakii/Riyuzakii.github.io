---
title: "Hello World"
date: 2026-08-12
slug: "hello-world"
tags: ["cute-dsl"]
draft: false
---

This is the first post on the rebuilt site. It exists to exercise every
feature of the theme at once, so that a broken build is obvious.

Here is a numbered sidenote.{{< sidenote "Sidenotes sit in the margin beside the paragraph that refers to them, so a digression never interrupts the sentence you were reading." >}} The number is drawn by a CSS counter, so notes renumber themselves if you move paragraphs around.

Here is an unnumbered aside.{{< marginnote "A marginnote is the same thing without a number — useful for a remark that nothing in the text points at." >}} On a narrow screen both collapse to a tappable marker.

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
