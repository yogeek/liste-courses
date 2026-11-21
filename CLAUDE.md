# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Mes courses": a French-language shared shopping-list PWA. The whole app is **one static `index.html`** (vanilla JS + CSS, no dependencies, no build step), served by GitHub Pages from `main` at https://yogeek.github.io/liste-courses/. Shared lists live in a Firebase Realtime Database (REST + SSE, no SDK). UI text, comments and commit messages are in French; keep it that way.

Other files: `sw.js` (service worker), `manifest.json` + icons (PWA), `.nojekyll`, `README.md` (user-facing features + the Firebase rules JSON).

## Running / testing

- Serve locally: `python3 -m http.server 8765` then open `http://localhost:8765/`.
- There is no test suite in the repo. Changes are verified with ad-hoc Playwright scripts (iPhone 13 emulation). Conventions that make that work:
  - Firebase is not reachable from the sandbox: serve a copy of `index.html` with `DB_URL` rewritten to a fake host and fulfil `GET`/`PUT /listes/<id>.json` from an in-memory store via `context.route`; answer `Accept: text/event-stream` requests with a keep-alive event. Strip empty arrays/nulls from PUT bodies to mimic Firebase.
  - A fresh profile shows the welcome + guide modals; skip them with an init script setting `localStorage.courses_v2_prefs = {"welcomed":true,"guided":true}`, and populate with `addExample(doc); commit()`.
  - Multi-device scenarios = several browser contexts sharing the mocked store; call `sync()` to force a round-trip instead of waiting for `POLL_MS`.
  - Watch `pageerror` events; all inputs must stay at 16px font (iOS zoom).

## Release checklist

Every deploy to `main` must bump `APP_VERSION` in `index.html` (users get a « 🆕 Nouvelle version » banner by comparing it against a freshly fetched `index.html?v=…`). Bump `CACHE` in `sw.js` when the cached asset list or SW logic changes. Work happens on the session branch and is pushed to both that branch and `main`.

Never put a real person's shopping list in the code (the history was rewritten once to purge one); `EXAMPLE` must stay neutral.

## Architecture (index.html script)

**Data model** — `doc = {cats, items, shops}`, persisted in localStorage (`courses_v2_doc`; prefs in `courses_v2_prefs`, sync id in `courses_v2_sync`, activity feed in `courses_v2_activity`; `courses_v1` is migrated by `docFromV1`). Every element carries `id`, `u` (last-modified timestamp) and optional `del` (tombstone, pruned after `TOMBSTONE_DAYS`). Items also have `t, cat, shop, need` (on the list vs. only in the catalogue), `done, miss` (pas trouvé), `fav` (régulier), `hist[]` (purchase dates, used by `prediction`), `o` (order), `by`/`dev` (author name / device id).

**Mutation rule** — always change elements through `touch(x, patch)` (stamps `u` monotonically plus `by`/`dev`), then `commit()` (save → render → scheduleSync). User actions wrap mutations in `withUndo(msg, fn)`; undo re-applies a snapshot with fresh timestamps so it wins the merge on other devices. Deletions set `del`, never splice.

**Sync** — `sync()` GETs `/listes/<id>.json`, runs `normalize`/`cleanDoc` (Firebase drops empty arrays and may return arrays as objects — always go through `arr()`), merges per element with last-writer-wins on `u` (`mergeArr`), feeds the diff to `recordRemote` (activity feed, ignores own `dev`), and PUTs only if the merged doc differs. Realtime via `EventSource` (`live()`), polling every `POLL_MS` as fallback. Share link format is `#l=<id>` (`hashId`/`setHash`); `createShare()` makes a random 20-char base64url id (Firebase rules require 16–40 chars). Joining a link replaces the local doc with the remote one.

**Classification & dedupe** — `guessKey` maps text to a rayon using `CAT_DEFS` keyword lists, `PHRASES` overrides, and learning from existing items' first word. Duplicate detection (`coreKey` → strips quantities/units/packaging/stopwords/parentheses; `tokEq` with Levenshtein; `compareCore` → same/less/more; `bestMatch`) is shared by the add field and the paste preview (`parsePaste`, `updatePastePreview`, per-line choices in `pasteChoice`).

**UI** — `render()` rebuilds the list from `doc` + `prefs` (current shop filter, search query from the add field, collapsed rayons). Menu/chip buttons dispatch through the `ACTIONS` map via `data-act` attributes; modals are `.modal` elements toggled with `open(id)`/`closeAll()`. `finishCourse()` moves done items back to the catalogue (`need:false`) and keeps missing/unchecked ones on the list.

**Service worker** — network-first with `cache: "no-cache"`, cache fallback offline, and bypasses any URL with a query string (that's what makes the version check reliable).

User feedback (Menu ⋯ → 💬) is written to Firebase `/avis/<id>` (write-only per the rules in README).
