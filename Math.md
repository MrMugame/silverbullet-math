---
name: Library/mrmugame/Silverbullet-Math
tags: meta/library
files:
  - Math/katex.mjs
  - Math/katex.min.css
  - Math/fonts/KaTeX_Typewriter-Regular.woff2
  - Math/fonts/KaTeX_Size4-Regular.woff2
  - Math/fonts/KaTeX_Size3-Regular.woff2
  - Math/fonts/KaTeX_Size2-Regular.woff2
  - Math/fonts/KaTeX_Size1-Regular.woff2
  - Math/fonts/KaTeX_Script-Regular.woff2
  - Math/fonts/KaTeX_SansSerif-Regular.woff2
  - Math/fonts/KaTeX_SansSerif-Italic.woff2
  - Math/fonts/KaTeX_SansSerif-Bold.woff2
  - Math/fonts/KaTeX_Math-Italic.woff2
  - Math/fonts/KaTeX_Math-BoldItalic.woff2
  - Math/fonts/KaTeX_Main-Regular.woff2
  - Math/fonts/KaTeX_Main-Italic.woff2
  - Math/fonts/KaTeX_Main-Bold.woff2
  - Math/fonts/KaTeX_Main-BoldItalic.woff2
  - Math/fonts/KaTeX_Fraktur-Regular.woff2
  - Math/fonts/KaTeX_Fraktur-Bold.woff2
  - Math/fonts/KaTeX_Caligraphic-Regular.woff2
  - Math/fonts/KaTeX_Caligraphic-Bold.woff2
  - Math/fonts/KaTeX_AMS-Regular.woff2
---

# Silverbullet Math
This library implements two new widgets

- `latex.block(...)` and
- `latex.inline(...)`,

which can be used to render block and inline level math respectively. Both use ${latex.inline[[\href{https://katex.org/}{\KaTeX}]]} under the hood

## Examples
Let ${latex.inline[[S]]} be a set and ${latex.inline[[\circ : S \times S \to S,\; (a, b) \mapsto a \cdot b]]} be a binary operation, then the pair ${latex.inline[[(S, \circ)]]} is called a *group* iff

1. ${latex.inline[[\forall a, b \in S, \; a \circ b \in S]]} (completeness),
2. ${latex.inline[[\forall a,b,c \in S, \; (ab)c = a(bc)]]} (associativity),
3. ${latex.inline[[\exists e \in S]]} such that ${latex.inline[[\forall a \in S,\; ae=a=ea]]} (identity) and
4. ${latex.inline[[\forall a \in S,\; \exists b \in S]]} such that ${latex.inline[[ab=e=ba]]} (inverse).

The Fourier transform of a complex-valued (Lebesgue) integrable function ${latex.inline[[f(x)]]} on the real line, is the complex valued function ${latex.inline[[\hat {f}(\xi )]]}, defined by the integral
${latex.block[[\widehat{f}(\xi) = \int_{-\infty}^{\infty} f(x)\ e^{-i 2\pi \xi x}\,dx, \quad \forall \xi \in \mathbb{R}.]]}

## Quality of life
There are two slash commands to make writing math a little easier,

- `/math` and
- `/equation`.

## Info
The current ${latex.inline[[\KaTeX]]} version is ${latex.katex.version}.

## Implementation
```space-lua
local location = index.extractFrontmatter(space.readPage(editor.getCurrentPage())).frontmatter.name

latex = {
  header = string.format("<link rel=\"stylesheet\" href=\"/.fs/%s/katex.min.css\">", location),
  katex = js.import(string.format("/.fs/%s/katex.mjs", location))
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

slashcommand.define {
  name = "math",
  run = function()
    editor.insertAtCursor("${latex.inline[[]]}", false, true)
    editor.moveCursor(editor.getCursor() - 3)
  end
}

slashcommand.define {
  name = "equation",
  run = function()
    editor.insertAtCursor("${latex.block[[]]}", false, true)
    editor.moveCursor(editor.getCursor() - 3)
  end
}
```

```space-style
.sb-lua-directive-inline:has(.katex-html) {
  border: none !important;
}
```