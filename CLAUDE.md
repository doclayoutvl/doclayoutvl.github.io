# DocLayout-VL — ECCV 2026 Paper Website

## Project overview
Static website for the ECCV 2026 paper **"DocLayout-VL: A Foundational Model for Hierarchical, Open-Set and Promptable Document Layout Segmentation"**.

Authors: Venkata Kesav Venna¹·*, Srihari Bandarupalli²·*, Anirudh Srinivasan¹·*, Raghuveer R¹, Sai Madhusudan Gunda², Ravi Kiran Sarvadevabhatla¹·²  
Affiliations: ¹ BharatGen, ² IIIT Hyderabad

---

## File structure

```
DocLayout/
├── index.html          # Main website (single-page)
├── annotate.html       # Polygon annotation tool (local use only)
├── examples.md         # Ground-truth prompts + model outputs for all 6 demo examples
├── static/
│   ├── images/         # Static figures (e.g. doclayoutvl_tasks.png)
│   └── examples/       # Demo document images
│       ├── Task_A.jpeg
│       ├── Task_B1.png
│       ├── Task_B2.png
│       ├── Task_C.png
│       ├── Task_D1.webp
│       └── Tasl_D2.jpg     ← note typo in filename (Tasl not Task)
└── reference/          # Reference website from a previous paper (design inspiration)
```

---

## index.html — Interactive Demo

### Section location
The `#demo` section HTML (tabs, image panel, chat panel) is at roughly lines 412–448.  
The CSS for the demo is at roughly lines 168–213.  
The demo JavaScript block is at roughly lines 888–1175 (wrapped in an IIFE).

### Data structure — examples

```javascript
const DE = [
  {
    id: 'task_a',
    label: 'Task A',
    image: 'static/examples/Task_A.jpeg',
    question: '...',         // shown in the user bubble
    spans: [                 // one entry per [LAY] token
      {
        label: 'article_title',
        masks: [{ points: [[x1,y1],[x2,y2],...] }]  // coords in 0-100 % space
      },
      ...
    ],
    tokens: [...]            // token stream (see below)
  },
  ...
];
```

**Coordinate space:** All polygon points are in `0-100` percentage space matching `viewBox="0 0 100 100" preserveAspectRatio="none"` on the SVG overlay. X = % of image width, Y = % of image height.

### Token stream format

Each example has a `tokens` array that drives the character-by-character streaming output:

| Token type | Fields | Effect |
|---|---|---|
| `{type:'text', content:'...'}` | `content` | Stream characters one by one |
| `{type:'span_start', spanIndex:N}` | `spanIndex` | Start coloring text with DC[N] color |
| `{type:'span_end'}` | — | Stop coloring |
| `{type:'lay', spanIndex:N}` | `spanIndex` | Emit a `[LAY]` chip + trigger polygon sweep-in |

### Color palette (DC array)

```javascript
const DC = [
  {fill:'rgba(21,149,143,.18)', fillBright:'rgba(21,149,143,.55)', border:'#15958f', text:'#0b7f79'},
  {fill:'rgba(23,111,138,.18)', fillBright:'rgba(23,111,138,.55)', border:'#176f8a', text:'#125e7b'},
  {fill:'rgba(245,166,35,.18)', fillBright:'rgba(245,166,35,.55)', border:'#e09610', text:'#9a6a00'},
  {fill:'rgba(168,85,247,.18)', fillBright:'rgba(168,85,247,.55)', border:'#a855f7', text:'#7822c4'},
  {fill:'rgba(255,107,53,.18)', fillBright:'rgba(255,107,53,.55)', border:'#e05520', text:'#b03a10'},
  {fill:'rgba(72,199,142,.18)', fillBright:'rgba(72,199,142,.55)', border:'#2eaa78', text:'#1e8a5e'},
];
```

Colors cycle with `DC[spanIndex % DC.length]`.

### Polygon reveal animation

Each polygon group (`<g class="demo-hl-poly">`) starts hidden.  
`revealHL(si)` adds class `sweeping` → CSS `clip-path: inset(0 100% 0 0)` → `inset(0 0% 0 0)` sweeps it in left-to-right over 0.6s.

### [LAY] chip click behavior

Clicking a `[LAY]` chip calls `toggleHL(si)`:
- If not yet revealed → `revealHL(si)` (sweep animation)
- If already revealed → `popHL(si)` (flash `fillBright` + stroke-width 1.5 for 380ms, then reset)

### Streaming

`flatTokens()` expands the token array into a flat event list. A `setInterval(25ms)` ticks through events one by one. Note: browsers throttle `setInterval` to ~1s when the tab is in the background — normal behavior, works at full speed when focused.

---

## Six demo examples

| ID | Task | Image | Spans | Output format |
|---|---|---|---|---|
| Task A | Layout as Segmentation | Task_A.jpeg | 12 | flat `label [LAY]\n` |
| Task B1 | Open-Set Generic Ontology | Task_B1.png | 12 | flat `label [LAY]\n` |
| Task B2 | Open-Set Domain-Specific (insurance) | Task_B2.png | 11 | flat `label [LAY]\n` |
| Task C | Hierarchical Segmentation | Task_C.png | 14 | XML tree with `[LAY]` tokens inline |
| Task D1 | Targeted PII Extraction | Task_D1.webp | 13 | flat `label [LAY]\n` |
| Task D2 | Spatial Reasoning | Tasl_D2.jpg | 1 | prose answer + `Final Answer: Figure 2 [LAY]` |

### Task C token structure (XML hierarchical)

```
<document>
  <header>
    fund-title [LAY]    ← spanIndex 0
    sub-title [LAY]     ← spanIndex 1
    issue-data-strip [LAY]
    logo [LAY]
  </header> [LAY]       ← spanIndex 4 (the </header> tag itself is a span)
  <body>
    fund-info-table [LAY]
    <performance-section>
      old-line-chart [LAY]
      new-line-chart [LAY]
      source [LAY]
    </performance-section> [LAY]
    asset-allocation-section [LAY]
  </body> [LAY]
  <footer>
    page-number [LAY]
  </footer> [LAY]
</document>
```

---

## annotate.html — Polygon Annotation Tool

A standalone browser tool for drawing polygon coordinates on each document image.

**Open:** `http://localhost:<port>/annotate.html` (serve from the DocLayout root)

### Features
- Sidebar: example tabs, span list with ✓/○ status, progress bar
- Click to place polygon vertices on the image
- **Enter** or **double-click** → close polygon, auto-save, auto-advance to next unannotated span
- **Backspace** → undo last point
- **Esc** → cancel current draw (stays in draw mode)
- **Tab** / **Arrow keys** → navigate between spans
- **Right-click** → undo last point
- All annotations persist in `localStorage` key `doclayout_annot`
- "Export this example" / "Export all examples" → generates `spans:[...]` JS code to paste into `index.html`

### Annotation coordinate format

Polygons are stored as `[[x,y], ...]` where x,y are in the 0-100 percentage space.  
Export output looks like:
```javascript
{label:'article_title',       masks:[{points:[[5.2,3.1],[94.8,3.1],[94.8,8.4],[5.2,8.4]]}]},
```
Paste the exported `spans:[...]` block into the matching example in `index.html` (around line 908 for Task A, etc.).

### localStorage schema

```json
{
  "task_a_0":  [[x1,y1],[x2,y2],...],
  "task_a_1":  [[x1,y1],...],
  "task_b1_0": [[x1,y1],...],
  ...
}
```
Key format: `<example_id>_<spanIndex>` (e.g. `task_c_4`).

---

## Pending work

- [ ] **Fill in real polygon coordinates** for all spans across all 6 examples. Currently all use rectangular placeholders. Use `annotate.html` to draw accurate masks, then export and paste into `index.html`.

---

## Stack / dependencies

- Pure HTML/CSS/JS — no build step
- Bulma CSS (CDN)
- Fonts: Inter, JetBrains Mono (Google Fonts CDN)
- CSS variables: `--ink`, `--muted`, `--line`, `--soft`, `--blue`, `--teal`, `--navy`, `--ours`
- SVG for polygon overlays (`viewBox="0 0 100 100"`, `preserveAspectRatio="none"`)
