# 02 — Quickshell Modules Setup

*English (primary) — [norsk versjon tilgjengelig](quickshell-modules-setup-no.md)*

## What it is

Installs Quickshell itself, plus the bar and overview modules, plus Ice SSB (used to run Google Calendar as a proper app instead of a browser tab). This is the **installer only** — for how the bar/overview/calendar actually work day-to-day, see the existing per-module docs:

- [Quickshell overview](quickshell-oversikt.md)
- [Bar](quickshell-bar.md)
- [Google Calendar](quickshell-kalender.md)
- [Norwegian virtual keyboard](wvkbd-norsk.md) *(separate install, not covered here)*

## What it does

- Installs `quickshell-git` and `ice-ssb` from AUR
- Copies the bar and overview QML modules to `~/.config/quickshell/`
- Sets up Ice's folder structure (`~/.local/share/ice/{icons,firefox,profiles}`)
- Copies the calendar icon into `~/.local/share/ice/icons/`
- Creates the Google Calendar Firefox profile (`Kallender3092`) with a clean, chrome-less window
- Creates a `.desktop` launcher so the calendar also shows up in app menus

## Settings it needs to work

- **Monitor must already be configured in `hyprland.lua`** (see `base-system-setup.md`) — the bar/overview render per-monitor.
- **Autostart lines are not added automatically** — add these yourself once, in your `hyprland.lua`:
  ```lua
  hl.exec_cmd("qs -c bar")
  hl.exec_cmd("qs -c overview")
  ```
- **Calendar icon**: drop `Kallender3092.png` into the `icons/` folder next to the script before running, or copy it manually afterwards to `~/.local/share/ice/icons/Kallender3092.png`.

## Install

```bash
chmod +x install-quickshell-modules.sh
./install-quickshell-modules.sh
```

## Files in this folder

```
02-quickshell-modules/
├── install-quickshell-modules.sh
├── quickshell/
│   ├── bar/          (shell.qml, widgets/Bar.qml, Theme.qml, Pill.qml)
│   └── overview/      (full module - see quickshell-oversikt.md)
└── icons/             (drop Kallender3092.png here before running)
```
