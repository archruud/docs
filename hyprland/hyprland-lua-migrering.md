# Migrering: hyprland.conf → hyprland.lua

> **Status:** Hyprland 0.55 gjorde det gamle `hyprlang`-formatet (`.conf`) deprecated til fordel for en ekte Lua-konfig. Legacy `.conf` støttes 1-2 releaser til, men nye funksjoner kommer kun i Lua. Config-filen ligger nå på `~/.config/hypr/hyprland.lua` i stedet for `hyprland.conf`.
>
> Kilder: [hypr.land/news/26_lua](https://hypr.land/news/26_lua/), [wiki.hypr.land/Configuring/Start](https://wiki.hypr.land/Configuring/Start/)

Dette dokumentet forklarer **hver eneste linje** i `hyprland.lua` opp mot den gamle `hyprland.conf`. Målet er at du skal kjenne igjen hvert konsept fra .conf-versjonen din og se nøyaktig hvor det havnet.

---

## 0. Det aller mest grunnleggende du trenger å vite om Lua her

Du spurte om man kan kommentere i Lua i det hele tatt — ja:

```lua
-- dette er en enkeltlinje-kommentar (ikke # som i .conf!)

--[[
  dette er en
  flerlinjes blokk-kommentar
]]
```

Utover det trenger du egentlig bare 4 ting for å lese denne filen:

| Konsept | Eksempel | Forklaring |
|---|---|---|
| Variabel | `local terminal = "kitty"` | `local` = lokal variabel. Tilsvarer `$terminal = kitty` i .conf |
| Tabell | `{ key = value, key2 = value2 }` | Lua sin "alt-i-ett" datastruktur. Brukes til nesten alt API-et |
| Funksjonskall | `hl.bind("SUPER + Q", ...)` | Kaller en funksjon fra Hyprlands innebygde `hl`-API |
| Streng-sammensetting | `mainMod .. " + Q"` | `..` limer sammen to strenger (`"SUPER" .. " + Q"` → `"SUPER + Q"`) |

Alt annet (funksjoner som argumenter, løkker) forklares der de dukker opp.

Hele API-et bor i den globale tabellen `hl`. `hl.config()` setter variabler, `hl.bind()` setter keybinds, `hl.dsp.*` er dispatchers (handlingene bak et bind), `hl.on()` abonnerer på hendelser.

---

## 1. MONITORS

**Før (.conf):**
```
monitor=DP-1,1920x1080@60,0x0,1
```

**Etter (.lua):**
```lua
hl.monitor({
    output   = "DP-1",
    mode     = "1920x1080@60",
    position = "0x0",
    scale    = 1,
})
```

Den gamle komma-separerte linjen (`navn,modus,posisjon,skala`) blir en tabell med navngitte felt. Ingen overraskelser her — bare mer å skrive, men lettere å lese når man kommer tilbake til det om 6 måneder (noe jeg vet du bryr deg om).

---

## 2. MY PROGRAMS ($-variabler → local)

**Før:**
```
$terminal = kitty
$fileManager = dolphin
$menu = killall rofi || $HOME/.config/rofi/launchers/type-3/launcher.sh
```

**Etter:**
```lua
local terminal    = "kitty"
local fileManager = "dolphin"
local menu        = "killall rofi || $HOME/.config/rofi/launchers/type-3/launcher.sh"
```

`$navn` i .conf var hyprlang sin egen variabel-syntaks. I Lua er det bare en vanlig lokal variabel. `$HOME` inni selve strengen er ikke Lua-syntaks — det tolkes av **skallet** når kommandoen kjøres (bash/sh), så den kan stå akkurat som før.

---

## 3. AUTOSTART (exec-once → event-abonnement)

Dette er den **største strukturelle endringen** i hele filen.

**Før:**
```
exec-once = ~/.config/hypr/scripts/awww-wallpaper.sh
exec-once = sleep 1 && waybar
exec-once = dunst
... (7 linjer til)
```

**Etter:**
```lua
hl.on("hyprland.start", function()
    hl.exec_cmd("~/.config/hypr/scripts/awww-wallpaper.sh")
    hl.exec_cmd("sleep 1 && waybar")
    hl.exec_cmd("dunst")
    -- ... resten av kommandoene
end)
```

`exec-once` finnes ikke som eget nøkkelord lenger. I stedet finnes det et **event** kalt `"hyprland.start"` som Hyprland trigger nøyaktig én gang, ved selve oppstarten (ikke ved `hyprctl reload`). `hl.on(event, function)` betyr "når `event` skjer, kjør denne funksjonen".

`function() ... end` er en **anonym funksjon** — en funksjon uten navn, sendt direkte inn som argument. Alt mellom `function()` og `end` kjører når eventet trigges. Rekkefølgen på `hl.exec_cmd()`-kallene inni er bevart nøyaktig som i din opprinnelige liste.

---

## 4. ENVIRONMENT VARIABLES

**Før:**
```
env = XCURSOR_SIZE,24
env = HYPRCURSOR_SIZE,24
env = XDG_MENU_PREFIX,arch-
```

**Etter:**
```lua
hl.env("XCURSOR_SIZE", "24")
hl.env("HYPRCURSOR_SIZE", "24")
hl.env("XDG_MENU_PREFIX", "arch-")
```

`env = NAVN,VERDI` blir `hl.env("NAVN", "VERDI")` — to argumenter i stedet for komma inni én streng. Verdier må være strenger (i anførselstegn) selv om de ser ut som tall.

---

## 5. PERMISSIONS

Du hadde alt kommentert ut her, så det er bevart identisk, bare med Lua-kommentartegn og riktig funksjonsnavn (`hl.permission(...)`) i tilfelle du vil skru det på senere. Husk: permission-endringer krever fortsatt full restart av Hyprland, akkurat som før — ikke live-reload.

---

## 6. LOOK AND FEEL (general / decoration)

Dette er det tydeligste eksempelet på mønsteret **"seksjon → nøstet tabell"**:

**Før:**
```
general {
    gaps_in = 3
    gaps_out = 6
    border_size = 2
    col.active_border = rgba(33ccffee) rgba(00ff99ee) 45deg
    col.inactive_border = rgba(595959aa)
    resize_on_border = false
    allow_tearing = false
    layout = dwindle
}
```

**Etter:**
```lua
hl.config({
    general = {
        gaps_in = 3,
        gaps_out = 6,
        border_size = 2,
        col = {
            active_border   = { colors = { "rgba(33ccffee)", "rgba(00ff99ee)" }, angle = 45 },
            inactive_border = "rgba(595959aa)",
        },
        resize_on_border = false,
        allow_tearing = false,
        layout = "dwindle",
    },
})
```

Legg merke til:
- `general { }` blir `general = { }` **inni** ett stort `hl.config({...})`-kall. `decoration { }` fra .conf-en din havner i samme kall, som en søster-tabell til `general`.
- `col.active_border = rgba(..) rgba(..) 45deg` (to farger + vinkel for gradient) blir en egen tabell: `{ colors = {farge1, farge2}, angle = 45 }`. Dette er det eneste feltet som virkelig endrer *form*, ikke bare syntaks.
- Strenger som `"dwindle"` og fargekoder MÅ ha anførselstegn i Lua — i .conf var det ikke nødvendig.
- `shadow { }` og `blur { }` er nøstet enda ett nivå dypere, nøyaktig som i .conf, bare med `=` og komma i stedet for linjeskift.

---

## 7. ANIMATIONS

**Viktig endring du bør legge merke til:**

```
animations {
    enabled = yes, please :)
```

Dette var en godlynt hyprlang-vits/alias for `true`. Den finnes **ikke** i Lua — du må bruke ekte boolean:

```lua
hl.config({
    animations = { enabled = true },
})
```

**Bezier-kurver** (`bezier = NAVN, X0, Y0, X1, Y1`) blir egne kall til `hl.curve()`:
```lua
hl.curve("easeOutQuint", { type = "bezier", points = { {0.23, 1}, {0.32, 1} } })
```
De fire tallene fra .conf blir to punkter, `{x,y}` og `{x,y}`, i en `points`-tabell.

**Animasjons-linjene** (`animation = NAVN, ONOFF, SPEED, CURVE, [STYLE]`) blir:
```lua
hl.animation({ leaf = "windowsIn", enabled = true, speed = 4.1, bezier = "easeOutQuint", style = "popin 87%" })
```
`NAVN` → `leaf`, `ONOFF` (1/0) → ekte `true`/`false` i feltet `enabled`, resten navngis eksplisitt. Alle 18 animasjonslinjene dine er oversatt 1:1 i samme rekkefølge.

---

## 8. DWINDLE / MASTER / MISC

Samme "seksjon → nøstet tabell i hl.config()"-mønster som pkt. 6:

```lua
hl.config({ dwindle = { preserve_split = true } })
hl.config({ master  = { new_status = "master" } })
hl.config({ misc    = { force_default_wallpaper = 0, disable_hyprland_logo = true } })
```

Jeg har delt disse i separate `hl.config()`-kall for lesbarhet — du kan slå dem sammen til ett stort kall hvis du vil, `hl.config()` er kumulativ (du kan kalle den så mange ganger du vil, den bare fyller på/overskriver de feltene du oppgir).

---

## 9. INPUT (inkludert touchpad, gesture, device)

**Før:**
```
input {
    kb_layout = no
    ...
    touchpad {
        natural_scroll = true
    }
}
gesture = 3, horizontal, workspace
device {
    name = epic-mouse-v1
    sensitivity = -0.5
}
```

**Etter:**
```lua
hl.config({
    input = {
        kb_layout = "no",
        kb_variant = "", kb_model = "", kb_options = "", kb_rules = "",
        follow_mouse = 1,
        sensitivity = 0,
        touchpad = { natural_scroll = true },
    },
})

hl.gesture({ fingers = 3, direction = "horizontal", action = "workspace" })

hl.device({ name = "epic-mouse-v1", sensitivity = -0.5 })
```

Tre ting å merke seg:
1. `kb_layout = no` → `kb_layout = "no"`. I .conf var `no` bare tekst; i Lua ville `no` uten anførselstegn blitt tolket som en variabel (og gitt feil), så alt som er tekst MÅ ha `""`.
2. `gesture = ...` og `device { }` er **ikke** inni `hl.config()` — de er egne toppnivå-funksjoner (`hl.gesture()`, `hl.device()`), litt inkonsekvent med resten, men slik er API-et.
3. Tomme felt (`kb_variant =`) blir tomme strenger `""`, ikke `nil`.

---

## 10. KEYBINDINGS — det store kapittelet

Generell oversettelsesregel:

```
bind = MODS, TAST, DISPATCHER, ARGUMENT
```
blir
```lua
hl.bind("MODS + TAST", hl.dsp.<dispatcher>(<argument>))
```

### 10.1 Enkle eksempler fra din config

| .conf | .lua |
|---|---|
| `bind = $mainMod, RETURN, exec, $terminal` | `hl.bind(mainMod .. " + Return", hl.dsp.exec_cmd(terminal))` |
| `bind = $mainMod, Q, killactive,` | `hl.bind(mainMod .. " + Q", hl.dsp.window.close())` |
| `bind = $mainMod, J, layoutmsg, togglesplit` | `hl.bind(mainMod .. " + J", hl.dsp.layout("togglesplit"))` |
| `bind = , XF86PowerOff, exec, wlogout` | `hl.bind("XF86PowerOff", hl.dsp.exec_cmd("wlogout"))` |

Merk: tomt mods-felt (`,` foran tasten) betyr i .conf "ingen modifikator". I Lua utelater du bare mods-delen helt av strengen — du skriver `"XF86PowerOff"`, ikke `" + XF86PowerOff"`.

`killactive` (gammelt dispatcher-navn) er nå `hl.dsp.window.close()` — dispatchere er nå organisert i "namespaces" (`window.*`, `focus`, `layout`, osv.) i stedet for flate navn.

### 10.2 exec-binds (dine egne script-binds)

Alle dine `bind = ..., exec, <kommando>`-linjer (skjermbilder, volum, lysstyrke, `fuzzel`-script osv.) følger samme mønster: kommandoen som var etter `exec,` blir argumentet til `hl.dsp.exec_cmd("...")`. Ingenting endrer seg i selve kommandoteksten din — bare innpakningen.

### 10.3 Løkken for workspace-binds — dette er der Lua faktisk sparer deg arbeid

Du hadde 20 nesten identiske linjer:
```
bind = $mainMod, 1, workspace, 1
bind = $mainMod, 2, workspace, 2
... (opp til 9, så 0 → 10)
bind = $mainMod SHIFT, 1, movetoworkspace, 1
... (samme mønster for SHIFT)
```

I Lua blir det en `for`-løkke:
```lua
for i = 1, 10 do
    local key = i % 10 -- 10 -> tast "0" (fordi 10 modulo 10 = 0)
    hl.bind(mainMod .. " + " .. key, hl.dsp.focus({ workspace = i }))
    hl.bind(mainMod .. " + SHIFT + " .. key, hl.dsp.window.move({ workspace = i }))
end
```
`for i = 1, 10 do ... end` kjører det som er inni fra `i = 1` til `i = 10`. `i % 10` ("modulo") gir resten etter divisjon — brukes bare for at `i = 10` skal gi tastenavnet `"0"` (siden `10 % 10 = 0`), akkurat som Super+0 gikk til workspace 10 hos deg. Dette er 4 linjer i stedet for 20 — samme funksjon.

`workspace` (gammelt dispatcher) → `hl.dsp.focus({ workspace = i })`. `movetoworkspace` → `hl.dsp.window.move({ workspace = i })`.

### 10.4 Mus-binds (bindm)

```
bindm = $mainMod, mouse:272, movewindow
bindm = $mainMod, mouse:273, resizewindow
```
→
```lua
hl.bind(mainMod .. " + mouse:272", hl.dsp.window.drag(),   { mouse = true })
hl.bind(mainMod .. " + mouse:273", hl.dsp.window.resize(), { mouse = true })
```
`bindm`-prefikset ("mus-modus") blir en tredje parameter til `hl.bind()`: en opsjons-tabell `{ mouse = true }`.

### 10.5 bindel / bindl (låst skjerm + repeat)

De gamle bokstav-suffiksene til `bind` er nå opsjons-felt i stedet:

| .conf-prefiks | Betydning | Lua-opsjon |
|---|---|---|
| `bindl` | virker selv med låst skjerm | `{ locked = true }` |
| `binde` | kan holdes inne (repeterer) | `{ repeating = true }` |
| `bindel` | begge deler | `{ locked = true, repeating = true }` |

Eksempel:
```
bindel = ,XF86AudioRaiseVolume, exec, wpctl set-volume -l 1 @DEFAULT_AUDIO_SINK@ 5%+
```
→
```lua
hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd("wpctl set-volume -l 1 @DEFAULT_AUDIO_SINK@ 5%+"), { locked = true, repeating = true })
```

### 10.6 Resize med piltaster (binde + resizeactive)

```
binde = $mainMod ALT, left, resizeactive, -50 0
```
→
```lua
hl.bind(mainMod .. " + ALT + left", hl.dsp.window.resize({ x = -50, y = 0, relative = true }), { repeating = true })
```
`resizeactive` (gammelt navn) er borte som eget dispatcher-navn. Samme `hl.dsp.window.resize()` som brukes for mus-resize brukes nå også for tastatur, men med `{ x=, y=, relative=true }` i stedet for tomt kall — `relative = true` betyr "endre med dette antallet piksler" i stedet for "sett absolutt størrelse".

### 10.7 Fullscreen — flagget som usikkert

```
bind = $mainMod, F, fullscreen
```
→
```lua
hl.bind(mainMod .. " + F", hl.dsp.window.fullscreen({ mode = "fullscreen", action = "toggle" }))
```
**Vær obs her:** `hl.dsp.window.fullscreen()` er en del av Lua-APIet som fortsatt har hatt rapporterte toggle-bugs i 0.55 (funnet i GitHub-diskusjoner fra mai 2026 — ferskt, og ting kan ha blitt fikset i patcher siden). Test denne konkret: trykk Super+F to ganger og se om vinduet går tilbake til tiling. Hvis ikke, åpne `hyprctl`-terminalen og bruk den innebygde Lua-REPL-en (nevnt på wikien: "API-et og Lua-staten kan enkelt utforskes med den innebygde Lua-REPL-en i hyprctl") til å prøve varianter som `action = "set"` i stedet for `"toggle"`.

---

## 11. WINDOWS AND WORKSPACES (windowrule)

```
windowrule = match:class ^(com\.network\.manager)$, float on
windowrule = match:class ^(com\.network\.manager)$, size 450 600
windowrule = match:class ^(com\.network\.manager)$, center on
windowrule = match:class ^(com\.network\.manager)$, animation slide
```

blir **ett** kall i stedet for fire:
```lua
hl.window_rule({
    name  = "float-networkmanager",
    match = { class = "^(com\\.network\\.manager)$" },
    float = true,
    size  = { 450, 600 },
    move  = { "monitor_w/2-window_w/2", "monitor_h/2-window_h/2" },
    animation = "slide",
})
```

Ting å merke seg:
- Matchekriteriet (`class = "..."`) havner inni en egen `match = {}`-tabell — alt inni `match` MÅ stemme for at regelen skal treffe.
- Effektene (`float`, `size`, `move`, `animation`) ligger som søsken til `match`, utenfor.
- **Backslash-escaping:** i .conf skrev du `\.` for å matche et bokstavelig punktum i regex. I en Lua-streng må backslash **dobles**: `\\.`. Ellers vil Lua selv prøve å tolke escape-sekvensen. Dette er den vanligste fallgruven når man limer inn gamle regex-mønstre rett inn i Lua.
- `name = "..."` er valgfritt, men gir regelen et navn du kan slå av/på senere i kjørende sesjon via `rule:set_enabled(false)` — nyttig funksjonalitet som ikke fantes i .conf.

**⚠️ Ikke 100% bekreftet:** `size = { 450, 600 }` som eget felt fant jeg ikke et direkte dokumentert eksempel på i wiki-kildene jeg søkte gjennom (de viste kun `move` med formler for posisjonering, ikke et separat `size`-felt for dimensjoner). Jeg har derfor bygget senterings-delen (`move = {...}`) på en **bekreftet** formel-syntaks med `monitor_w`/`monitor_h`/`window_w`/`window_h`-variabler i stedet for å gjette på et `center = true`-felt. Test `size`-feltet først — hvis Hyprland klager i loggen (`hyprctl reload` eller se `journalctl`/Hyprland sin egen logg), fjern det og still størrelse via en tilsvarende formel i `move`, eller spør i r/hyprland / Hyprland sitt Discord for gjeldende syntaks i din versjon.

---

## 12. Sist: cursor { }

```
cursor {
    no_warps = true
}
```
→
```lua
hl.config({
    cursor = { no_warps = true },
})
```
Samme mønster som resten — ingen overraskelser.

---

## Oppsummering: de 5 tingene som faktisk er annerledes (ikke bare syntaks)

1. **`exec-once` finnes ikke** — bruk `hl.on("hyprland.start", function() ... end)`.
2. **`animations { enabled = yes, please :) }` fungerer ikke** — bruk ekte `true`.
3. **Dispatcher-navn er omorganisert i namespaces**: `killactive` → `window.close()`, `workspace` → `focus({workspace=..})`, `movetoworkspace` → `window.move({workspace=..})`, `resizeactive` → `window.resize({x=,y=,relative=true})`.
4. **Regex-backslash må dobles** i windowrules (`\.` → `\\.`).
5. **For-løkker** kan erstatte repeterende bind-blokker (workspace 1-10 er det klareste eksempelet).

## Praktisk fremgangsmåte for å teste

```bash
# 1. Ta backup av gjeldende config
cp ~/.config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf.bak

# 2. Legg inn den nye hyprland.lua
# (kopier filen inn i ~/.config/hypr/hyprland.lua)

# 3. Reload - config leses på nytt automatisk ved lagring, men du kan tvinge det:
hyprctl reload

# 4. Se etter Lua-feil i loggen
hyprctl monitors   # sanity check at Hyprland fortsatt kjører i det hele tatt
```

Hyprland leser `hyprland.lua` **hvis den finnes**, ellers faller den tilbake på `hyprland.conf`. Du trenger altså ikke slette `.conf`-filen med det samme — behold den som backup til du har verifisert at alle binds fungerer som forventet.
