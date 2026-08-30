# Quickshell modules — overview

*English (primary) — [norsk versjon tilgjengelig](quickshell-oversikt-no.md)*

Optional Quickshell modules for Hyprland. None of the installations
touch `hyprland.lua` — you add the required lines yourself, clearly
marked in each individual doc file below.

<div style="display:flex; align-items:center; gap:8px; background:rgba(46,146,219,0.1); border:1px solid #2e92db; border-radius:8px; padding:10px 14px; margin:16px 0; font-family:monospace; flex-wrap:wrap;">
  <code id="qs-install-cmd" style="flex:1; min-width:200px; word-break:break-all; color:#2e92db;">curl -O https://raw.githubusercontent.com/archruud/scripts/main/arch-hyprland/install-quickshell.sh && chmod +x install-quickshell.sh && ./install-quickshell.sh</code>
  <button onclick="navigator.clipboard.writeText(document.getElementById('qs-install-cmd').innerText); this.innerText='Copied!'; setTimeout(() => this.innerText='Copy', 1500);" style="background:#2e92db; color:#1a1c22; border:none; border-radius:6px; padding:6px 14px; font-weight:bold; cursor:pointer; white-space:nowrap;">Copy</button>
</div>

Click "Copy", then paste into your terminal and press Enter.

| Module | Doc | hyprland.lua? |
|---|---|---|
| Status bar | [quickshell-bar.md](quickshell-bar.md) | 1 autostart line |
| App overview | [quickshell-overview.md](quickshell-overview.md) | 1 autostart line + 1 keybind |
| Google Calendar (bar add-on) | [quickshell-kalender.md](quickshell-kalender.md) | 1 windowrule |
| Virtual keyboard, Norwegian (bar add-on) | [wvkbd-norsk.md](wvkbd-norsk.md) | 1 autostart line + 1 keybind + 1 layer rule |
| Creating your own keyboard layout | [wvkbd-eget-layout.md](wvkbd-eget-layout.md) | — (standalone) |

## Installation script

```bash
curl -O https://raw.githubusercontent.com/archruud/scripts/main/arch-hyprland/install-quickshell.sh
chmod +x install-quickshell.sh
./install-quickshell.sh
```

Asks interactively which modules you want. Installs Quickshell itself
+ all dependencies. Idempotent — safe to run multiple times, existing
config is always backed up first.

After running, the script prints out **all** the `hyprland.lua` lines
you need for what you selected, gathered in one place — but see the
individual doc files above for an explanation of *why* and *where in
the file* they go, not just what they are.

## After installation

```bash
hyprctl reload   # or your own reload bind, e.g. SUPER+SHIFT+R
```
