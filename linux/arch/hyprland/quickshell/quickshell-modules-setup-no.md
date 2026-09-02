# 02 — Quickshell-moduler oppsett

*[English (primary)](quickshell-modules-setup.md) — norsk versjon*

## Hva det er

Installerer selve Quickshell, pluss bar- og overview-modulene, pluss Ice SSB (brukes til å kjøre Google Kalender som en ordentlig app i stedet for en nettleserfane). Dette er **kun installasjonen** — for hvordan bar/overview/kalender faktisk fungerer i det daglige, se de eksisterende dokumentene per modul:

- [Quickshell oversikt](quickshell-oversikt-no.md)
- [Bar](quickshell-bar-no.md)
- [Google Kalender](quickshell-kalender-no.md)
- [Norsk virtuelt tastatur](wvkbd-norsk-no.md) *(egen installasjon, dekkes ikke her)*

## Hva det gjør

- Installerer `quickshell-git` og `ice-ssb` fra AUR
- Kopierer bar- og overview-QML-modulene til `~/.config/quickshell/`
- Setter opp Ice sin mappestruktur (`~/.local/share/ice/{icons,firefox,profiles}`)
- Kopierer kalender-ikonet inn i `~/.local/share/ice/icons/`
- Oppretter Google Kalender-profilen (`Kallender3092`) med et rent vindu uten adressefelt/faner
- Lager en `.desktop`-snarvei så kalenderen også dukker opp i app-menyer

## Innstillinger den trenger for å fungere

- **Monitor må allerede være konfigurert i `hyprland.lua`** (se `base-system-setup-no.md`) — bar/overview tegnes per monitor.
- **Autostart-linjer legges ikke inn automatisk** — legg til disse selv, én gang, i `hyprland.lua`:
  ```lua
  hl.exec_cmd("qs -c bar")
  hl.exec_cmd("qs -c overview")
  ```
- **Kalender-ikon**: legg `Kallender3092.png` i `icons/`-mappen ved siden av scriptet før du kjører, eller kopier den inn manuelt etterpå til `~/.local/share/ice/icons/Kallender3092.png`.

## Installasjon

```bash
chmod +x install-quickshell-modules.sh
./install-quickshell-modules.sh
```

## Filer i denne mappen

```
02-quickshell-modules/
├── install-quickshell-modules.sh
├── quickshell/
│   ├── bar/          (shell.qml, widgets/Bar.qml, Theme.qml, Pill.qml)
│   └── overview/      (hele modulen - se quickshell-oversikt-no.md)
└── icons/             (legg Kallender3092.png her før du kjører)
```
