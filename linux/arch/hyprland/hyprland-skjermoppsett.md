# Hyprland: Dual-skjerm + Clamshell (Dell laptop + Lenovo Legion 27Q-10)

Oppsett for å kjøre Dell-laptopen med lokk igjen, kun ekstern
skjerm + tastatur/mus i bruk, mens du fortsatt kan bruke
laptop-panelet når lokket er åpent.

**Gjelder:** Hyprland ≥ 0.55 (Lua-config, `hyprland.lua`). Den
gamle `.conf`/hyprlang-syntaksen er deprecated fra og med 0.55 —
denne guiden bruker kun Lua-API-et (`hl.monitor`, `hl.bind`).

**Testet og bekreftet fungerende:** 27.08.2026. Med lokk åpent
kjører begge skjermer samtidig. Med lokk lukket er `eDP-1`
deaktivert og kun `HDMI-A-1` er aktiv — ingen manuell handling
nødvendig.

---

## Maskinvare

| Output     | Panel                          | Native   | Kjørt mode      | Scale |
|------------|---------------------------------|----------|-----------------|-------|
| `eDP-1`    | Dell-laptop (BOE 0x0CC6)        | 1920x1200| 1920x1200@60    | 1     |
| `HDMI-A-1` | Lenovo Legion 27Q-10             | 2560x1440| 1920x1080@60    | 1     |

Legion-skjermen kjøres ned fra native 1440p til 1080p — dette er en
reell modus fra `availableModes`, ikke skalert/interpolert av
Hyprland.

---

## 1. Finn dine egne skjermnavn først

Portnavn (`eDP-1`, `HDMI-A-1` osv.) er ikke garantert like på annen
maskinvare eller etter en kabel-/dock-endring. Kjør alltid dette
før du limer inn config på en ny maskin:

```bash
hyprctl monitors        # kun aktive skjermer
hyprctl monitors all    # inkl. frakoblede/kjente
```

Se etter `description:`-feltet for å identifisere hvilken fysisk
skjerm som er hvilken (f.eks. `BOE 0x0CC6` = det interne panelet,
`Lenovo Group Limited Legion 27Q-10` = ekstern).

---

## 2. Delene som går inn i `hyprland.lua`

Alt under limes inn i `------ MONITORS ------`-seksjonen (eller
tilsvarende) i `hyprland.lua`. Rekkefølgen mellom de to blokkene
under spiller ingen rolle, men **hele MONITORS-seksjonen må stå før
LOKK/CLAMSHELL-seksjonen** i punkt 3 — se forklaring der.

### 2.1 Laptop-panelet

```lua
-- Dell-panelet (internt)
hl.monitor({
    output   = "eDP-1",
    mode     = "1920x1200@60",
    position = "0x0",
    scale    = "1",
})
```

**Hvorfor:**
- `mode = "1920x1200@60"` — panelets native oppløsning og
  oppdateringsfrekvens. Uten eksplisitt mode velger Hyprland en
  "beste gjetning" som ikke alltid stemmer.
- `position = "0x0"` — plasserer laptop-panelet i det virtuelle
  skjermlandskapets øvre venstre hjørne. Alt annet regnes relativt
  til dette.
- `scale = "1"` — ingen skalering. Valgt eksplisitt fremfor
  automatisk fraksjonell skalering (f.eks. 1.5), som ellers er
  vanlig på en 13.3"-flate med denne oppløsningen. Konsekvens: UI
  og tekst blir fysisk mindre på laptop-panelet enn med skalering,
  men holder pikslene 1:1 og unngår at posisjonsberegningen for
  nabo-skjermen må regnes om til logiske piksler.

### 2.2 Ekstern skjerm

```lua
-- Ekstern skjerm (Lenovo Legion 27Q-10)
hl.monitor({
    output   = "HDMI-A-1",
    mode     = "1920x1080@60",
    position = "1920x0",
    scale    = "1",
})
```

**Hvorfor:**
- `mode = "1920x1080@60"` — kjørt lavere enn skjermens native
  2560x1440 per ønsket oppsett. Denne modusen finnes reelt i
  skjermens `availableModes`.
- `position = "1920x0"` — legges rett til høyre for laptop-panelet.
  Siden begge kjører `scale = 1`, er logisk og fysisk bredde det
  samme (1920px), så ingen omregning trengs. (Hadde en av skjermene
  kjørt fraksjonell skalering, måtte denne verdien vært regnet i
  *logiske* piksler, ikke fysiske — verdt å huske ved fremtidige
  endringer.)
- `scale = "1"` — samme begrunnelse som over, valgt eksplisitt.

> **Vertikal justering:** de to skjermene er ulik høyde (1200 vs.
> 1080px), så de er implisitt toppjustert her (`y=0` på begge).
> I praksis er dette irrelevant for dette oppsettet, siden
> laptop-panelet uansett deaktiveres når lokket lukkes (se punkt 3)
> — det er bare i det korte vinduet der lokket står åpent og
> begge skjermer er aktive at justeringen har noe å si for hvor
> musepekeren "møter" nabo-skjermen.

### 2.3 Clamshell-logikk (lokk lukket → kun ekstern skjerm)

```lua
local function onLidClose()
    hl.monitor({ output = "eDP-1", disabled = true })
end

local function onLidOpen()
    hl.monitor({
        output   = "eDP-1",
        mode     = "1920x1200@60",
        position = "0x0",
        scale    = "1",
        disabled = false,
    })
end

hl.bind("switch:on:Lid Switch",  onLidClose, { locked = true })
hl.bind("switch:off:Lid Switch", onLidOpen,  { locked = true })
```

**Hvorfor:**
- `onLidClose` / `onLidOpen` — to funksjoner som hhv. deaktiverer
  og reaktiverer `eDP-1` med full config (mode/posisjon/scale må
  gjentas ved reaktivering, ikke bare `disabled = false`).
- `hl.bind("switch:on:Lid Switch", ...)` — binder laptopens
  lokk-sensor (en switch-enhet i kernel/libinput) til funksjonen.
  `switch:on` = lokk lukkes, `switch:off` = lokk åpnes.
- `{ locked = true }` — gjør at bindingen fungerer selv når
  skjermen er låst (f.eks. via hyprlock). Uten denne vil ikke
  lokk-hendelsen trigge config-endringen om du har låst skjermen
  før du lukker lokket.
- **Plassering i filen er kritisk:** disse bindingene *må* stå
  etter `hl.monitor()`-blokkene i punkt 2. Det er en kjent
  rekkefølge-bug i Hyprland 0.55.x der switch-binds ikke
  registreres korrekt hvis de evalueres før monitor-konfigen er
  satt opp (se Hyprland GH-diskusjon #14858).

**Verifiser bryternavnet på din maskin:**
```bash
hyprctl devices
```
Se etter en "switch"-enhet i output. Hvis den ikke heter nøyaktig
`"Lid Switch"`, bytt strengen i `hl.bind(...)`-kallene over.

---

## 3. Utenfor `hyprland.lua`: systemd må også få beskjed

Hyprland-config alene styrer bare *displayet*. Uten dette steget vil
**systemd fortsatt suspendere hele maskinen** når lokket lukkes,
uavhengig av at Hyprland korrekt deaktiverer `eDP-1`.

```bash
sudo mkdir -p /etc/systemd/logind.conf.d
sudo tee /etc/systemd/logind.conf.d/lid.conf <<'EOF'
[Login]
HandleLidSwitchDocked=ignore
EOF
sudo systemctl restart systemd-logind
```

**Hvorfor `HandleLidSwitchDocked` og ikke `HandleLidSwitch`:**
`HandleLidSwitchDocked` gjelder spesifikt når systemd vurderer
maskinen som "dokket" (ekstern skjerm tilkoblet). `ignore` betyr at
systemd ikke gjør noe med selve strømtilstanden i den situasjonen —
Hyprland tar seg av skjermbyttet. Uten ekstern skjerm tilkoblet
gjelder fortsatt vanlig `HandleLidSwitch` (suspend), som du
sannsynligvis vil beholde for normal laptop-bruk uten dokk.

---

## 4. Verifisering

1. Koble til ekstern skjerm, hold lokket åpent.
   → Begge skjermer aktive, `hyprctl monitors` viser begge som
   `disabled: false`.
2. Lukk lokket.
   → `hyprctl monitors` (kjørt fra en SSH-økt eller lignende, siden
   du ikke har tilgang til laptop-skjermen lenger) skal vise
   `eDP-1` som `disabled: true`, og maskinen skal **ikke**
   suspendere.
3. Åpne lokket igjen.
   → `eDP-1` skal reaktiveres automatisk med riktig
   mode/posisjon/scale.

Config leses på nytt automatisk ved lagring — ingen restart eller
utlogging nødvendig. `hyprctl reload` kan brukes for å tvinge fram
en reload manuelt ved behov.

---

## 5. Kjent fallgruve: kjøling

En Dell som står lukket permanent kan få dårligere kjøling hvis
vifteinntak/-utblåsning blir blokkert av at lokket er igjen. Verdt
å sjekke fysisk plassering på pulten før du lar den stå slik over
lengre tid.

---

## Kilder
- https://wiki.hypr.land/Configuring/Basics/Monitors/
- https://wiki.hypr.land/Configuring/Basics/Binds/
- https://github.com/hyprwm/Hyprland/discussions/14858
