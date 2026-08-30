# Quickshell: bar

*English (primary) — [norsk versjon tilgjengelig](quickshell-bar-no.md)*

The status bar for Hyprland, built directly against Quickshell's own
API. Nine built-in features: app launcher, 10 workspaces, active
window, clock with date-hover, wifi/ethernet with IP-hover,
bluetooth, microphone, volume (scroll + right-click for pavucontrol),
and a power menu.

Two optional add-ons, see their own docs:
- [Virtual keyboard (wvkbd-norsk)](wvkbd-norsk.md)
- [Google Calendar button](quickshell-kalender.md)

## Installation

Installed via [`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— see the [overview](quickshell-oversikt.md) for the full run-through.

Manually, without the script: place `shell.qml`, `widgets/Bar.qml`,
`widgets/Pill.qml`, and `widgets/Theme.qml` in
`~/.config/quickshell/bar/` (the files live in
[archruud/scripts](https://github.com/archruud/scripts)'
quickshell folder).

## hyprland.lua

Add this line **inside** the `hl.on("hyprland.start", function() ... end)`
block, alongside your other `hl.exec_cmd()` autostart lines:

```lua
hl.on("hyprland.start", function()
    -- ... your other autostart lines ...
    hl.exec_cmd("qs -c bar")
end)
```

No keybind needed for the bar itself — it's always visible, nothing
to toggle.

## Testing without reloading all of Hyprland

```bash
qs -c bar
```

## Removing/disabling

```bash
rm -rf ~/.cache/quickshell && qs -c bar
```
Run this after any change to the files, to force a full reload from
scratch (Quickshell caches compiled QML files).
