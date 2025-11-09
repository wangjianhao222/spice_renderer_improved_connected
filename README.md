# SPICE Netlist Generator & Improved Planar Renderer

## Overview

This single-file web tool (`spice_renderer_improved_connected.html`) converts a simple SPICE-style netlist into a clear planar circuit diagram in the browser. It parses the netlist, constructs a node/edge graph, runs a force-directed layout to place nodes, and renders an SVG schematic with component symbols, accurate connector lines, a grid, and interactive label dragging.

## Quick demo netlist (preloaded)

```spice
V1 n001 0 DC 5
R1 n001 n002 1k
C1 n002 0 10uF
R2 n002 n003 2k
L1 n003 0 10uH
Q1 n003 n002 0 2N3904
```

Paste the text above into the netlist editor and click **Generate Diagram** to see a live example.

## UI layout (three-column)

* **Left column — Netlist editor + controls**

  * Multi-line textarea for SPICE-style netlist input.
  * Controls: `Generate Diagram`, `Auto-layout`, `Export SVG`, `Export PNG`, `Copy Netlist`.
  * Options: Zoom controls, Fit, Toggle Grid, Toggle Snap, Highlight Floating Nets.

* **Center column — SVG canvas**

  * Scalable SVG viewport for rendering the circuit: grid, wires, components, node dots, and overlays.
  * Pan and zoom interactions (mouse wheel zoom, middle-drag or Alt+drag to pan).

* **Right column — Inspector**

  * Parsed component list and nets summary with reference counts to help identify floating nets.

## Core features

### Parsing

* `parseNetlist(text)` reads the netlist line-by-line and recognizes components by their leading character:

  * Two-terminal parts: `R, C, L, D, V, I` → `{id, type, nodes:[n1,n2], value}`.
  * Transistors: `Q` (BJT) and `M` (MOSFET) parse 3–4 terminals.
  * Generic ICs and subcircuits (`U`) accept multiple node tokens.
* Lines beginning with `*` or `;` are treated as comments.

### Graph construction

* `buildGraph()` creates node objects keyed by net name and edge objects for connections.
* Nodes hold metadata: `name`, `x, y` (initial random positions), and `refs` (components that reference the node).

### Layout algorithm

* `layoutGraph(iters)` runs a force-directed layout combining pairwise repulsion and edge springs.
* The algorithm aims for a target edge length and iterates to reduce overlaps and create readable spacing.

### Rendering

* `render()` draws:

  * Optional grid (configurable step size).
  * Wires as straight SVG lines between node centers.
  * Component symbols placed at edge midpoints and rotated to match connection angles.
  * Short connector lines connecting component endpoints to exact node centers so visuals join cleanly.
  * Node dots and draggable node labels.

### Supported symbols

* Resistor (zig-zag), capacitor (two plates), inductor (series arcs), diode/LED, voltage/current sources (labeled circle), BJT/MOSFET symbols, generic IC outlines, and ground symbol.

## Interaction & controls

* **Generate Diagram**: parse netlist, build graph, run layout, and render.
* **Auto-layout**: re-run the layout to refine positions while preserving manual adjustments unless reset.
* **Zoom / Pan**: zoom to cursor; pan with middle mouse or Alt+drag.
* **Toggle Grid / Snap**: grid uses a fixed step; when Snap is on, dragging labels snaps to the grid.
* **Drag node labels**: click-and-drag a node label to move the node; the SVG updates live.
* **Highlight Floating Nets**: visually mark nets with one or zero references to find unconnected nodes quickly.

## Export & clipboard

* **Export SVG**: serializes the SVG for download (`diagram.svg`).
* **Export PNG**: rasterizes the SVG into a PNG (`diagram.png`).
* **Copy Netlist**: copies the netlist editor contents to the clipboard.

## Developer notes (key functions & state)

* Global state variables include: `components`, `graph`, `snap`, `gridOn`, `scale`, `panX`, and `panY`.
* Important functions:

  * `parseNetlist(text)` → fills `components`.
  * `buildGraph()` → constructs `graph.nodes` and `graph.edges`.
  * `layoutGraph(iters)` → runs the force solver.
  * `render()` → draws the canvas and components.

## Limitations & tips

* The parser is intentionally simple: expect one component per line with whitespace-separated tokens. Advanced SPICE syntax (line continuations, complex `.param`, nested subckt definitions) is not supported.
* Force-directed layout works well for small to medium nets; very large nets may take longer and may require manual label adjustments.
* Symbols are schematic approximations for documentation — not PCB footprints.

## Example workflow

1. Paste your SPICE-style netlist into the left editor.
2. Click **Generate Diagram** to parse and render.
3. Use **Auto-layout** to refine, drag labels to fine-tune positions.
4. Toggle grid/snap for tidy alignment.
5. Export as SVG or PNG for insertion into documentation.

## Next steps you might ask for

* Produce a `README.md` file ready for download.
* Generate a compact quick-start cheat sheet.
* Add additional symbol types or support for common SPICE directives.

---

*End of introduction.*
