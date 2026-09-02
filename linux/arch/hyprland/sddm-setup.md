# 03 — SDDM (Login Screen)

*English (primary) — [norsk versjon tilgjengelig](sddm-setup-no.md)*

## What it is

Installs a custom SDDM theme ("archruud") — a copy of the Elarun theme (login form on the left, which matches your preference) with your own wallpaper swapped in as the background.

## Why the background disappeared

This was **not** caused by a Hyprland or Lua-related change — SDDM runs as its own system user (`sddm`) and starts *before* your Hyprland session even exists. It has no knowledge of `hyprland.lua` at all.

The actual cause: SDDM's greeter can't read into your home directory. `/home/archruud` normally has `700` permissions (only you can enter it), and the `sddm` user isn't you. Any background path pointing into `$HOME` — directly or indirectly — silently fails once something (an update, a permission reset) exposes this. This is a long-standing, well-documented SDDM behavior, not a regression specific to your setup.

The fix: copy the wallpaper into the theme's own folder under `/usr/share/sddm/themes/archruud/images/background.png`, a location every user (including `sddm`) can read.

## Config file locations (confirmed against `sddm.conf(5)`)

| Path | Purpose |
|---|---|
| `/usr/lib/sddm/sddm.conf.d/` | System defaults — **never edit** |
| `/etc/sddm.conf.d/` | Local overrides — **edit here** |
| `/etc/sddm.conf` | Older single-file alternative, still works but less tidy |

This script writes to `/etc/sddm.conf.d/theme.conf`.

## What it does

- Installs `sddm` + Qt6 dependencies if missing
- Copies the `archruud` theme (Elarun-based) to `/usr/share/sddm/themes/archruud`
- Copies your current wallpaper (`~/.config/hypr/wallpapers/ARCHRUUD_2560x1440.png`) into the theme's `images/background.png`
- Sets global read permissions on the theme folder (the actual fix for the permission trap above)
- Writes `/etc/sddm.conf.d/theme.conf` with `Current=archruud`
- Enables `sddm.service`

## Settings it needs to work

- Run `02-awww` first so the wallpaper file exists.
- Whenever you change your desktop wallpaper and want SDDM to match, re-copy it:
  ```bash
  sudo cp ~/.config/hypr/wallpapers/ARCHRUUD_2560x1440.png /usr/share/sddm/themes/archruud/images/background.png
  ```

## Install

```bash
chmod +x install-sddm-theme.sh
./install-sddm-theme.sh
```

## Files in this folder

```
03-sddm/
├── install-sddm-theme.sh
└── theme/archruud/    (Elarun, renamed + re-pointed to your background)
```
