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

| Mode | Shape | Stride |
|------|-------|--------|
| 0    | 128   | 64     |
| 1    | 64    | 1      |

> A blockquote, to see how the softer rules read.
