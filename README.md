# herdr customizations

My layouts, mods, themes and settings for the [Herdr](https://herdr.dev)
desktop app, kept in git.

Herdr redraws itself live from `~/.config/herdr-app`. This repo holds the
originals under `config/` (same structure), and `bin/sync` copies them there.
Copies, not symlinks: Herdr doesn't hot-reload symlinked mods.

## Use

```sh
bin/sync            # push config/ to Herdr, check mods, report errors
bin/sync watch      # same, on every save
bin/sync status     # what differs between config/ and Herdr
bin/sync pull PATH  # adopt a file from Herdr, e.g. a clone or Settings change
```

To change a shipped layout or mod, clone it first. Herdr replaces its own
files on every start:

```sh
herdr-app do layout.clone '{"name":"main","as":"my-main"}'
bin/sync pull layouts/my-main.json
```

If something breaks the window, `ctrl-alt-shift-escape` resets to the
default layout.

## Layout

| Path | What |
|---|---|
| `config/layouts/` | whole-window layouts |
| `config/mods/` | panels (gpui-kit JavaScript) |
| `config/themes/` | color themes |
| `config/settings.json` | zoom, sounds, appearance, fonts |
| `bin/sync` | push / pull / status / watch |
| `AGENTS.md` | how coding agents should work here |

Reference for everything Herdr can do: `~/.config/herdr-app/AGENTS.md`
(rewritten by the app for the installed version).
