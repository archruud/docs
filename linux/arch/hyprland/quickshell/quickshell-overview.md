# Quickshell: overview

*English (primary) — [norsk versjon tilgjengelig](quickshell-overview-no.md)*

App overview for Hyprland, based on
[Shanu-Kumawat/quickshell-overview](https://github.com/Shanu-Kumawat/quickshell-overview).
Toggles with `CTRL+Tab` — shows thumbnails of all workspaces and the
windows in them.

## Installation

Installed via [`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— see the [overview](quickshell-oversikt.md) for the full run-through.

Manually, without the script:
```bash
git clone https://github.com/Shanu-Kumawat/quickshell-overview ~/.config/quickshell/overview
```

## hyprland.lua

The autostart line, **inside** the `hl.on("hyprland.start", function() ... end)`
block alongside your other autostart lines:

```lua
hl.on("hyprland.start", function()
    -- ... your other autostart lines ...
    hl.exec_cmd("qs -c overview")
end)
```

Keybind to toggle, as a **separate, standalone** `hl.bind(...)` line
somewhere among your other keybindings (not inside the autostart
block):

```lua
hl.bind("CTRL + Tab", hl.dsp.exec_cmd("qs ipc -c overview call overview toggle"))
```

## Showing scratchpad/special workspaces

The config file sets `showSpecialWorkspaces: true` by default when
installed via the script, so windows on special workspaces
(scratchpad) are visible in the overview, not just in regular
workspace navigation.

File: `~/.config/quickshell/overview/config.json`
```json
{
  "overview": {
    "showSpecialWorkspaces": true
  }
}
```

Other adjustable settings in the same file: `rows`/`columns` (number
of workspace tiles shown), `scale` (thumbnail size),
`emptyWorkspaceWallpaper` (background for empty workspaces).

Theme/colors: `~/.config/quickshell/overview/common/Appearance.qml`

## Testing without reloading all of Hyprland

```bash
qs -c overview
qs ipc -c overview call overview toggle
```

## Known unresolved issue

Workspace display can become inconsistent over time (works right
after reboot/startup, may drift as the session goes on). Under
investigation — if you experience this, run `hyprctl workspaces` and
`hyprctl clients` and look for `special:` entries, that's the first
thing to check.
