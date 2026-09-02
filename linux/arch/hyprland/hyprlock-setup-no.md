# 10 — Hyprlock

*[English (primary)](hyprlock-setup.md) — norsk versjon*

## Hva det er

Selve låseskjermen: bakgrunnen din (blurret/dimmet), et passordfelt, live klokke og dato, og brukernavnet ditt — trigges automatisk av `09-hypridle`.

## Hva det gjør

- Installerer `hyprlock` hvis den mangler
- Lar deg velge hvilken wallpaper-oppløsning som skal brukes (gjenbruker samme filer `02-awww` allerede har lagt i `~/.config/hypr/wallpapers/` — ingen duplikatkopi)
- Lager `~/.config/hypr/hyprlock.conf`: blurret/dimmet bakgrunn, sentrert passordfelt, stor klokke (JetBrains Mono Bold), dato, og brukernavn-label
- Sjekker om `SUPER + L` allerede er bundet i `hyprland.lua`, og printer linjen hvis ikke

## Innstillinger den trenger for å fungere

- **Kjør `02-awww` først** så wallpaper-fila finnes — ellers faller hyprlock tilbake til en enkel fargebakgrunn inntil du gjør det.
- Keybind hører sammen med de andre bindingene dine:
  ```lua
  hl.bind(mainMod .. " + L", hl.dsp.exec_cmd("hyprlock"))
  ```
- Bruker JetBrains Mono — allerede i `pacman-packages.txt`.

## Installasjon

```bash
chmod +x install-hyprlock.sh
./install-hyprlock.sh
```

## Bugs fikset fra originalscriptet

- Oppløsnings-spørsmålet skrev ut valgene, men leste aldri svaret ditt — valgte alltid alternativ 1 uansett. Bruker nå ekte `read`, og legger til 2560x1440-valget som matcher archminis skjerm.
- Bunter ikke lenger sin egen kopi av wallpaper-filene — `02-awww` eier allerede den jobben; å duplisere dem her risikerte bare at de to gikk ut av synk.
- Avslutningsteksten sa du skulle legge keybinden i `hyprland.conf` — oppdatert til ekte `hyprland.lua`-syntaks, og scriptet sjekker nå automatisk i stedet for å bare printe en statisk påminnelse.
