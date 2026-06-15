# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, dependency-free Progressive Web App (PWA) that tracks time, money, and health milestones since the user quit smoking. There is no build step, no package manager, and no test suite — everything is plain HTML/CSS/JS.

## Running

Open `index.html` directly in a browser, or serve the directory over HTTP so the manifest and PWA install flow work:

```
python3 -m http.server 8000   # then visit http://localhost:8000
```

A static server is required (not `file://`) to exercise PWA behavior (`manifest.webmanifest`, install prompt, icons).

## Architecture

Everything lives in `index.html`: markup, an inline `<style>` block (CSS custom properties under `:root` drive the dark theme), and an inline `<script>`.

Core data flow:
- **Settings** (`quitISO`, `perDay`, `packPrice`, `packSize`) are the single source of truth, persisted to `localStorage` under the key `smokefree:settings`. `loadSettings()` merges stored values over `DEFAULTS`; `saveSettings()` writes them back. Both swallow errors so a corrupt/unavailable store degrades to defaults rather than crashing.
- **`render()`** recomputes everything from `settings` + current time on every call: elapsed time, cigarettes avoided (`perDay/86400000 * elapsed`), money saved (`packPrice/packSize` per cigarette), the rotating encouragement line, and the milestone list. It is called once on load and then every second via `setInterval(render, 1000)`. Rendering is fully derived state — there is no mutation of intermediate caches.
- **`MILESTONES`** is a static array of `{ ms, t, d }` (offset from quit time, title, description). `render()` marks each done/in-progress by comparing elapsed time against `ms` and draws a progress bar for pending ones.
- The Settings `<details>` form writes back to `settings` on Save and re-renders; "Reset quit time to now" builds a local-time `datetime-local` string and saves.

`manifest.webmanifest` plus `icon-192.png` / `icon-512.png` make it installable. Theme color `#0f1419` is duplicated in the manifest and the `<meta name="theme-color">` — keep them in sync if changed.

## Conventions

- Keep it a single self-contained file with no external dependencies or build tooling unless there's a clear reason to add them.
- `localStorage` is the only persistence layer; the schema is the `DEFAULTS` object shape. Adding a setting means updating `DEFAULTS`, the form inputs, the Save handler, and `fillForm()` together.
