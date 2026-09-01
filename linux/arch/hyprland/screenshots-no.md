# 14 — Skjermbilder

*[English (primary)](screenshots.md) — norsk versjon*

## Hva det er

Fullt skjermbilde-verktøy for Hyprland: velg område, hele skjermen, eller aktivt vindu — alt sendes rett inn i et tegne-/annoteringsverktøy (piler, tekst, tegning) før lagring eller kopiering til utklippstavlen.

## Hva det gjør

- Installerer `grim`, `slurp`, `swappy`, `jq`, `wl-clipboard`
- Setter opp `~/.config/swappy/config` (lagringssted, filnavn-format)
- Lager `~/.config/hypr/scripts/screenshot-window.sh` (tar bilde av nøyaktig det fokuserte vinduets geometri)
- Oppretter `~/Bilder/Screenshots` som lagringsmappe

## Tastebindinger

| Kombinasjon | Handling |
|---|---|
| `SUPER + SHIFT + S` | Velg område |
| `SUPER + CTRL + S` | Hele skjermen |
| `SUPER + ALT + S` | Kun aktivt vindu |

Etter at bildet er tatt åpner Swappy: **Save** lagrer til `~/Bilder/Screenshots`, **Copy** legger det på utklippstavlen (kan limes rett inn her i chatten, Discord, osv.), og verktøylinjen lar deg tegne/annotere før du gjør noen av delene.

## Innstillinger den trenger for å fungere

- Tastebindingene må ligge i `hyprland.lua` — scriptet sjekker og printer de tre `hl.bind()`-linjene hvis de mangler, skriver ikke lenger til `hyprland.conf` (ubrukt siden Lua-migreringen).
- Lagringsmappa forutsetter den norske XDG-bildemappa (`~/Bilder`) — matcher locale-fiksen fra `base-system-setup`.

## Installasjon

```bash
chmod +x install-screenshot.sh
./install-screenshot.sh
```

## Status

Allerede installert og fungerende på hovedmaskinen din — dette dokumentet + scriptet finnes så en fersk installasjon (som `archmini`) får nøyaktig samme oppsett uten å skrive det inn for hånd på nytt.
