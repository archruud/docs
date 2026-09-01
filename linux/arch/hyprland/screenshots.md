# 14 — Screenshots

*English (primary) — [norsk versjon tilgjengelig](screenshots-no.md)*

## What it is

Full screenshot tooling for Hyprland: region select, full screen, or active window — all pipe straight into an annotation tool (draw, arrows, text) before saving or copying to clipboard.

## What it does

- Installs `grim`, `slurp`, `swappy`, `jq`, `wl-clipboard`
- Sets up `~/.config/swappy/config` (save location, filename format)
- Creates `~/.config/hypr/scripts/screenshot-window.sh` (captures only the focused window's exact geometry)
- Creates `~/Bilder/Screenshots` as the save folder

## Keybinds

| Bind | Action |
|---|---|
| `SUPER + SHIFT + S` | Select a region |
| `SUPER + CTRL + S` | Full screen |
| `SUPER + ALT + S` | Active window only |

After capture, Swappy opens: **Save** writes to `~/Bilder/Screenshots`, **Copy** puts it on the clipboard (paste directly into Claude, Discord, etc.), and the toolbar lets you draw/annotate before either.

## Settings it needs to work

- Keybinds must be present in `hyprland.lua` — the script checks and prints the three `hl.bind()` lines if missing, it no longer writes to `hyprland.conf` (unused since the Lua migration).
- Save folder assumes the Norwegian XDG picture folder (`~/Bilder`) — matches the `base-system-setup` locale fix.

## Install

```bash
chmod +x install-screenshot.sh
./install-screenshot.sh
```

## Status

Already installed and working on your main machine — this doc + script exists so a fresh install (like `archmini`) gets the exact same setup without re-typing it by hand.
