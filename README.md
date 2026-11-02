# Pele styling walkthrough

A minimal, working example of styling **stock [Pele](https://github.com/obsidianmd/pele)** without forking it, changing its core, or adding a plugin system.

**[Open the GitHub Pages walkthrough](https://wallabyway.github.io/pele-styling-walkthrough/)**

The included GitHub Pages demo covers:

- A white Pele theme using documented CSS variables
- SVG drop shadows using Pele's element classes
- A small CSS entrance animation with reduced-motion support
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

## 2. Apply a white theme

Pele reads documented CSS variables inherited by its generated SVG. Scope the theme with your own class so it only affects the diagrams you choose:

```css
.pele-white {
  --pele-bg: #ffffff;
  --pele-fg: #1a1a1a;
  --pele-muted: #52615a;
  --pele-line: #1a1a1a;
  --pele-surface: #ffffff;
  --pele-surface-alt: #edf3f0;
  --pele-border: #9aaca3;
  --pele-accent: #176b4e;
  --pele-font: "Rethink Sans", sans-serif;
  --pele-radius: 8px;
}
```

These are Pele's own theme variables. See [Pele's theming documentation](https://pele.run/theming/) for the complete list and the available SVG classes.

## 3. Add drop shadows

Use CSS `filter: drop-shadow()` for SVG elements. Pele exposes `.pele-node` for each flowchart node:

```css
.pele-white {
  --pele-drop-shadow: 3px 4px 5px rgb(0 0 0 / 22%);
}

.pele-white .pele-node {
  filter: drop-shadow(var(--pele-drop-shadow));
}
```

`--pele-drop-shadow` is a **custom variable introduced by this stylesheet**, not a built-in Pele option. Declaring it alone has no effect; the `filter` declaration is what consumes it.

The general CSS syntax is:

```css
filter: drop-shadow(offset-x offset-y blur-radius color);
```

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

## 5. Add a small animation

Pele's generated elements can be animated with ordinary CSS:

```css
@keyframes pele-fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

.pele-white .pele-node {
  animation: pele-fade-in 420ms ease-out both;
}

.pele-white .pele-edge {
  animation: pele-fade-in 420ms 100ms ease-out both;
}

@media (prefers-reduced-motion: reduce) {
  .pele-white .pele-node,
  .pele-white .pele-edge {
    animation: none;
  }
}
```

This example animates opacity rather than CSS `transform`. SVG groups may already use transform attributes for positioning, and a CSS transform can override that placement. If you add movement, test every diagram type you intend to support.

## 6. Compare with original Pele

The bottom of `index.html` renders the same Mermaid source twice:

1. **Styled result:** the white theme, shadows, animations, and Lucide resolver.
2. **Original Pele:** default Pele presentation, with only the same icon resolver so the comparison remains readable.

Below the styled result, a **gallery** renders 14 more diagrams covering every major Mermaid type with the same white theme, shadows, animation, and Lucide resolver. The resolver there also accepts `fa:users`, mind map `fa fa-database`, and architecture names such as `internet` and `disk`, mapping them onto Lucide shapes. Each card has a collapsible **Mermaid source**.

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
