# Arch Linux Hyprland – ekstra skjerm og egne Waybar-workspaces

Denne guiden er laget for et JaKooLit/LinuxBeginnings-oppsett med modulbasert Waybar.

## Målet

### Bare laptop

- Laptopskjermen bruker workspace **1–10**.

### Én ekstern skjerm og laptop

- Ekstern skjerm bruker workspace **1–10**.
- Laptopskjermen bruker workspace **11–20**.
- Waybar på hver skjerm viser bare workspacene som tilhører den skjermen.

### Tre eksterne skjermer og laptop

- Ekstern skjerm 1 bruker workspace **1–10**.
- Ekstern skjerm 2 bruker workspace **11–20**.
- Ekstern skjerm 3 bruker workspace **21–30**.
- Laptop bruker workspace **31–40** når den interne skjermen er aktiv.

---

# Viktig om JaKooLit/LinuxBeginnings

Ikke rediger `ModulesWorkspaces` direkte. Den filen kan bli erstattet ved en senere oppdatering.

Denne guiden bruker i stedet:

```text
~/.config/waybar/UserModules
```

Waybar-konfigurasjonen inkluderer `UserModules` etter de andre modulfilene. Derfor kan vi legge inn en egen versjon av:

```text
hyprland/workspaces#rw
```

med samme navn. Den bruker fortsatt JaKooLit-oppsettet og ikonene, men vår brukerdefinerte versjon får prioritet.

---

# 1. Kontroller nødvendige programmer

Kjør:

```bash
command -v hyprctl
command -v waybar
command -v jq
```

Forventet resultat:

```text
/usr/bin/hyprctl
/usr/bin/waybar
/usr/bin/jq
```

Kontroller Hyprland-versjonen:

```bash
hyprctl version
```

---

# 2. Finn skjermnavn og oppløsning

Kjør:

```bash
hyprctl monitors
```

I dette konkrete oppsettet er skjermene:

```text
eDP-1 = intern laptopskjerm
DP-1  = ekstern Lenovo-skjerm
```

Aktuelle innstillinger:

```text
eDP-1: 1920x1200@60, scale 1.20
DP-1:  2560x1440@60, scale 1.00
```

Få en kort oversikt:

```bash
hyprctl monitors -j |
jq -r '.[] | "\(.name) | \(.description) | \(.width)x\(.height) | posisjon \(.x)x\(.y) | scale \(.scale)"'
```

---

# 3. Ta sikkerhetskopi

Kjør:

```bash
cp -a ~/.config/hypr/hyprland.conf \
  ~/.config/hypr/hyprland.conf.backup-$(date +%Y%m%d-%H%M%S)

cp -a ~/.config/waybar/config \
  ~/.config/waybar/config.backup-$(date +%Y%m%d-%H%M%S)

cp -a ~/.config/waybar/UserModules \
  ~/.config/waybar/UserModules.backup-$(date +%Y%m%d-%H%M%S)
```

---

# 4. Lag mapper

Kjør:

```bash
mkdir -p ~/.config/hypr/scripts
mkdir -p ~/.config/hypr/generated
```

---

# 5. Lag skjerm- og workspace-scriptet

Åpne:

```bash
nano ~/.config/hypr/scripts/monitor-workspaces.sh
```

Lim inn:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

HYPR_DIR="${HOME}/.config/hypr"
GENERATED_DIR="${HYPR_DIR}/generated"
WORKSPACE_FILE="${GENERATED_DIR}/workspaces.conf"

mkdir -p "${GENERATED_DIR}"

for command_name in hyprctl jq; do
    if ! command -v "${command_name}" >/dev/null 2>&1; then
        echo "FEIL: ${command_name} ble ikke funnet." >&2
        exit 1
    fi
done

for _ in {1..20}; do
    if hyprctl monitors -j >/dev/null 2>&1; then
        break
    fi
    sleep 0.25
done

MONITORS_JSON="$(hyprctl monitors -j)"

LAPTOP_MONITOR="$(
    jq -r '
        map(select(.name | startswith("eDP")))
        | first
        | .name // empty
    ' <<< "${MONITORS_JSON}"
)"

mapfile -t EXTERNAL_MONITORS < <(
    jq -r '
        map(select((.name | startswith("eDP")) | not))
        | sort_by(.x, .y, .name)
        | .[].name
    ' <<< "${MONITORS_JSON}"
)

write_workspace_range() {
    local monitor="$1"
    local first="$2"
    local last="$3"

    for ((workspace = first; workspace <= last; workspace++)); do
        printf 'workspace = %s, monitor:%s, persistent:true\n' \
            "${workspace}" "${monitor}" >> "${WORKSPACE_FILE}"
    done
}

{
    echo "# Denne filen lages automatisk."
    echo "# Ikke rediger den manuelt."
    echo
} > "${WORKSPACE_FILE}"

EXTERNAL_COUNT="${#EXTERNAL_MONITORS[@]}"

if [[ "${EXTERNAL_COUNT}" -eq 0 ]]; then
    if [[ -z "${LAPTOP_MONITOR}" ]]; then
        echo "FEIL: Fant ingen aktiv laptopskjerm." >&2
        exit 1
    fi

    write_workspace_range "${LAPTOP_MONITOR}" 1 10

elif [[ "${EXTERNAL_COUNT}" -eq 1 ]]; then
    write_workspace_range "${EXTERNAL_MONITORS[0]}" 1 10

    if [[ -n "${LAPTOP_MONITOR}" ]]; then
        write_workspace_range "${LAPTOP_MONITOR}" 11 20
    fi

else
    MAX_EXTERNALS="${EXTERNAL_COUNT}"

    if [[ "${MAX_EXTERNALS}" -gt 3 ]]; then
        MAX_EXTERNALS=3
    fi

    for ((index = 0; index < MAX_EXTERNALS; index++)); do
        first=$((index * 10 + 1))
        last=$((first + 9))
        write_workspace_range "${EXTERNAL_MONITORS[index]}" "${first}" "${last}"
    done

    if [[ -n "${LAPTOP_MONITOR}" ]]; then
        first=$((MAX_EXTERNALS * 10 + 1))
        last=$((first + 9))
        write_workspace_range "${LAPTOP_MONITOR}" "${first}" "${last}"
    fi
fi

hyprctl reload >/dev/null

echo "Skjerm- og workspace-oppsettet er oppdatert."
echo "Laptopskjerm: ${LAPTOP_MONITOR:-ikke aktiv}"
echo "Eksterne skjermer: ${EXTERNAL_MONITORS[*]:-ingen}"
echo "Generert fil: ${WORKSPACE_FILE}"
```

Lagre med:

```text
Ctrl+O
Enter
Ctrl+X
```

Gjør scriptet kjørbart:

```bash
chmod +x ~/.config/hypr/scripts/monitor-workspaces.sh
```

---

# 6. La Hyprland lese den genererte filen

Opprett filen først:

```bash
touch ~/.config/hypr/generated/workspaces.conf
```

Åpne:

```bash
nano ~/.config/hypr/hyprland.conf
```

Legg til:

```conf
source = ~/.config/hypr/generated/workspaces.conf
```

Start scriptet automatisk:

```conf
exec-once = sleep 2 && ~/.config/hypr/scripts/monitor-workspaces.sh
```

Legg også til en hurtigtast for manuell oppdatering:

```conf
bind = $mainMod CTRL, R, exec, ~/.config/hypr/scripts/monitor-workspaces.sh
```

Etter tilkobling eller frakobling av skjerm kan du trykke:

```text
SUPER + CTRL + R
```

---

# 7. Generell monitorregel

I `~/.config/hypr/hyprland.conf` kan denne generelle regelen brukes:

```conf
monitor = , preferred, auto, 1
```

På laptop med 1200p-panel og ønsket skalering kan det være bedre å beholde en egen regel:

```conf
monitor = eDP-1, 1920x1200@60, 0x0, 1.20
```

For Lenovo-skjermen:

```conf
monitor = DP-1, 2560x1440@60, 1600x0, 1
```

`1600x0` er logisk riktig fordi:

```text
1920 / 1.20 = 1600
```

Dermed starter den eksterne skjermen rett etter laptopskjermens logiske bredde.

---

# 8. Kjør scriptet

Kjør:

```bash
~/.config/hypr/scripts/monitor-workspaces.sh
```

Kontroller:

```bash
cat ~/.config/hypr/generated/workspaces.conf
```

Med én ekstern skjerm skal filen inneholde:

```conf
workspace = 1, monitor:DP-1, persistent:true
workspace = 2, monitor:DP-1, persistent:true
workspace = 3, monitor:DP-1, persistent:true
workspace = 4, monitor:DP-1, persistent:true
workspace = 5, monitor:DP-1, persistent:true
workspace = 6, monitor:DP-1, persistent:true
workspace = 7, monitor:DP-1, persistent:true
workspace = 8, monitor:DP-1, persistent:true
workspace = 9, monitor:DP-1, persistent:true
workspace = 10, monitor:DP-1, persistent:true

workspace = 11, monitor:eDP-1, persistent:true
workspace = 12, monitor:eDP-1, persistent:true
workspace = 13, monitor:eDP-1, persistent:true
workspace = 14, monitor:eDP-1, persistent:true
workspace = 15, monitor:eDP-1, persistent:true
workspace = 16, monitor:eDP-1, persistent:true
workspace = 17, monitor:eDP-1, persistent:true
workspace = 18, monitor:eDP-1, persistent:true
workspace = 19, monitor:eDP-1, persistent:true
workspace = 20, monitor:eDP-1, persistent:true
```

---

# 9. Rette JSON-feilen i Waybar `config`

Feilen:

```text
Error parsing JSON: * Line 27, Column 5
Syntax error: value, object or array expected
```

peker på hovedfilen:

```text
~/.config/waybar/config
```

Vis området rundt feilen:

```bash
nl -ba ~/.config/waybar/config | sed -n '20,34p'
```

Linje 27 skal være nøyaktig:

```jsonc
    "hyprland/workspaces#rw",
```

Hele delen skal se slik ut:

```jsonc
"modules-left": [
    "custom/menu#user",
    "hyprland/workspaces#rw",

    "hyprland/window",
],
```

Ikke lim inn selve modulobjektet inne i `modules-left`. Denne listen skal bare inneholde modulnavn som tekststrenger.

Åpne filen:

```bash
nano ~/.config/waybar/config
```

Rett linjen, lagre og kontroller på nytt:

```bash
waybar
```

Avslutt testen med `Ctrl+C` når Waybar starter uten JSON-feil.

---

# 10. Bruk egen `workspaces#rw` i `UserModules`

Vi beholder navnet:

```text
hyprland/workspaces#rw
```

Dermed trenger vi ikke endre hovedfilen `config`.

Kopier den ferdige brukerfilen fra denne dokumentasjonen, eller opprett den ved å kopiere den eksisterende modulen fra:

```text
~/.config/waybar/ModulesWorkspaces
```

til:

```text
~/.config/waybar/UserModules
```

I kopien skal bare denne innstillingen endres:

```jsonc
"all-outputs": true,
```

til:

```jsonc
"all-outputs": false,
```

Dette gjør at hver Waybar bare viser workspaces som tilhører skjermen den står på.

## Viktig

Ikke endre originalen:

```text
~/.config/waybar/ModulesWorkspaces
```

En oppdatering kan overskrive den.

Endre bare:

```text
~/.config/waybar/UserModules
```

---

# 11. Installer den ferdige `UserModules`-filen

Etter at du har lastet ned filen `UserModules-Archruud-MultiMonitor`, kopierer du den slik:

```bash
cp ~/.config/waybar/UserModules \
   ~/.config/waybar/UserModules.backup-$(date +%Y%m%d-%H%M%S)
```

Deretter:

```bash
cp ~/Nedlastinger/UserModules-Archruud-MultiMonitor \
   ~/.config/waybar/UserModules
```

Tilpass nedlastingsstien dersom filen ligger et annet sted.

Kontroller starten og slutten:

```bash
head -n 15 ~/.config/waybar/UserModules
tail -n 30 ~/.config/waybar/UserModules
```

---

# 12. Test Waybar-konfigurasjonen

Kjør Waybar i terminalen:

```bash
waybar -c ~/.config/waybar/config -s ~/.config/waybar/style.css
```

Dersom det ikke kommer JSON-feil, avslutt testen med:

```text
Ctrl+C
```

Start Waybar normalt:

```bash
pkill waybar
sleep 1
waybar >/tmp/waybar.log 2>&1 & disown
```

Kontroller prosessen:

```bash
pgrep -a waybar
```

Kontroller feil:

```bash
cat /tmp/waybar.log
```

---

# 13. Kontroller forventet resultat

På ekstern Lenovo-skjerm:

```text
1 2 3 4 5 6 7 8 9 10
```

På laptopskjermen:

```text
11 12 13 14 15 16 17 18 19 20
```

Kontroller workspace-fordelingen:

```bash
hyprctl workspaces -j |
jq -r '.[] | "Workspace \(.id) -> \(.monitor)"' |
sort -V
```

---

# 14. Flytt eksisterende workspaces ved behov

Workspace-regler flytter ikke alltid allerede åpne workspaces umiddelbart.

Kjør derfor én gang:

```bash
for i in {1..10}; do
    hyprctl dispatch moveworkspacetomonitor "$i" DP-1
done

for i in {11..20}; do
    hyprctl dispatch moveworkspacetomonitor "$i" eDP-1
done
```

Velg workspace 1 på Lenovo:

```bash
hyprctl dispatch focusmonitor DP-1
hyprctl dispatch workspace 1
```

Velg workspace 11 på laptop:

```bash
hyprctl dispatch focusmonitor eDP-1
hyprctl dispatch workspace 11
```

Sett fokus tilbake på Lenovo:

```bash
hyprctl dispatch focusmonitor DP-1
```

---

# 15. Hente de nyeste LinuxBeginnings-filene uten å installere dem

Du trenger ikke installere hele oppsettet bare for å undersøke filene.

Installer Git dersom nødvendig:

```bash
sudo pacman -S --needed git
```

Klon dotfiles til en egen arbeidsmappe:

```bash
git clone --depth=1 \
  https://github.com/LinuxBeginnings/Hyprland-Dots.git \
  ~/LinuxBeginnings-Hyprland-Dots
```

Waybar-filene ligger under:

```text
~/LinuxBeginnings-Hyprland-Dots/config/waybar/
```

Vis dem:

```bash
find ~/LinuxBeginnings-Hyprland-Dots/config/waybar \
  -maxdepth 2 -type f -print | sort
```

Sammenlign med ditt aktive oppsett:

```bash
diff -ru \
  ~/.config/waybar \
  ~/LinuxBeginnings-Hyprland-Dots/config/waybar |
less
```

Ikke kopier hele mappen direkte over ditt aktive oppsett før forskjellene er kontrollert.

Oppdater arbeidskopien senere:

```bash
git -C ~/LinuxBeginnings-Hyprland-Dots pull --ff-only
```

JaKooLit sitt eldre dotfile-arkiv kan undersøkes separat:

```bash
git clone --depth=1 \
  https://github.com/JaKooLit/Hyprland-Dots.git \
  ~/JaKooLit-Hyprland-Dots
```

---

# 16. Feilsøking

## JSON-feil på en bestemt linje

Vis linjene med nummer:

```bash
nl -ba ~/.config/waybar/config | sed -n '1,60p'
```

For `UserModules`:

```bash
nl -ba ~/.config/waybar/UserModules | tail -n 100
```

Typiske feil:

- manglende `"`;
- manglende komma;
- ekstra komma på feil sted;
- modulobjekt limt inn i `modules-left`;
- én avsluttende `}` for mye eller for lite.

## Waybar viser 1–20 på begge skjermer

Kontroller at brukerdefinisjonen har:

```jsonc
"all-outputs": false,
```

Finn alle definisjonene:

```bash
grep -Rni -A8 -B2 \
  '"hyprland/workspaces#rw"' \
  ~/.config/waybar
```

`UserModules` må være inkludert etter `ModulesWorkspaces` i hovedkonfigurasjonen.

## Waybar vises riktig, men ny manuell start feiler

Da kjører en gammel Waybar-prosess fortsatt med den tidligere, gyldige konfigurasjonen.

Rett JSON-feilen først og kjør deretter:

```bash
pkill waybar
waybar
```

## Vis Hyprland-konfigurasjonsfeil

```bash
hyprctl configerrors
```

---

# 17. Filer som er endret

| Fil | Formål |
|---|---|
| `~/.config/hypr/hyprland.conf` | Leser genererte workspace-regler og starter scriptet |
| `~/.config/hypr/scripts/monitor-workspaces.sh` | Oppdager skjermer og fordeler workspaces |
| `~/.config/hypr/generated/workspaces.conf` | Automatisk genererte workspace-regler |
| `~/.config/waybar/config` | Inneholder modulrekkefølgen; beholder `hyprland/workspaces#rw` |
| `~/.config/waybar/UserModules` | Inneholder din oppdateringssikre kopi av `workspaces#rw` |
| `~/.config/waybar/ModulesWorkspaces` | Skal ikke endres |

---

# 18. Oppsummering

Den oppdateringssikre løsningen er:

1. Hyprland-scriptet oppdager aktive skjermer.
2. Scriptet binder riktige workspace-numre til hver skjerm.
3. Hovedfilen til Waybar beholder modulnavnet `hyprland/workspaces#rw`.
4. Din egen versjon av modulen ligger i `UserModules`.
5. `"all-outputs": false` gjør at hver skjerm bare viser sine egne workspaces.
6. Originalfilene fra JaKooLit/LinuxBeginnings blir ikke endret.
