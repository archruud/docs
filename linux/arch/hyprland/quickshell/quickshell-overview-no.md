# Quickshell: overview

*Norsk — [English primary version](quickshell-overview.md)*

App-oversikt for Hyprland, basert på
[Shanu-Kumawat/quickshell-overview](https://github.com/Shanu-Kumawat/quickshell-overview).
Toggles med `CTRL+Tab` — viser miniatyrer av alle workspaces og
vinduene i dem.

## Installasjon

Installeres via [`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— se [hovedoversikten](quickshell-oversikt.md) for full kjørebeskrivelse.

Manuelt, uten scriptet:
```bash
git clone https://github.com/Shanu-Kumawat/quickshell-overview ~/.config/quickshell/overview
```

## hyprland.lua

Autostart-linjen, **inni** `hl.on("hyprland.start", function() ... end)`-
blokken sammen med dine andre autostart-linjer:

```lua
hl.on("hyprland.start", function()
    -- ... dine andre autostart-linjer ...
    hl.exec_cmd("qs -c overview")
end)
```

Keybind for å toggle, som en **egen, frittstående** `hl.bind(...)`-linje
et sted blant dine andre keybindings (ikke inni autostart-blokken):

```lua
hl.bind("CTRL + Tab", hl.dsp.exec_cmd("qs ipc -c overview call overview toggle"))
```

## Vise scratchpad/special workspaces

Config-filen setter `showSpecialWorkspaces: true` som standard ved
installasjon via scriptet, slik at vinduer i special workspaces
(scratchpad) er synlige i overview, ikke bare i vanlig
workspace-navigasjon.

Fil: `~/.config/quickshell/overview/config.json`
```json
{
  "overview": {
    "showSpecialWorkspaces": true
  }
}
```

Andre justerbare innstillinger i samme fil: `rows`/`columns` (antall
workspace-ruter vist), `scale` (miniatyr-størrelse),
`emptyWorkspaceWallpaper` (bakgrunn for tomme workspaces).

Tema/farger: `~/.config/quickshell/overview/common/Appearance.qml`

## Testing uten å reloade hele Hyprland

```bash
qs -c overview
qs ipc -c overview call overview toggle
```

## Kjent uavklart feil

Workspace-visning kan bli inkonsistent over tid (fungerer rett etter
reboot/oppstart, kan avvike utover i økten). Under utredning — hvis du
opplever dette, kjør `hyprctl workspaces` og `hyprctl clients` og se
etter `special:`-oppføringer, det er første sjekkepunkt.
