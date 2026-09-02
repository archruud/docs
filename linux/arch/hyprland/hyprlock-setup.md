# 10 — Hyprlock

*English (primary) — [norsk versjon tilgjengelig](hyprlock-setup-no.md)*

## What it is

The actual lock screen: your wallpaper (blurred/dimmed), a password field, live clock and date, and your username — triggered automatically by `09-hypridle`.

## What it does

- Installs `hyprlock` if missing
- Lets you pick which wallpaper resolution to use (reuses the same files `02-awww` already placed in `~/.config/hypr/wallpapers/` — no duplicate copy)
- Creates `~/.config/hypr/hyprlock.conf`: blurred/dimmed background, centered password field, large clock (JetBrains Mono Bold), date, and username label
- Checks whether `SUPER + L` is already bound in `hyprland.lua`, and prints the line if not

## Settings it needs to work

- **Run `02-awww` first** so the wallpaper file exists — otherwise hyprlock falls back to a plain color background until you do.
- Keybind goes with your other binds:
  ```lua
  hl.bind(mainMod .. " + L", hl.dsp.exec_cmd("hyprlock"))
  ```
- Uses JetBrains Mono — already in `pacman-packages.txt`.

## Install

```bash
chmod +x install-hyprlock.sh
./install-hyprlock.sh
```

## Bugs fixed from the original script

- The resolution prompt printed options but never actually read your answer — always silently picked option 1. Now uses a real `read`, and adds the 2560x1440 option to match archmini's monitor.
- No longer bundles its own copy of the wallpaper files — `02-awww` already owns that job; duplicating them here just risked the two going out of sync.
- The closing instructions said to add the keybind to `hyprland.conf` — updated to the real `hyprland.lua` syntax, and the script now checks for it automatically instead of just printing a static reminder.
