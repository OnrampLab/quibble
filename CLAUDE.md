# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

quibble is a single-file, zero-dependency, no-build vanilla-JS commenting layer.
A page includes `quibble.js` via one `<script>` tag; the human selects text or
clicks an element, leaves comments (persisted in `localStorage`), and exports
them as structured JSON to feed back to an AI agent. The whole product is
`quibble.js` — there is no framework, bundler, package manager, or transpile step.

## Commands

There is no build. CI (`.github/workflows/ci.yml`) does exactly two things, and
they are the canonical local checks too:

```bash
node --check quibble.js                                              # syntax check
node -e "JSON.parse(require('fs').readFileSync('examples/feedback.sample.json','utf8'))"  # validate sample JSON
```

Testing is manual: open `demo/index.html` in a browser and run the checklist at
the bottom of `README.md` (select text → comment → reload re-anchors → jump →
delete element → orphaned). There is no automated test suite.

## Distribution & releases

Consumers load the file straight from the repo via jsDelivr, pinned to the moving
`@v1` tag — nothing is published to npm:

```
https://cdn.jsdelivr.net/gh/OnrampLab/quibble@v1/quibble.js
```

Release process (single-repo, CDN-pinned — does NOT use cl-release-manager):
- **RC** = pre-release git tag `v1.<minor>.0-rc.<N>`, tested via its own CDN URL
  (`@v1.1.0-rc.1`).
- **Promote** by tagging final `v1.<minor>.0` and moving the `v1` tag to it, so
  `@v1` picks up the release. Do not move `v1` until promotion.

Because `@v1` is the live URL for every existing page, treat `quibble.js` as a
published API: the script-tag attributes (`data-project`, `data-storage-key`) and
the exported JSON schema are a contract. The header comment block at the top of
`quibble.js` and the schema in `README.md` document that contract for AI agents
consuming the export — keep all three (header, README, `examples/feedback.sample.json`)
in sync when the schema changes.

## Architecture

Everything is one IIFE in `quibble.js`, guarded by `window.__quibbleLoaded`.
Config is read from the `<script>` tag's data attributes. State is an array of
comment records persisted under `localStorage` key `quibble:<project>`.

The non-obvious pieces that span the file:

- **Two anchor strategies.** A comment targets either text (`{type:"text", quote}`)
  or an element (`{type:"element", selector, tag, snippet}`). On every load,
  `reapply()` re-resolves both against the current DOM. Text re-anchors by
  *content*: `findRange()` flattens all page text (whitespace-normalized) via a
  TreeWalker and searches for the stored quote — so it survives DOM restructuring
  as long as the prose is unchanged. Elements re-anchor by `computeSelector()`'s
  `nth-of-type` path (or `#id` when unique). A target that no longer resolves is
  rendered **orphaned** (greyed, kept) rather than dropped — see `isOrphan()`.

- **Highlighting has two backends.** `HL_OK` gates the CSS Custom Highlight API
  (handles multi-element ranges without mutating the DOM); the fallback wraps the
  range in a `<mark>`. Every highlight/flash/remove path branches on `HL_OK`.

- **Top-layer host (`qbRoot` + `rehome()`).** All quibble surfaces live in one
  `qb-root` container. Modal `<dialog>` (showModal) and fullscreen elements paint
  in the browser top layer above *everything* regardless of z-index; the only way
  to sit above them is to be inside them. So a MutationObserver watches for open
  modals (`:modal`) and moves `qbRoot` into the topmost one, falling back to
  `<body>`. Overlay-style modals (plain high-z-index divs) are instead beaten by
  quibble's own near-`2147483647` z-indexes. This is issue #4 — be careful editing
  `rehome`/`topLayerHost`/z-index values.

- **Element rings are positioned overlays**, not DOM wrappers: `rings[id]` holds a
  fixed-position `div` re-laid-out on scroll/resize via a rAF-batched
  `layoutRings()`. Text uses ranges; elements use rings — the flash/jump/delete
  logic mirrors across both.

- **UI guards.** `isUI()` / `inUIText()` keep quibble's own chrome out of
  selection, picking, and text-search so it never comments on itself.

`demo/index.html` and `docs/index.html` are standalone HTML pages that include the
script for manual testing and the landing page respectively.
