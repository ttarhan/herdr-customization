# Herdr customizations: agent guide

This repo is where we customize the **Herdr app** (a desktop app for terminals
and coding agents) with layouts, mods, themes, keys and settings. The repo is
the source of truth. Herdr reads its interface from `~/.config/herdr-app`, and
`bin/sync` copies our files there.

## Read first

**`~/.config/herdr-app/AGENTS.md`** is the authoritative reference. Herdr
rewrites it on every start to match the installed version, so read it at the
start of any task instead of relying on memory or on this file. It covers the
model (spaces, tabs, panes, agents), layouts, keys, settings, programs, mods,
commands and `hc`.

Other references:

- `types/gpui-kit.d.ts`: every element and the `herdr` module (copied
  from the app by `bin/sync`). The types are the truth, not the website.
- `herdr-app commands`: every app command with its arguments.
- `herdr-app --skill`, `hc --skill`: manuals for driving the window and herdr-core.
- `~/.config/herdr-app/mods/<shipped>/main.js`: Herdr's own mods. They're the
  best examples to copy from (`spaces`, `agents`, `tabs`, `files`, `changes`, ...).

## Layout of this repo

```
config/            mirrors ~/.config/herdr-app, holding only the files we own
  layouts/<name>.json  (+ <name>.svg preview)
  mods/<name>/         main.js, mod.json, assets
  themes/<name>.json
  settings.json        personal settings (zoom, sounds, appearance, ...)
  keys.json            add only when we rebind keys (see below)
  programs.json        add only when needed
bin/sync           push/pull/status/watch between config/ and herdr home
types/             gpui-kit.d.ts from the app (git-ignored, refreshed by sync)
jsconfig.json      editor type-checking for config/mods/**/*.js
```

## Workflow

1. Edit files under `config/`. Don't edit `~/.config/herdr-app` directly for
   anything we own; the repo copy would overwrite it, or `sync` would flag drift.
2. Run `bin/sync` (or keep `bin/sync watch` running). It copies changed files,
   runs `herdr-app check` on every changed mod, then reports `*.error` files
   and the errors listed by `herdr-app status`. Exit code is nonzero on any failure.
3. Look at the result in the running app: `herdr-app status`, and
   `herdr-app shot <scratch>/x.png` to see the window. A passing check doesn't
   prove the mod works: it doesn't lay out or run async code. Never call a mod
   done without a passing check *and* a look in the app.
4. Commit.

`bin/sync status` shows what differs. `bin/sync pull [path]` brings files from
herdr home into `config/`.

## Rules

- **Copy, never symlink.** On Herdr 0.1.13 a symlinked mod folder loads, but
  edits don't hot-reload and errors aren't reported. That's why `bin/sync` copies.
- **Shipped layouts and mods are read-only.** Herdr replaces them on every
  start. `bin/sync` refuses to push to a shipped name (it reads
  `installed.json`). To change one, clone it in the app, adopt the clone, then edit it here:
  ```sh
  herdr-app do layout.clone '{"name":"main","as":"my-main"}'   # also switches to it
  herdr-app do mod.clone '{"name":"files","as":"my-files"}'
  bin/sync pull layouts/my-main.json mods/my-files
  ```
  A cloned layout still points at the shipped mods. Point its `content.mod`
  at your clone to change code.
- **Look-only changes** (theme, colors, fonts, sizes) go in
  `config/settings.json` when the layout leaves them unset. That needs no clone.
- **`keys.json`**: any edit freezes the whole file as ours, and Herdr stops
  adding new default shortcuts to it. Only add `config/keys.json` when we
  actually rebind something. Start from the current
  `~/.config/herdr-app/keys.json` (`bin/sync pull keys.json`). Don't take plain
  ctrl+letters that shells rely on unless the user asks.
- **Settings screen drift.** The app's Settings screen writes `settings.json`
  (and the theme picker changes layouts). `bin/sync` won't overwrite a file
  changed outside the repo; it says so. Then run `bin/sync pull <file>` to keep
  the app's version, or `bin/sync push --force` to keep ours.
- **Never touch Herdr-owned files**: `AGENTS.md`, `*.d.ts`, `frame/`,
  `icons/`, `data/`, `view.json`, `shared.json`, `seen.json`, `installed.json`,
  `window*.json`, `layouts/.state/` (layout clicks).
- **Mod rules** (details in the reference): default-export a class extending
  `View`; never keep a `cx` for later; no `setTimeout`/`setInterval` (use
  `cx.timer`); animate with `.frames` / `.spin` / `.loop` / `.transition`,
  never a timer; use `theme()` colors; `exec`, not `process.run`. Keep
  side effects off `globalThis` at module top level (hot reload re-runs it).
- **Experiments**: if you put a throwaway mod straight into herdr home, remove
  it afterwards, along with `data/<name>/` and its entry in `seen.json`.
  `ctrl-alt-shift-escape` always resets to the default `main` layout if a
  layout or `keys.json` breaks the window.
- **Sharing** a layout or mod (herdr.dev): give it a `RECIPE.md` and a
  `preview.png` in the format the reference describes.

## Git

- Commit `config/`, `bin/`, docs. Never commit `types/`, `.sync-state.json`
  or generated `gpui-kit.d.ts` / `jsconfig.json` inside mod folders.
- Commit messages say what changed for the person ("agents list: group by
  space"). Don't add AI attribution or `Co-Authored-By` trailers.
