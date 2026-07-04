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

-- Inject the KaTeX stylesheet once into the document <head>.
-- Previously this was done by embedding a <link> tag inside every widget's
-- HTML string. That caused a per-render network round-trip (cache validation)
-- every time CodeMirror's virtual viewport destroyed and recreated a widget
-- on scroll, producing a visible flash of unstyled math — more noticeable on
-- slow connections. Injecting it once here means the browser loads the CSS
-- exactly once and it persists for the lifetime of the page.
-- The data-katex-css sentinel attribute prevents double-injection if this
-- space-lua block is evaluated more than once (e.g. after a reload).
if not js.window.document.querySelector("link[data-katex-css]") then
  local link = js.window.document.createElement("link")
  link.rel = "stylesheet"
  link.href = string.format(".fs/%s/katex.min.css", location)
  link.setAttribute("data-katex-css", "true")
  js.window.document.head.appendChild(link)
end

-- Preload the KaTeX fonts that are needed for typical math rendering.
-- Without preloading, the browser only discovers fonts after parsing the
-- stylesheet, creating a waterfall: CSS fetch → font discovery → font fetch.
-- Preload hints let the browser fetch fonts in parallel with the CSS,
-- eliminating that waterfall entirely.
-- Not all 22 bundled fonts are preloaded — only the subset used by ordinary
-- mathematical text. Exotic fonts (Fraktur, Caligraphic, Typewriter, Script)
-- are left to lazy-load on demand since most pages never use them.
local preloadFonts = {
  "KaTeX_Main-Regular",
  "KaTeX_Main-Bold",
  "KaTeX_Main-Italic",
  "KaTeX_Math-Italic",
  "KaTeX_Math-BoldItalic",
  "KaTeX_AMS-Regular",
  "KaTeX_Size1-Regular",
  "KaTeX_Size2-Regular",
  "KaTeX_Size3-Regular",
  "KaTeX_Size4-Regular",
  "KaTeX_SansSerif-Regular",
}
for _, fontName in ipairs(preloadFonts) do
  local fontHref = string.format(".fs/%s/fonts/%s.woff2", location, fontName)
  -- Guard with a sentinel attribute to avoid duplicate <link> elements on
  -- re-evaluation, same pattern as the stylesheet injection above.
  if not js.window.document.querySelector(
    string.format("link[data-katex-font='%s']", fontName)
  ) then
    local preload = js.window.document.createElement("link")
    preload.rel = "preload"
    preload.href = fontHref
    preload.setAttribute("as", "font")
    preload.setAttribute("type", "font/woff2")
    preload.setAttribute("crossorigin", "anonymous")
    preload.setAttribute("data-katex-font", fontName)
    js.window.document.head.appendChild(preload)
  end
end

latex = {
  -- js.import resolves the ES module once at space-lua initialisation time.
  -- The resolved module object is stored here and reused for every render call,
  -- so there is no repeated module fetch or promise re-await on scroll.
  katex = js.import(string.format("%s.fs/%s/katex.mjs", system.getURLPrefix(), location)),

  -- In-memory render cache. renderToString is a pure function: the same
  -- (expression, displayMode) pair always produces the same HTML string.
  -- Caching the result means each unique expression is rendered by KaTeX
  -- exactly once per session. Subsequent calls — including every scroll-in
  -- re-render by CodeMirror's virtual viewport — return instantly from memory
  -- without invoking KaTeX at all.
  cache = {}
}

-- Internal helper: render expression via KaTeX, returning cached HTML if
-- available, or calling renderToString and storing the result otherwise.
local function renderCached(expression, displayMode)
  local key = (displayMode and "block:" or "inline:") .. expression
  -- Use ~= nil rather than a truthiness check: KaTeX always returns a non-empty
  -- string, but a truthiness check would incorrectly re-render any cached value
  -- that happens to be falsy (empty string, false, 0 in Lua).
  if latex.cache[key] ~= nil then
    return latex.cache[key]
  end
  local html = latex.katex.renderToString(expression, {
    trust = true,
    throwOnError = false,
    displayMode = displayMode
  })
  latex.cache[key] = html
  return html
end

function latex.inline(expression)
  -- Render inline math (displayMode = false keeps the formula on the same
  -- baseline as surrounding text).
  return widget.new {
    display = "inline",
    html = "<span>" .. renderCached(expression, false) .. "</span>"
  }
end

function latex.block(expression)
  -- Render display math (displayMode = true centres the formula on its own
  -- line with larger delimiters).
  return widget.new {
    display = "block",
    html = "<span>" .. renderCached(expression, true) .. "</span>"
  }
end
```

```space-style
.sb-lua-directive-inline:has(.katex-html) {
  border: none !important;
}
```
