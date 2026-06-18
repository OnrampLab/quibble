# Pages-as-CDN + version switch + demo consolidation

**Date:** 2026-06-18
**Branch:** `feat/pages-cdn-version-switch`

## Goal

1. The landing page loads **our own** `quibble.js` same-origin (not jsDelivr).
2. A version `<select>` lets you load any published release from jsDelivr instead.
3. Remove `demo/` and `docs/rc.html`; migrate the dialog + fullscreen top-layer
   tests (issue #4) into `docs/index.html` first.

## Hosting & deploy

- New `.github/workflows/pages.yml`: on push to `main`, assemble `_site/` =
  `docs/` contents at root + repo-root `quibble.js` copied to `/quibble.js`, then
  `upload-pages-artifact` + `deploy-pages`.
- Pages source switched from "branch `main` / `docs`" to **GitHub Actions**.
- Result: `…/quibble/` = landing page; `…/quibble/quibble.js` = current build,
  same-origin, ~1 min after each push. Repo-root `quibble.js` stays the single
  source of truth; jsDelivr `@x.y.z` pinned URLs unaffected.

## Version switch (reload-based)

- Loader: `?ver` absent/`local` → `src="quibble.js"` (same-origin, default);
  `X.Y.Z` → jsDelivr `@X.Y.Z`.
- `<select>`: first option **"This build (local)"**; remaining options are
  published versions fetched from the jsDelivr data API (`· jsDelivr`,
  pre-releases tagged). `change` reloads with/without `?ver=`.
- quibble is a single-load IIFE with no teardown API, so switching reloads the
  page rather than hot-swapping in place.

## Dialog + fullscreen tests (migrated, restyled without Tailwind)

- "Top-layer tests (issue #4)" block in the demo with **Open modal dialog** and
  **View hero fullscreen** buttons.
- Native `<dialog class="qb-modal">` styled with the landing page's own CSS;
  click-outside-to-close preserved (verifies quibble chrome doesn't bubble-close).
- Fullscreen button calls `requestFullscreen()` on `.hero-img` (verifies the
  `fullscreenchange` rehome path).

## Removals & doc updates

- Delete `demo/index.html`, `demo/head-load.html`, `docs/rc.html`.
- `quibble.js` header: one-line note that loading from `<head>` works (replaces the
  head-load demo, which the code already handles via `boot()` on load).
- `README.md`: drop the RC section (panel shipped) and `demo/` reference; fold the
  modal/fullscreen/version-picker steps into the test checklist.
- `CLAUDE.md`: document the Actions Pages deploy + same-origin loading; note
  `docs/index.html` is landing + test surface; never add a second `quibble.js`.

## Manual test checklist (deployed page)

1. `…/quibble/quibble.js` → 200 + `content-type: application/javascript`.
2. Landing loads same-origin by default (URL note shows `quibble.js`).
3. Pick a version → reloads with `?ver=` → loads from jsDelivr.
4. Modal dialog → quibble above modal; clicking quibble chrome doesn't close it.
5. Hero fullscreen → quibble rehomes into the fullscreen element.
6. Text + element comments persist across reload.
