# 09 — Hypridle

*English (primary) — [norsk versjon tilgjengelig](hypridle-setup-no.md)*

## What it is

Idle daemon for Hyprland: dims the screen, turns it off, locks the session, and suspends the system after configurable periods of inactivity.

## What it does

- Installs `hypridle` (+ `hyprlock` and `brightnessctl` if missing)
- Creates `~/.config/hypr/hypridle.conf` with four staged timeouts:

| Timeout | Action |
|---|---|
| 5 min | Dim screen to 10% brightness |
| 10 min | Turn screen off (DPMS) |
| 15 min | Lock session |
| 30 min | Suspend system |

- Checks whether `hypridle` is already in your `hyprland.lua` autostart block, and prints the line if not

## Settings it needs to work

- Autostart line goes inside the existing `hl.on("hyprland.start", function() ... end)` block:
  ```lua
  hl.exec_cmd("hypridle")
  ```
- Depends on `hyprlock` (see `hyprlock-setup.md`) for the actual lock screen — hypridle only triggers it.

## Install

```bash
chmod +x install-hypridle.sh
./install-hypridle.sh
```

## Bug fixed from the original script

The old script checked for `hyprland.conf` and **exited with an error if it wasn't found** — since that file no longer exists post-Lua-migration, the script would have failed outright. Now checks `hyprland.lua` instead and never hard-fails on this.
