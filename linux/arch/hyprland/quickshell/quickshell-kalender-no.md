# Quickshell: Google Kalender-knapp (valgfritt tillegg til bar)

*Norsk — [English primary version](quickshell-kalender.md)*

Klokke-modulen i [bar](quickshell-bar.md) kan åpne en ekte Google
Kalender i et flytende, sentrert vindu ved klikk — en tynn native
ramme (via `ice-ssb`) rundt selve nettsiden, ikke en egen
API-integrasjon.

**Viktig å vite før du setter dette opp:** uansett hvilken teknisk
løsning som brukes (nettapp-ramme eller en ekte API-integrasjon), må
du logge inn med Google-brukernavn, passord og sikkerhetskode fra
autoriserings-appen din. Det er et Google-krav for enhver app som
skal hente ut kalenderdataene dine, og lar seg ikke omgå. Denne
løsningen unngår i det minste API-nøkkel-vedlikehold på din side.

## ice-ssb, ikke webapp-manager — verdt å vite hvorfor

Begge verktøyene finnes (`ice-ssb` på AUR, `webapp-manager` i
offisielt Arch-repo) og deler samme underliggende mappestruktur
(`~/.local/share/ice/`). Forskjellen som faktisk betyr noe her:

- **`ice-ssb`** bruker **eksakt navnet du skriver inn** som
  profil-ID/klassenavn — skriver du `GoogleKalender`, får du en mappe
  kalt nøyaktig `GoogleKalender`. Ingen tilfeldig suffiks.
- **`webapp-manager`** legger automatisk til 4 tilfeldige siffer på
  slutten av navnet **hver gang** du oppretter appen (f.eks.
  `GoogleKalender3444` én gang, `kalender6845` neste gang ved
  reinstallasjon) — bekreftet gjennom gjentatt testing.

Siden `ice-ssb` gir et **forutsigbart** navn, slipper vi hele steget
med å lete opp en genererte ID etterpå — du bestemmer navnet på
forhånd, og bruker akkurat det samme navnet konsekvent overalt.
Derfor er `ice-ssb` anbefalingen her, ikke `webapp-manager`.

## Fremgangsmåte

Bruk **samme navn** (`GoogleKalender` i eksemplene under — bytt det
til noe annet hvis du vil, men vær konsekvent) i alle tre stegene.

### 1. Installer ice-ssb

```bash
yay -S ice-ssb
```
(eller `paru -S ice-ssb`)

### 2. Skaff et ikon

```bash
rsvg-convert -w 64 -h 64 google-calendar.svg -o ~/.local/share/ice/icons/GoogleKalender.png
```

Ved å legge ikonet i `~/.local/share/ice/icons/` — `ice-ssb` sin egen
ikon-mappe — bør det være rett tilgjengelig i fil-dialogen uten at
du må lete på nettet etter et.

### 3. Opprett nettappen i ice-ssb

Start `ice-ssb`, trykk `+`, fyll inn:

| Felt | Verdi |
|---|---|
| Navn | `GoogleKalender` |
| Adresse | `https://calendar.google.com/calendar/u/0/r?pli=1` |
| Ikon | PNG-en fra steg 2 |
| Nettleser | Firefox |

Trykk OK, lukk `ice-ssb`.

Bekreft at profilen faktisk fikk nøyaktig dette navnet:
```bash
ls ~/.local/share/ice/firefox/
```
Du skal se en mappe kalt `GoogleKalender` — ingen tall lagt til.

### 4. Sett navnet inn i Bar.qml

Åpne `~/.config/quickshell/bar/widgets/Bar.qml`, søk etter
`calendarProc` (rundt linje 243), og sett inn navnet **tre steder**
på samme linje (`--class`, `--name`, og `--profile`-stien) — merk:
**ingen `WebApp-`-prefiks** med `ice-ssb`:

```qml
command: ["sh", "-c", "XAPP_FORCE_GTKWINDOW_ICON=\"$HOME/Nedlastinger/google-calendar64.png\" firefox --class GoogleKalender --name GoogleKalender --profile $HOME/.local/share/ice/firefox/GoogleKalender --no-remote \"http://calendar.google.com\""]
```

### 5. hyprland.lua — windowrule for blå kant

Legg til som en **egen, frittstående** `hl.window_rule({...})`-blokk,
sammen med dine andre windowrules (nmgui, blueman, pavucontrol) —
ikke inni autostart-blokken. Merk igjen: ingen `WebApp-`-prefiks i
`match`-mønsteret:

```lua
hl.window_rule({
    name  = "float-googlekalender",
    match = { class = "^(GoogleKalender)$" },
    float = true,
    size  = "1000 750",
    center = true,
    animation = "slide",
    border_color = "rgba(2e92dbff)",
})
```

## Hopper du over dette

Sett `command: ["true"]` i stedet for hele `sh -c`-kommandoen på
`calendarProc`-linjen i `Bar.qml`. Klokke-klikket gjør da ingenting —
klokkeslett og dato-hover fungerer fortsatt som normalt, ingen
`hyprland.lua`-endring nødvendig siden windowrule-en (steg 5) da
ikke trengs.

## Automatisert versjon

[`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
gjør steg 1, 2 og 4 automatisk, med `GoogleKalender` som
forhåndsbestemt navn — venter kun på at du fullfører steg 3 manuelt
(siden `ice-ssb` ikke har noen kommandolinje-modus), og skriver ut
den ferdige windowrule-teksten for steg 5 på slutten. Se
[hovedoversikten](quickshell-oversikt.md).
