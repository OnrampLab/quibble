# quibble

**Drop-in text + element commenting layer for AI-assisted iteration.** One script
tag turns any HTML page into a commentable surface: select text or click an element,
leave a comment, then **Copy JSON** and paste it back to your AI agent as structured
feedback.

No backend. No dependencies. No build step.

---

## The loop

1. Your AI agent drafts a document, mockup, or questionnaire as HTML.
2. You open the page (quibble is already included) and annotate it — comment on
   prose (text selections) or on UI elements (images, cards, buttons).
3. Comments persist in `localStorage` and are highlighted on the page.
4. Click **Copy JSON** → paste into your AI chat → the agent regenerates the HTML.

> _quibble with the draft, send the quibbles back to the AI._

## Install

Add one script tag. Served straight from the repo via jsDelivr — nothing to publish:

```html
<script src="https://cdn.jsdelivr.net/gh/onramplab/quibble@v1/quibble.js"
        data-project="my-mockups"></script>
```

Or self-host: drop `quibble.js` next to your HTML and
`<script src="quibble.js" data-project="my-mockups"></script>`.

| Attribute          | Default              | Purpose                                            |
| ------------------ | -------------------- | -------------------------------------------------- |
| `data-project`     | `untitled`           | Names the project; appears in the exported JSON.   |
| `data-storage-key` | `quibble:<project>`  | Override the `localStorage` namespace.             |

A floating **Element** / **Feedback** bar appears in the bottom-right corner.

## Release candidate — v1.1 (config panel)

> ⚠️ **Pre-release.** `@v1` stays on the current stable build until this is promoted.

A **Settings** panel (⚙ gear on the bar) is in testing. It adds, all persisted in
`localStorage`:

- **Default mode on load** — arm **Element** picking automatically ([#2](https://github.com/OnrampLab/quibble/issues/2)).
- **Export format** — **JSON**, **YAML**, or **Markdown**; Copy and Export both follow it ([#1](https://github.com/OnrampLab/quibble/issues/1)).
- **Theme** — accent / bar colors and font.
- **Project / storage key** — rename or re-namespace without touching the script tag.

**Try it live:** **<https://onramplab.github.io/quibble/rc.html>**

Or pin the release candidate yourself (defaults match `@v1`, so JSON stays default):

```html
<script src="https://cdn.jsdelivr.net/gh/OnrampLab/quibble@v1.1.0-rc.1/quibble.js"
        data-project="my-mockups"></script>
```

## Usage

- **Comment on text:** select any text → click **Comment** → type → **Save**.
  (Cmd/Ctrl+Enter saves.)
- **Comment on an element:** click **Element** in the bar, then click any element
  on the page (image, button, card…). Press **Esc** to cancel.
- **Review:** click **Feedback** to open the panel — jump to a comment, delete it,
  **Copy JSON**, **Export** to a file, or **Clear** all.

Comments survive reloads and re-anchor automatically. If the AI regenerates the
page and a comment's target no longer exists, it is marked **orphaned** (greyed,
"⚠ target not found") rather than dropped — so you never silently lose feedback.

## Exported JSON

```json
{
  "tool": "quibble",
  "project": "my-mockups",
  "exportedAt": "2026-06-17T10:30:00.000Z",
  "count": 2,
  "quibbles": [
    {
      "id": "q_abc123",
      "target": { "type": "text", "quote": "the exact selected text" },
      "page": "index.html",
      "title": "Driftwood Coffee — Subscription",
      "section": "Nearest heading above the selection",
      "comment": "Lead with this — it's the strongest selling point.",
      "ts": "2026-06-17T10:28:00.000Z"
    },
    {
      "id": "q_def456",
      "target": {
        "type": "element",
        "selector": "body > main > section:nth-of-type(1) > div",
        "tag": "div",
        "snippet": "PRODUCT PHOTO"
      },
      "page": "index.html",
      "title": "Driftwood Coffee — Subscription",
      "section": "Get started",
      "comment": "Replace the placeholder with a real product photo.",
      "ts": "2026-06-17T10:29:00.000Z"
    }
  ]
}
```

Paste this back to your agent with a prompt like _"Here is my feedback as quibble
JSON — please revise the HTML accordingly."_ See
[`examples/feedback.sample.json`](examples/feedback.sample.json).

## Theming

quibble ships a clean self-contained look (no icon fonts — icons are inline SVG)
and adapts to light/dark. Override any `--quibble-*` CSS variable on `:root` to
match your design system:

```css
:root {
  --quibble-accent: #6366f1;       /* highlight + active controls */
  --quibble-surface: #111827;      /* floating bar / bubble background */
  --quibble-on-surface: #f9fafb;
  --quibble-panel: #ffffff;        /* review panel background */
  --quibble-on-panel: #111827;
  --quibble-font: "Inter", system-ui, sans-serif;
}
```

Full list: `--quibble-highlight`, `--quibble-highlight-flash`, `--quibble-accent`,
`--quibble-on-accent`, `--quibble-surface`, `--quibble-on-surface`,
`--quibble-border`, `--quibble-panel`, `--quibble-on-panel`,
`--quibble-border-panel`, `--quibble-muted`, `--quibble-font`.

## Browser support

Text highlights use the [CSS Custom Highlight API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Custom_Highlight_API)
where available (Chrome/Edge/Safari), with a `<mark>` fallback for older browsers.
Element rings work everywhere.

## Demo

Open [`demo/index.html`](demo/index.html) in a browser. Manual test checklist:

- [ ] Select text → comment → it highlights and counts in the bar
- [ ] Element mode → click the image/card → ring appears
- [ ] Reload → both comments re-anchor
- [ ] Click a comment in the panel → page scrolls and flashes the target
- [ ] Copy JSON / Export / Clear
- [ ] Delete an element, reload → its comment shows as orphaned

## License

[MIT](LICENSE) © onramplab
