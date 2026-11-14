# AGENTS.md

Yair Mau's "double cover" book — a Quarto book, in the same family as `mishmash`
(served under `yairmau.com`), where he works through topics in depth and writes
up his own take. The working title comes from the $SU(2) \to SO(3)$ double
cover, which shows up explicitly once the spinor chapters connect to the Lie
algebra material.

## Why this repo exists

This isn't a reference or a textbook summary. The point is for Yair to *grok*
the material — reworking someone else's videos, papers, or examples is a
normal and expected part of learning here, not something to avoid. Two goals
matter more than anything else when drafting or reviewing content:

1. **No deus ex machina.** Every derivation should feel like something the
   reader could have come up with themselves, step by step, not a formula that
   drops out of nowhere "because that's the definition." If a result needs a
   trick, show where the trick comes from or why it's the natural thing to try
   — don't just assert it and move on. When a draft has a gap like this, it's
   worth more rework before being called done.
2. **Visual, interactive explanations are a first-class goal, not a nice-to-have.**
   Yair is a visual thinker. Good diagrams, animations, and interactive
   widgets (sliders, draggable points, live-updating plots) pull a lot of
   weight here and are worth real effort — more than they'd be worth in a
   purely textual math reference. Prefer an OJS/Observable widget or a
   tikz/d3 diagram over a wall of equations whenever a geometric idea is being
   explained. See `_extensions/` for what's already wired in (tikz is used
   for the Lie algebra diagrams).

## Structure

- `_quarto.yml` defines the book. Chapters are grouped into named **parts**:
  - **"math fundamentals"** (`math_fundamentals/`) — general-purpose linear
    algebra / Lie theory facts (e.g. $\det(\exp A) = \exp(\operatorname{tr} A)$,
    conjugation) that are prerequisites for more than one later part. One
    narrowly-scoped page per fact or technique, not a grab-bag notebook — if a
    page starts covering two unrelated things, split it.
  - **"spinors for beginners"** (`spinors_for_beginners/`) — working through
    eigenchris' *Spinors for Beginners* YouTube series (transcripts in
    `archive/transcripts/`) in Yair's own words and derivations. Source
    material gets reworked, not transcribed — see goal 1 above.
  - **"SL(2,C)"** (`sl2c/`) — the original Lie algebra chapters (`sl2.ipynb`,
    `real_forms_of_sl2.ipynb`, `su2_and_spinors.ipynb`), written before the
    spinors-for-beginners material and conceptually upstream of it.
  - New chapters should go into the subdirectory for one of these parts, or a
    new named part with its own subdirectory if the topic doesn't fit either
    — don't add bare top-level chapters alongside the parts.
- Each part gets its own subdirectory, one level deep, holding every chapter
  file (`.qmd`/`.ipynb`) that belongs to that part plus any assets those
  chapters need (e.g. `sl2c/DrawingHands.jpg`, `sl2c/sl2_files/`). This keeps
  the repo root from accumulating every chapter from every part as the book
  grows. Don't nest further than one level, and don't mix chapters from two
  parts into the same directory.
- `archive/` holds material that isn't part of the book proper: reference
  transcripts (`archive/transcripts/`), the logo source notebook and exported
  SVGs (`archive/logo/`). Quarto only renders what's listed under
  `book: chapters:` in `_quarto.yml`, so `archive/` is safe for scratch
  material.

## Build and deploy

- `quarto render` builds the whole book into `docs/`. GitHub Pages serves
  `docs/` from `main`, so **pushing to `main` publishes the book**. Ask before
  pushing.
- Commit the rebuilt `docs/` together with the source changes that produced
  it. `docs/.nojekyll` must stay in place.
- The `tikz`, `iconify`, and `fontawesome` extensions in `_extensions/` are
  required to render the Lie algebra notebooks (tikz diagrams) — don't delete
  `_extensions/` even though it looks like vendored boilerplate.
- During a render, stray `*.html` files or a `site_libs/` directory can appear
  at the repo root if a render is interrupted. Delete those and re-render;
  never commit root-level `site_libs/` or generated `.html` files outside
  `docs/`.

## Look and feel

- Same Midnight palette and layout as `mishmash` and `yairmau.github.io`:
  `custom.scss` carries the Sass variables and link-underline rules, `styles.css`
  has the curved-paper image effect, `kate-custom.theme` is the code highlight
  theme, and the sidebar has a "← Back to yairmau.com" link plus the logo from
  `archive/logo/`.
- Favicon and sidebar logo are set in `_quarto.yml` (`favicon:`, `sidebar.logo:`)
  and point at SVGs in `archive/logo/`, exported from
  `archive/logo/make_logo.ipynb`. Don't hand-edit the SVGs — change the
  notebook and re-export.
- Keep the same visual language as `mishmash`: no decorative Bootstrap
  `alert-*` boxes, no hover-zoom effects.

## Content conventions

- Chapters open with a one-line `#` title (no YAML front matter needed for
  plain `.qmd` chapters — the book-level `format:` in `_quarto.yml` already
  applies). Notebooks (`.ipynb`) keep their own front-matter cell since they
  need `execute:`/`format:` options for code.
- Write in first person, as working notes — matching the tone of `index.qmd`
  ("half-baked thoughts and learning notes"). It's fine for a chapter to say
  "I found this confusing until..." or "here's the way I'd derive this myself."
- Math: use LaTeX (`$...$`, `$$...$$`); the book renders math with KaTeX
  (`html-math-method: katex` in `_quarto.yml`), not the default MathJax.
- When reworking a source (a video, paper, or someone else's explanation),
  cite it in passing in the prose (e.g. "I'm working through eigenchris'
  *Spinors for Beginners* series") rather than as a formal citation — this
  book doesn't use `references.bib`.
