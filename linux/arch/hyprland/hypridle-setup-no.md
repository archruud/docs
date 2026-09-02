# 09 — Hypridle

*[English (primary)](hypridle-setup.md) — norsk versjon*

## Hva det er

Inaktivitets-daemon for Hyprland: dimmer skjermen, skrur den av, låser sesjonen, og suspenderer systemet etter konfigurerbare perioder med inaktivitet.

## Hva det gjør

- Installerer `hypridle` (+ `hyprlock` og `brightnessctl` hvis de mangler)
- Lager `~/.config/hypr/hypridle.conf` med fire trinnvise tidsavbrudd:

| Tidsavbrudd | Handling |
|---|---|
| 5 min | Dimme skjerm til 10% lysstyrke |
| 10 min | Skru av skjerm (DPMS) |
| 15 min | Lås sesjon |
| 30 min | Suspender system |

- Sjekker om `hypridle` allerede er i `hyprland.lua` sin autostart-blokk, og printer linjen hvis ikke

## Innstillinger den trenger for å fungere

- Autostart-linjen skal inn i den eksisterende `hl.on("hyprland.start", function() ... end)`-blokken:
  ```lua
  hl.exec_cmd("hypridle")
  ```
- Avhenger av `hyprlock` (se `hyprlock-setup-no.md`) for selve låseskjermen — hypridle bare trigger den.

## Installasjon

```bash
chmod +x install-hypridle.sh
./install-hypridle.sh
```

## Bug fikset fra originalscriptet

Det gamle scriptet sjekket etter `hyprland.conf` og **avsluttet med feil hvis den ikke fantes** — siden den filen ikke lenger finnes etter Lua-migreringen, ville scriptet ha feilet fullstendig. Sjekker nå `hyprland.lua` i stedet og feiler aldri hardt på dette.
