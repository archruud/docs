# 02 — AWWW (Bakgrunnsbilde)

*[English (primary)](awww-setup.md) — norsk versjon*

## Hva det er

Wayland wallpaper-daemon (tidligere `swww`, omdøpt til `awww` i oktober 2025 — [kilde](https://codeberg.org/LGFae/awww)). Må installeres **før** `02-quickshell-modules` — overview-widgeten leser samme wallpaper-fil til forhåndsvisningen av tomme arbeidsområder, og feiler hvis den ikke finner den.

## Hva det gjør

- Installerer `awww` fra AUR
- Kopierer wallpaper-filer inn i `~/.config/hypr/wallpapers/`
- Lar deg velge standard-oppløsning (2560x1440 for archmini, eller 1920x1200 / 2560x1600 for andre maskiner)
- Lager `~/.config/hypr/scripts/awww-wallpaper.sh` (starter daemonen, venter på socket, setter bakgrunn med fade)
- Sjekker om autostart-linjen allerede ligger inni `hl.on("hyprland.start", function() ... end)`-blokken i `hyprland.lua`, og printer den hvis ikke

## Innstillinger den trenger for å fungere

- **Oppløsningen må matche skjermen din faktisk.** Overview-modulens `emptyWorkspaceWallpaper` i `quickshell/overview/config.json` peker til et spesifikt filnavn — endrer du oppløsning her, må den stien oppdateres også (se `quickshell-modules-setup-no.md`).
- **Autostart-linjen skal inn i den eksisterende funksjonsblokken**, ikke som en løs toppnivå-linje:
  ```lua
  hl.on("hyprland.start", function()
      hl.exec_cmd("qs -c bar")
      hl.exec_cmd("qs -c overview")
      hl.exec_cmd("wvkbd-norsk -L 320 --hidden")
      hl.exec_cmd("~/.config/hypr/scripts/awww-wallpaper.sh")  -- legg til denne
  end)
  ```

## Installasjon

```bash
chmod +x install-awww.sh
./install-awww.sh
```

## Bug fikset fra originalscriptet

Oppløsnings-spørsmålet skrev ut valgene, men leste aldri svaret ditt — valgte alltid alternativ 1 uansett. Bruker nå ekte `read`.
