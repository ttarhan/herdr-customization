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
upstream/<ver>/    Herdr's originals of our forks (see "Forks of Herdr's files")
skills/            agent skills (herdr-core); bin/install-skills links them into ~/.claude/skills
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
  start, and `bin/sync` refuses to push to a shipped name (it reads
  `installed.json`). To change one, fork it: see "Forks of Herdr's files".
- **Look-only changes** (theme, colors, fonts, sizes) go in
  `config/settings.json` when the layout leaves them unset. That needs no clone.
- **`keys.json`**: any edit freezes the whole file as ours, and Herdr stops
  adding new default shortcuts to it, so it's a fork too (see "Forks of
  Herdr's files"). Only add `config/keys.json` when we actually rebind
  something. Don't take plain ctrl+letters that shells rely on unless the
  user asks.
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

## Forks of Herdr's files

A **fork** is our copy of something Herdr ships, which Herdr then stops
updating: a cloned layout, a cloned mod, or `keys.json` once we own it. Our
own new layouts, mods and themes aren't forks, and neither is
`settings.json` (Herdr never ships changes to it). Prefer settings over a
fork whenever settings can do it.

### The standard

- **One base version for all forks.** `upstream/<version>/` holds Herdr's
  original of every fork, exactly as that version shipped it, at the same
  paths as in herdr home (`layouts/main.json`, `mods/files/`, `keys.json`).
  There is only ever one version folder. On a Herdr update, every fork
  moves to the new version together.
- **The folder name is the version.** A layout or mod clone's
  `cloned_from.version` must equal it. `bin/sync` (push and status) prints a
  `FORKS` warning when Herdr's version differs from the folder, when an
  original is missing from it, or when a `cloned_from.version` doesn't match.
- **Our changes stay small and listed.** Each fork has a row in the table
  below saying what we changed. A plain `diff` between the fork and its
  original shows only those changes (and `cloned_from`, and `mods/<clone>/`
  asset paths in mods).
- **Commit the fork, its original and its table row together.**

### Our forks

Keep this table current: add a row when you fork something, update "What
we changed" whenever you edit a fork, and remove the row (and its original
in `upstream/`) when you delete one.

| Fork | Original | What we changed |
|---|---|---|
| `layouts/my-main.json` | `layouts/main.json` | Files panel block (`id: files`) uses `my-files` instead of `files`. Nothing else. |
| `mods/my-files/` | `mods/files/` | `main.js`: Markdown files (`md`, `markdown`, `mdown`, `mkd`, `mdx`) render with `TextView.markdown` and a Source/Preview toggle in the header; relative links open in the panel, URLs in the browser; Open ↗ (local, macOS) runs `open -a "Markdown Preview"`. The rest is upstream. |

### Making a fork

Below, `<ver>` is the folder in `upstream/` (it equals `herdr-app --version`
unless a Herdr update is pending; do that first).

- **Layout**:
  ```sh
  herdr-app do layout.clone '{"name":"main","as":"my-main"}'   # also switches to it
  cp ~/.config/herdr-app/layouts/main.json upstream/<ver>/layouts/
  ```
  `layout.clone` reorders the JSON keys. Run `bin/sync pull layouts/my-main.json`
  (so sync tracks it), then overwrite `config/layouts/my-main.json` with the
  original in its own key order:
  `jq --indent 2 '{id} + {cloned_from: {name: "main", version: "<ver>"}} + . | <our edits>' upstream/<ver>/layouts/main.json`.
  Herdr keeps the file as written (clicks go to `layouts/.state/`). A cloned
  layout still points at the shipped mods: to change a mod's code, fork the
  mod too and point the layout's `content.mod` at the fork.
- **Mod**:
  ```sh
  herdr-app do mod.clone '{"name":"files","as":"my-files"}'
  bin/sync pull mods/my-files
  cp -R ~/.config/herdr-app/mods/files upstream/<ver>/mods/
  rm -f upstream/<ver>/mods/files/{jsconfig.json,*.d.ts}
  ```
  The clone's own asset paths say `mods/my-files/` instead of `mods/files/`.
- **`keys.json`**: before the first edit, while Herdr still owns the file:
  `cp ~/.config/herdr-app/keys.json upstream/<ver>/` and
  `bin/sync pull keys.json`. `keys.json` has no `cloned_from`; its version is
  the folder's.

Then add the table row.

### On each Herdr update

`bin/sync` warns when `herdr-app --version` differs from `upstream/<base>/`.
`<new>` is the new version.

1. Save the new originals of every fork in the table: `upstream/<new>/`
   with the same paths (leave out `jsconfig.json` and `*.d.ts` in mods).
   For `keys.json`, Herdr doesn't write new defaults into a file we own.
   How to get the new version's default keys hasn't been worked out yet
   (perhaps a scratch `HERDR_APP_HOME`); work it out the first time and
   write it here.
2. See what Herdr changed: `diff -ru upstream/<base> upstream/<new>`.
3. Apply those changes to each fork in `config/`, keeping what the table
   lists. For a large rewrite, it's simpler to clone fresh and re-apply our
   changes than to merge.
4. Check that only our changes remain:
   `diff config/layouts/<fork>.json upstream/<new>/layouts/<original>.json`,
   `diff <(sed 's|mods/<fork>/|mods/<original>/|g' config/mods/<fork>/main.js) upstream/<new>/mods/<original>/main.js`.
5. Set every `cloned_from.version` to `<new>` and delete `upstream/<base>/`.
6. `bin/sync` (no `FORKS` warnings), then look in the app (`herdr-app
   status`, `herdr-app shot`). Also read the new `~/.config/herdr-app/AGENTS.md`
   for changes that affect our own mods and layouts too (renamed commands,
   `herdr` API changes).
7. Commit: "Herdr <new>: update forks".

## Git

- Commit `config/`, `bin/`, docs. Never commit `types/`, `.sync-state.json`
  or generated `gpui-kit.d.ts` / `jsconfig.json` inside mod folders.
- Commit messages say what changed for the person ("agents list: group by
  space"). Don't add AI attribution or `Co-Authored-By` trailers.
