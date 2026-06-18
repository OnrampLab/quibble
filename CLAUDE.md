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

Testing is manual: open `docs/index.html` (the landing page doubles as the test
surface) in a browser and run the checklist at the bottom of `README.md` (select
text → comment → reload re-anchors → jump → delete element → orphaned → modal
dialog → fullscreen). There is no automated test suite.

## Distribution & releases

Consumers load the file straight from the repo via jsDelivr, pinned to the
floating `@latest` tag — nothing is published to npm:

```
https://cdn.jsdelivr.net/gh/OnrampLab/quibble@latest/quibble.js
```

Release process (single-repo, CDN-pinned — does NOT use cl-release-manager):
- **RC** = pre-release git tag `v1.<minor>.0-rc.<N>`, tested via its own CDN URL
  (`@v1.1.0-rc.1`).
- **Promote** by tagging a new, higher `v1.<minor>.0` (e.g. `v1.2.0`). jsDelivr's
  `@latest` floats to the newest stable tag automatically — no purge, no waiting.
- **Never** force-move an existing version tag and **never** maintain a literal
  `v1` git tag. A literal `v1` tag makes jsDelivr serve `@v1` as an **immutable**
  pin (`Cache-Control: immutable`, cached ~1 year), so moving it silently freezes
  the CDN on stale content. Always cut a brand-new version number instead.

The landing page is published by GitHub Pages via the `pages.yml` Actions workflow
(Pages source = "GitHub Actions", not branch/`docs`): it copies `docs/` to the
site root and the repo-root `quibble.js` to `/quibble.js`. So `docs/index.html`
loads quibble **same-origin** from `/quibble.js` by default (the current `main`
build, live ~1 min after a push), and its version picker can reload with
`?ver=X.Y.Z` to load a published release from jsDelivr instead. `quibble.js` stays
the single source of truth at the repo root — never add a second copy under `docs/`.

Because `@latest` is the live URL for every existing page, treat `quibble.js` as a
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

`docs/index.html` is the landing page **and** the manual test surface: it embeds a
sample draft plus top-layer test triggers (modal `<dialog>` + fullscreen, issue #4)
and a version picker. It loads `quibble.js` same-origin by default (see Distribution).
