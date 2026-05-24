# Entangled

A live-preview editor for CrochetPARADE patterns. Type instructions on the
left; see the standard crochet chart on the right, with real stitch symbols
(chains, slip stitches, sc/hdc/dc/tr/dtr/trtr, post stitches, puffs, bobbles,
popcorns, etc.) — exactly the symbol library the original CrochetPARADE uses
for its SVG export.

This is a stripped-down spin-off of
[CrochetPARADE](https://www.crochetparade.org/) by Svetlin Tassev. The 3D
rendering, Pyodide translator, GLTF export, periphery analyzer, manual,
example dropdown, and animation features have been removed. What remains is
the bare pipeline:

```
your text  →  parse64.js / simplify64.js   →  graph + DOT
DOT        →  graph64.wasm (force-directed 2D layout)  →  node positions
positions  →  main.js (places standard crochet symbols along the edges
              of the graph using SVG.js, matching the original .svg export)
```

## Running it

Because `graph64.js` loads `graph64.wasm` over `fetch()`, you need to serve
the folder over HTTP — opening `index.html` directly with `file://` won't
work in most browsers. Any static server works, for example:

```
cd crochet-sandbox
python3 -m http.server 8000
# then open http://localhost:8000
```

## How to use

- Edit the textarea on the left. Stop typing and the chart re-renders
  ~250 ms later.
- The status pill in the header turns plum while compiling, green when
  done, raspberry on error.
- An error overlay slides up over the canvas when the parser rejects your
  pattern. The last successful chart stays visible behind it so you can
  see what you're editing against.
- **Save SVG** downloads the current chart as a portable `.svg` file.

## How the chart is drawn

For each non-hidden stitch node in the parsed graph:

1. Find the two **blue** edges flanking it — these are the same-row links to
   the neighbouring stitches and define the stitch's position and angle.
2. For chains (`ch`) and rings (`ring`), draw the chain symbol between
   those flanks.
3. For everything else, draw a faint guide line (and, for stitches like
   `hdc` / `dc` / `tr` etc., a short black top-bar).
4. Then walk each incoming **red** edge — the within-stitch vertical link —
   and place the matching stitch symbol along it, scaled to ~90% of the
   edge length and rotated to match its angle. Cluster stitches (puffs,
   bobbles, popcorns) walk further back along the red graph to find their
   visual base.

The symbol path data is copied verbatim from CrochetPARADE / `mesh64.js`.

## File map

| File             | Purpose                                                |
| ---------------- | ------------------------------------------------------ |
| `index.html`     | Layout and script loading order                        |
| `style.css`      | All styling, organised with BEM class names            |
| `main.js`        | Entry point — parser-warning suppression, emscripten   |
|                  |   Module config, and the sandbox glue: parse →         |
|                  |   layout → place symbols, debouncing, download.        |
|                  |   Includes the full stitch symbol library.             |
| `parse64.js`     | Verbatim from CrochetPARADE: lexes the grammar         |
| `simplify64.js`  | Verbatim from CrochetPARADE: helpers used by parse64   |
| `graph64.js`     | Verbatim from CrochetPARADE: emscripten WASM loader    |
| `graph64.wasm`   | Verbatim from CrochetPARADE: the 2D layout solver      |
| `svg.min.js`     | Verbatim from CrochetPARADE: SVG.js v2, used for       |
|                  |   bounding-box math and transform composition          |

## CSS naming convention

Styles use [BEM](https://getbem.com/) (`block__element--modifier`). The
blocks are:

| Block        | Purpose                                          |
| ------------ | ------------------------------------------------ |
| `.header`    | The top bar                                      |
| `.brand`     | Title + tagline cluster on the left of `.header` |
| `.controls`  | Status pill + Save-SVG button on the right       |
| `.workspace` | The two-pane grid below the header               |
| `.pane`      | One side of the workspace (Pattern or Chart)     |
| `.editor`    | The pattern textarea                             |
| `.canvas`    | The grid-paper chart area                        |
| `.error`     | Parser-error overlay anchored inside `.canvas`   |
| `.splash`    | Full-screen loading overlay shown at boot        |

State is expressed via modifier classes the JS toggles:
`.controls__dot--ok / --err / --busy`, `.error--visible`,
`.splash--hidden`.

The original CrochetPARADE code is GPLv3.
