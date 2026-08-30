# Quickshell: bar

*Norsk — [English primary version](quickshell-bar.md)*

Statusbaren for Hyprland, bygget direkte mot Quickshell sitt eget API.
Ni innebygde funksjoner: app-launcher, 10 workspaces, aktivt vindu,
klokke med dato-hover, wifi/kabel med IP-hover, bluetooth, mikrofon,
volum (scroll + høyreklikk for pavucontrol), og strøm-meny.

To valgfrie tillegg, se egne dokumenter:
- [Virtuelt tastatur (wvkbd-norsk)](wvkbd-norsk.md)
- [Google Kalender-knapp](quickshell-kalender.md)

## Installasjon

Installeres via [`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— se [hovedoversikten](quickshell-oversikt.md) for full kjørebeskrivelse.

Manuelt, uten scriptet: legg `shell.qml`, `widgets/Bar.qml`,
`widgets/Pill.qml` og `widgets/Theme.qml` i
`~/.config/quickshell/bar/` (filene finnes i
[archruud/scripts](https://github.com/archruud/scripts) sitt
quickshell-underlag).

## hyprland.lua

Legg denne linja **inni** `hl.on("hyprland.start", function() ... end)`-
blokken, sammen med dine andre `hl.exec_cmd()`-autostart-linjer:

```lua
hl.on("hyprland.start", function()
    -- ... dine andre autostart-linjer ...
    hl.exec_cmd("qs -c bar")
end)
```

Ingen keybind nødvendig for selve baren — den er alltid synlig, ikke
noe å toggle.

## Testing uten å reloade hele Hyprland

```bash
qs -c bar
```

## Fjerne/deaktivere

```bash
rm -rf ~/.cache/quickshell && qs -c bar
```
kjøres etter enhver endring i filene, for å tvinge frem full
gjeninnlasting fra bunnen (Quickshell mellomlagrer kompilerte
QML-filer).
