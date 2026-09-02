# 02 — AWWW (Wallpaper)

*English (primary) — [norsk versjon tilgjengelig](awww-setup-no.md)*

## What it is

Wayland wallpaper daemon (formerly `swww`, renamed to `awww` in October 2025 — [source](https://codeberg.org/LGFae/awww)). Must be installed **before** `02-quickshell-modules` — the overview widget reads the same wallpaper file for its empty-workspace preview, and errors if it can't find it.

## What it does

- Installs `awww` from AUR
- Copies wallpaper files into `~/.config/hypr/wallpapers/`
- Lets you pick a default resolution (2560x1440 for archmini, or 1920x1200 / 2560x1600 for other machines)
- Creates `~/.config/hypr/scripts/awww-wallpaper.sh` (starts the daemon, waits for its socket, applies the wallpaper with a fade)
- Checks whether the autostart line is already inside your `hl.on("hyprland.start", function() ... end)` block in `hyprland.lua`, and prints it if not

## Settings it needs to work

- **Resolution must match your actual monitor.** The overview module's `emptyWorkspaceWallpaper` in `quickshell/overview/config.json` points to a specific filename — if you change resolution here, update that path too (see `quickshell-modules-setup.md`).
- **Autostart line goes inside the existing function block**, not as a loose top-level line:
  ```lua
  hl.on("hyprland.start", function()
      hl.exec_cmd("qs -c bar")
      hl.exec_cmd("qs -c overview")
      hl.exec_cmd("wvkbd-norsk -L 320 --hidden")
      hl.exec_cmd("~/.config/hypr/scripts/awww-wallpaper.sh")  -- add this line
  end)
  ```

## Install

```bash
chmod +x install-awww.sh
./install-awww.sh
```

## Bug fixed from the original script

The resolution prompt used to print options but never actually read your answer — it silently always picked option 1. Now uses a real `read`.
