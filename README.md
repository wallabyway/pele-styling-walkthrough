# Pele styling walkthrough

A minimal, working example of styling **stock [Pele](https://github.com/obsidianmd/pele)** without forking it, changing its core, or adding a plugin system.

**[Open the GitHub Pages walkthrough](https://wallabyway.github.io/pele-styling-walkthrough/)**

The included GitHub Pages demo covers:

- A paper-white Pele theme: documented CSS variables plus a grain pattern on cards
- Elevated, stacked shadows (contact, short, long) as SVG filters, lifting higher on hover
- Intro + loop motion in CSS: cards rise in turn, lines wipe in, then lines flow as moving dashes, with reduced-motion support and a Motion / Replay control
- Lucide icons supplied through Pele's existing `icons` option
- Rendered/source tabs for both the styled example and an unstyled Pele comparison
- A gallery of 14 diagrams (flowchart, ER, architecture, Gantt, sequence, state, class, pie, XY chart, timeline, mind map, git graph) with the same styling, adapted from the [Pele looks gallery](https://wallabyway.github.io/pele-looks-demo/)

There is no build step. The entire demo lives in [`index.html`](index.html).

## Run it locally

Serve the repository root with any static web server. For example:

```sh
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

Opening `index.html` directly may also work, but a local server better matches GitHub Pages. Pele and the page font are loaded from CDNs, so the demo needs a network connection.

## 1. Start with stock Pele

Load Pele's browser bundle and create a container for the diagram:

```html
<div id="diagram" class="pele-white"></div>
<script src="https://cdn.jsdelivr.net/npm/pele@1.12.0/dist/pele.global.js"></script>
```

The demo pins Pele 1.12.0 so its behavior is reproducible. You can use a newer compatible release after checking the [Pele documentation](https://pele.run/).

Render some Mermaid source:

```js
const source = `flowchart TD
  A[fa:fa-file-text Mermaid] --> B{Style it?}
  B -->|fa:fa-check Yes| C[fa:fa-zap Stock Pele]
  B -->|fa:fa-x No| D[fa:fa-book Defaults]`;

pele.mount(document.querySelector('#diagram'), source);
```

At this point the icon names are just text because no icon resolver has been supplied yet. The next sections add presentation without changing Pele.

## 2. Apply a paper-white theme

Pele reads documented CSS variables inherited by its generated SVG. Scope the theme with your own class so it only affects the diagrams you choose:

```css
.pele-white {
  --pele-bg: #ffffff;
  --pele-fg: #1d1c1a;
  --pele-muted: #76716a;
  --pele-line: #3f3c37;
  --pele-surface: #fefdfa;
  --pele-surface-alt: #f8f6f0;
  --pele-border: #ddd8cc;
  --pele-accent: #2f63ad;
  --pele-font: "Rethink Sans", sans-serif;
  --pele-radius: 8px;
}
```

For the paper texture, the page carries **one hidden `<svg>`** with `<defs>`: two noise filters (`pele-grain`, `pele-mottle`) layered into a `<pattern id="pele-paper">`. CSS on the page can point any inline SVG at those ids, so shapes Pele fills with its surface color can use the pattern, with the plain color as fallback:

```css
.pele-white .pele-node > [fill="var(--_s)"] { fill: url(#pele-paper) var(--pele-surface); }
.pele-white .pele-cluster > rect { fill: url(#pele-paper-alt) var(--pele-surface-alt); fill-opacity: 1; }
.pele-white svg.pele { overflow: visible; } /* let long shadows spill past the edge */
```

Hide the defs with zero size, not `display: none`, which disables its filters and patterns. See [Pele's theming documentation](https://pele.run/theming/) for the variables and SVG classes.

## 3. Add elevated shadows

The same hidden defs hold stacked shadow filters. Each layer blurs the shape's alpha, offsets it, tints it, and all layers merge under the shape:

| Filter | Layers `[dy, blur, opacity]` | Used on |
| --- | --- | --- |
| `pele-lift` | `[1,1.5,.34] [6,8,.22] [20,26,.26]` | nodes, slices |
| `pele-lift-hover` | `[1,1.5,.34] [10,12,.24] [30,38,.3]` | hovered nodes |
| `pele-sheet` | `[1,1,.22] [8,16,.16]` | clusters |
| `pele-tag` | `[.5,.5,.3] [1.5,1.5,.2]` | edge labels, git commits |
| `pele-bar` | `[.5,.5,.3] [2,2.5,.2]` | Gantt bars (short and close, so long bars don't smear) |

```css
.pele-white :is(.pele-node, .pele-task, .pele-slice, .pele-commit) { filter: url(#pele-lift); }
.pele-white :is(.pele-node, .pele-task):hover { filter: url(#pele-lift-hover); }
.pele-white .pele-cluster > rect { filter: url(#pele-sheet); }
.pele-white .pele-edge-label > rect { filter: url(#pele-tag); }
.pele-white .pele-task { filter: url(#pele-bar); }
```

Why SVG filters rather than CSS `drop-shadow()`? Safari does not draw CSS filter functions on shapes inside an SVG; `filter: url(#…)` works in every browser. Timelines and mind maps put text and see-through tints inside their nodes, so there only the solid card shapes lift.

## 4. Use Lucide icons

Pele's `icons` option is a synchronous callback that returns trusted SVG markup for an icon name. Pele supplies the icon's 24 × 24 frame, so a small map of Lucide shapes is enough:

```js
const lucideIcons = Object.freeze({
  'file-text': '<path d="M6 22a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h8a2.4 2.4 0 0 1 1.704.706l3.588 3.588A2.4 2.4 0 0 1 20 8v12a2 2 0 0 1-2 2z"/><path d="M14 2v5a1 1 0 0 0 1 1h5"/><path d="M10 9H8"/><path d="M16 13H8"/><path d="M16 17H8"/>',
  check: '<path d="M20 6 9 17l-5-5"/>',
  x: '<path d="M18 6 6 18"/><path d="m6 6 12 12"/>',
  zap: '<path d="M4 14a1 1 0 0 1-.78-1.63l9.9-10.2a.5.5 0 0 1 .86.46l-1.92 6.02A1 1 0 0 0 13 10h7a1 1 0 0 1 .78 1.63l-9.9 10.2a.5.5 0 0 1-.86-.46l1.92-6.02A1 1 0 0 0 11 14z"/>',
  book: '<path d="M4 19.5v-15A2.5 2.5 0 0 1 6.5 2H19a1 1 0 0 1 1 1v18a1 1 0 0 1-1 1H6.5a1 1 0 0 1 0-5H20"/>'
});

const icons = (name) => lucideIcons[name.replace(/^fa:fa-/, '')];

pele.mount(document.querySelector('#diagram'), source, { icons });
```

With this resolver, `fa:fa-check` selects the Lucide `check` shape. Returning `undefined` for an unknown name lets Pele render the diagram without that icon.

Only return markup from a source you trust. Do not build SVG strings from untrusted icon names or diagram text; Pele inserts icon markup without filtering it. See [Pele's security guidance](https://pele.run/security/).

The demo uses icon shapes from [Lucide](https://lucide.dev/), licensed under the ISC License. Lucide Icons and Contributors retain their copyright; some Lucide icons derive from Feather, whose MIT license credits Cole Bemis.

## 5. Animate: intro, then loop

Two classes on the container pick the motion: `pele-intro`, `pele-loop`, or both (the default here). The page's **Motion** select and **Replay** button toggle them; removing and re-adding the classes restarts every CSS animation.

- **Intro:** frames and titles fade in, nodes rise one after another, Gantt bars grow, slices and commits pop, lines are wiped in along the flow with `clip-path`, and edge labels pop in after them.
- **Loop:** once the intro is done (`--loop-delay: 2.2s`), every line becomes moving dashes. Lines that are already dashed, such as sequence replies, keep their own pattern.

The stagger is CSS only. `:nth-child()` gives each child of `.pele-nodes`, `.pele-edges`, and similar groups an index `--i`, and delays are computed from it:

```css
.pele-white :is(.pele-nodes, .pele-edges, .pele-edge-labels, .pele-tasks) > :nth-child(2) { --i: 1; }
/* … through :nth-child(n+12) { --i: 11; } */

@media (prefers-reduced-motion: no-preference) {
  .pele-white.pele-intro .pele-node {
    animation: pele-rise 520ms cubic-bezier(.2, .8, .2, 1.25) backwards;
    animation-delay: calc(150ms + var(--i) * 110ms);
  }
  .pele-white.pele-intro :is(g.pele-edge, path.pele-edge) {
    animation: pele-wipe 650ms ease-in-out backwards calc(350ms + var(--i) * 110ms);
  }
  .pele-white.pele-loop :is(.pele-edge > path, path.pele-edge):not(.pele-marker, [stroke-dasharray]) {
    stroke-dasharray: 5 4;
  }
  .pele-white.pele-loop :is(.pele-edge > path, path.pele-edge):not(.pele-marker) {
    animation: pele-flow 1s linear infinite var(--loop-delay);
  }
}

@keyframes pele-rise { from { opacity: 0; translate: 0 10px; filter: none; } to { filter: none; } }
@keyframes pele-wipe { from { clip-path: inset(0 100% 100% 0); } to { clip-path: inset(-20px); } }
@keyframes pele-flow { to { stroke-dashoffset: -18; } }
```

Two details keep this correct and smooth:

- Nodes move with the CSS **`translate`** property, which adds to the `transform` attribute Pele uses for placement rather than overriding it.
- Anything rising or popping has **`filter: none`** in its keyframes, so a blurred shadow is not redrawn on every frame; the shadow comes on when it lands. Cards hold still in the loop for the same reason, and only the lines flow.

All motion sits inside `prefers-reduced-motion: no-preference`.

## 6. Compare with original Pele

The bottom of `index.html` renders the same Mermaid source twice:

1. **Styled result:** the paper-white theme, elevated shadows, intro + loop motion, and Lucide resolver.
2. **Original Pele:** default Pele presentation, with only the same icon resolver so the comparison remains readable.

Below the styled result, a **gallery** renders 14 more diagrams covering every major Mermaid type with the same paper-white theme, elevated shadows, intro + loop motion, and Lucide resolver. The resolver there also accepts `fa:users`, mind map `fa fa-database`, and architecture names such as `internet` and `disk`, mapping them onto Lucide shapes. Each card has a collapsible **Mermaid source**.

Both examples have **Rendered result** and **Mermaid source** tabs. The tabs use vanilla JavaScript and support click, arrow keys, Home, and End.

## Publish with GitHub Pages

The repository is already a static Pages site; no workflow or build command is required.

1. Open the repository on GitHub.
2. Go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Select the `main` branch and `/ (root)`, then save.
5. Wait for GitHub to publish the site and show its URL.

GitHub's plan and repository-visibility rules apply. In particular, publishing Pages from a private repository may require a paid GitHub plan. Changing the repository to public is another option if its contents are meant to be shared.

## What this does not require

- No Pele fork
- No changes to Pele core
- No custom `look` or `animate` render options
- No framework or package installation
- No JavaScript animation library

It is just Pele's documented rendering API, CSS variables and classes, plus a trusted icon map.
