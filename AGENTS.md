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
upstream/<ver>/    Herdr's originals of what we cloned (see "On each Herdr update")
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
  start. `bin/sync` refuses to push to a shipped name (it reads
  `installed.json`). To change one, clone it in the app, adopt the clone, then edit it here:
  ```sh
  herdr-app do layout.clone '{"name":"main","as":"my-main"}'   # also switches to it
  herdr-app do mod.clone '{"name":"files","as":"my-files"}'
  bin/sync pull layouts/my-main.json mods/my-files
  ```
  A cloned layout still points at the shipped mods. Point its `content.mod`
  at your clone to change code. Every clone goes in the table under
  "On each Herdr update", with its original saved in `upstream/<version>/`.
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

## On each Herdr update

Clones never get Herdr's updates. After Herdr updates (`herdr-app --version`
differs from the newest folder in `upstream/`), carry Herdr's changes into
every clone below.

### Our clones

Keep this table current: add a row when you clone something, change the
"What we changed" column when you edit a clone, and remove the row when you
delete one.

| Clone | From | Base | What we changed |
|---|---|---|---|
| `layouts/my-main.json` | `main` | 0.1.13 | Files panel block (`id: files`) uses `my-files` instead of `files`. Nothing else. |
| `mods/my-files/` | `files` | 0.1.13 | `main.js`: Markdown files (`md`, `markdown`, `mdown`, `mkd`, `mdx`) render with `TextView.markdown` and a Source/Preview toggle in the header; relative links open in the panel, URLs in the browser; Open ↗ (local, macOS) runs `open -a "Markdown Preview"`. The rest is upstream. |

`upstream/<version>/` holds Herdr's original files for each base, as shipped.
They're the only record of what a clone started from, because Herdr
overwrites its own copies on every update. Keep only the versions some clone
still uses as its base.

### Steps

1. Snapshot the new originals of everything in the table:
   `mkdir -p upstream/<new>/{layouts,mods}`, then copy
   `~/.config/herdr-app/layouts/<from>.json` and `mods/<from>/` into it (leave
   out `jsconfig.json` and `*.d.ts`).
2. For each clone, see what Herdr changed:
   - mods: `diff -ru upstream/<base>/mods/<from> upstream/<new>/mods/<from>`
   - layouts (clones reorder keys, so sort them first):
     `diff <(jq -S . upstream/<base>/layouts/<from>.json) <(jq -S . upstream/<new>/layouts/<from>.json)`
3. No changes: just set the Base column to `<new>`. Otherwise apply Herdr's
   changes to the clone in `config/` by hand, keeping what the table lists.
   Mod clones use `mods/<clone>/` in asset paths where the original says
   `mods/<from>/`. For a large rewrite, it's simpler to clone fresh
   (`mod.clone`/`layout.clone` with a new name, pull, re-apply our changes,
   rename) than to merge.
4. Check that only our changes remain:
   `diff <(sed 's|mods/<clone>/|mods/<from>/|g' config/mods/<clone>/main.js) upstream/<new>/mods/<from>/main.js`,
   and `jq -S 'del(.cloned_from)'` on layouts.
5. `bin/sync`, then look in the app (`herdr-app status`, `herdr-app shot`).
   Also read the new `~/.config/herdr-app/AGENTS.md` for changes that affect
   our own mods (renamed commands, `herdr` API changes).
6. Update the Base column, delete `upstream/` folders no longer used as a
   base, and commit ("Herdr <new>: update my-files, my-main").

## Git

- Commit `config/`, `bin/`, docs. Never commit `types/`, `.sync-state.json`
  or generated `gpui-kit.d.ts` / `jsconfig.json` inside mod folders.
- Commit messages say what changed for the person ("agents list: group by
  space"). Don't add AI attribution or `Co-Authored-By` trailers.
