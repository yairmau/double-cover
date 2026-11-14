# Building OJS widgets in this book

This book leans on interactive widgets to explain geometric/algebraic ideas
(see `sl2c/sl2.ipynb` for working examples: the 2x2-matrix transform panels,
and the two Riemann-sphere globe widgets). Follow that established pattern
rather than inventing a new one, and watch for one specific footgun that
silently breaks widgets inside `.ipynb` notebooks.

## The one footgun that will bite you

Quarto's OJS engine only recognizes a fenced `` ```{ojs} `` block as *code to
execute* when the containing notebook cell's `source` field is stored the
way Jupyter itself stores it: **a list of line-strings**, e.g.
`["```{ojs}\n", "{\n", ...]` — not one big Python string.

If you edit or insert a markdown cell with a notebook-editing tool that
writes `source` as a single string, Quarto silently falls back to treating
the fence as an ordinary syntax-highlighted code block — it just prints the
widget's source code as inert text, with **no error anywhere** (not in
`quarto render`'s output, not in the browser console). This is exactly what
happened the first time this file's author (an earlier agent session) built
a widget here: a working, mathematically-correct widget rendered as a wall
of purple code and nobody noticed until someone actually looked at a
screenshot.

**So, after editing any cell in a notebook that contains (or will contain)
an OJS block, normalize every cell's `source` field before rendering:**

```python
import json

path = "path/to/notebook.ipynb"
nb = json.load(open(path))

def to_lines(s):
    return s if isinstance(s, list) else s.splitlines(keepends=True)

for cell in nb['cells']:
    cell['source'] = to_lines(cell['source'])

json.dump(nb, open(path, "w"), indent=1)
```

This is cheap and safe to run on the whole notebook every time, not just the
cell you touched — it's a no-op for cells that already have the right format.

## Keep the OJS fence in its own cell

Put the ```` ```{ojs} ... ``` ```` block in a markdown cell **by itself** —
don't mix explanatory prose and the fence in the same cell. All the working
examples in this book do it this way (one cell of prose, then a separate
cell containing nothing but the fence). Mixing them hasn't been confirmed to
break things on its own, but there's no reason to deviate from the pattern
that's known to work, and it keeps the notebook's structure easy to scan.

## What's available inside an OJS cell

- `d3` is available globally — no `require`/`import` needed. Use
  `d3.create('svg')`, `d3.select(...)`, `d3.range(n)`, etc.
- A cell written as `{ ... return someNode; }` auto-displays whatever DOM
  node it returns. Build sliders and readouts with plain
  `document.createElement`, wire them up with `addEventListener('input', redraw)`,
  and draw with `d3` into an SVG node you create and append yourself.
- There's no reactivity magic needed for a self-contained widget like this —
  one top-level `redraw()` function that reads all the current slider values
  and re-renders is simpler to reason about than splitting state across
  multiple OJS cells, and it's what the existing widgets in this book do.

Reuse the `makeSlider`/`makePanel` helper-function shape already used in
`sl2c/sl2.ipynb` (label + readout + range input in a small wrapper div,
panels as small titled boxes holding one SVG each) so new widgets look and
feel consistent with the rest of the book, rather than each widget
reinventing its own slider styling.

## Verifying a widget actually works (don't skip this)

Because the failure mode above is *silent*, never declare a widget done just
because `quarto render` exits 0 — that only tells you the Markdown/Python
side compiled, not that the browser is actually running your JS.

1. Render the book (`quarto render` from the project root — it's a book
   project, so `--to html -o ...` on a single file won't work; `-o` also
   rejects paths, see `AGENTS.md`).
2. Fast sanity check before even looking at pixels — grep the rendered page
   for the OJS runtime markers:
   ```bash
   grep -c "ojs-module\|observablehq\|quarto-ojs" docs/path/to/page.html
   ```
   A count of 0 means the engine never picked up your block — go re-check
   the `source` format (the footgun above) before doing anything else.
3. Only once that count is nonzero, actually look at it. Serve `docs/`
   locally and screenshot it with headless Chrome, then read the screenshot
   image back in (the model can view images directly):
   ```bash
   python3 -m http.server 8911 --directory docs &
   "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
     --headless --disable-gpu --screenshot=/tmp/check.png \
     --window-size=1000,3600 --virtual-time-budget=4000 \
     "http://127.0.0.1:8911/path/to/page.html"
   ```
   Use a tall `--window-size` (OJS content can be far down a long page) and
   `--virtual-time-budget` to give the OJS runtime time to actually execute
   before the screenshot is taken. Crop/read the image to confirm you see
   real rendered sliders and SVG panels, not highlighted source text.
4. If you can verify against known math (as with the heart-transform
   widget: the three default slider values should reproduce the exact
   static figure already in the page), do that numerically in a throwaway
   Python/Node script *before* touching the notebook — it's much faster to
   debug the transform logic in a REPL than by re-rendering the whole book
   each time, and it separates "is my math right" from "is Quarto executing
   my code," which are two different failure modes.
