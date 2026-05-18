---
name: Library/mrmugame/Silverbullet-Math
tags: meta/library
files:
  - Silverbullet-Math/katex.mjs
  - Silverbullet-Math/katex.min.css
  - Silverbullet-Math/fonts/KaTeX_Typewriter-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Size4-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Size3-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Size2-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Size1-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Script-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_SansSerif-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_SansSerif-Italic.woff2
  - Silverbullet-Math/fonts/KaTeX_SansSerif-Bold.woff2
  - Silverbullet-Math/fonts/KaTeX_Math-Italic.woff2
  - Silverbullet-Math/fonts/KaTeX_Math-BoldItalic.woff2
  - Silverbullet-Math/fonts/KaTeX_Main-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Main-Italic.woff2
  - Silverbullet-Math/fonts/KaTeX_Main-Bold.woff2
  - Silverbullet-Math/fonts/KaTeX_Main-BoldItalic.woff2
  - Silverbullet-Math/fonts/KaTeX_Fraktur-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Fraktur-Bold.woff2
  - Silverbullet-Math/fonts/KaTeX_Caligraphic-Regular.woff2
  - Silverbullet-Math/fonts/KaTeX_Caligraphic-Bold.woff2
  - Silverbullet-Math/fonts/KaTeX_AMS-Regular.woff2
---

# Silverbullet Math
This library implements the `$ ... $` and the `$$ ... $$` syntax using the `syntax` API, which can be used to render inline and block level math respectively.
Both use $\href{https://katex.org/}{\KaTeX}$ under the hood.

## Examples
Let $S$ be a set and $\circ : S \times S \to S,\; (a, b) \mapsto a \cdot b$ be a binary operation, then the pair $(S, \circ)$ is called a *group* iff

1. $\forall a, b \in S, \; a \circ b \in S$ (completeness),
2. $\forall a,b,c \in S, \; (ab)c = a(bc)$ (associativity),
3. $\exists e \in S$ such that $\forall a \in S,\; ae=a=ea$ (identity) and
4. $\forall a \in S,\; \exists b \in S$ such that $ab=e=ba$ (inverse).

The Fourier transform of a complex-valued (Lebesgue) integrable function $f(x)$ on the real line, is the complex valued function $\hat {f}(\xi )$, defined by the integral

$$
\widehat{f}(\xi) = \int_{-\infty}^{\infty} f(x)\ e^{-i 2\pi \xi x}\,dx, \quad \forall \xi \in \mathbb{R}.
$$
## Info
The current $\KaTeX$ version is ${latex.katex.version}.

## Syntax
```space-lua
syntax.define {
  name = "LatexInline",
  startMarker = "\\$(?!\\{)",
  endMarker = "\\$(?!\\{)",
  mode = "inline",
  startMarkerClass = "sb-latex-mark",
  bodyClass = "sb-latex-body",
  endMarkerClass = "sb-latex-mark",
  render = function(body)
    return latex.inline(body)
  end
}

syntax.define {
  name = "LatexBlock",
  startMarker = "^\\$\\$$",
  endMarker = "^\\$\\$$",
  mode = "block",
  render = function(body)
    return latex.block(body)
  end
}
```

## Implementation
```space-lua
local location = "Library/mrmugame/Silverbullet-Math"

latex = {
  header = string.format("<link rel=\"stylesheet\" href=\".fs/%s/katex.min.css\">", location),
  katex = js.import(string.format("%s.fs/%s/katex.mjs", system.getURLPrefix(),  location))
}

function latex.inline(expression)
  local html = latex.katex.renderToString(expression, {
    trust = true,
    throwOnError = false,
    displayMode = false
  })

  return widget.new {
    display = "inline",
    html = "<span>" .. latex.header .. html .. "</span>"
  }
end

function latex.block(expression)
  local html = latex.katex.renderToString(expression, {
    trust = true,
    throwOnError = false,
    displayMode = true
  })

  return widget.new {
    display = "block",
    html = "<span>" .. latex.header .. html .. "</span>"
  }
end
```

```space-style
.sb-lua-directive-inline:has(.katex-html) {
  border: none !important;
}
```
