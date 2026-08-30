# Quickshell-moduler — oversikt

*Norsk — [English primary version](quickshell-oversikt.md)*

Valgfrie Quickshell-moduler for Hyprland. Ingen av installasjonene
rører `hyprland.lua` — du legger selv inn de nødvendige linjene, som
er tydelig markert i hver enkelt doc-fil under.

<div style="display:flex; align-items:center; gap:8px; background:rgba(46,146,219,0.1); border:1px solid #2e92db; border-radius:8px; padding:10px 14px; margin:16px 0; font-family:monospace; flex-wrap:wrap;">
  <code id="qs-install-cmd-no" style="flex:1; min-width:200px; word-break:break-all; color:#2e92db;">curl -O https://raw.githubusercontent.com/archruud/scripts/main/arch-hyprland/install-quickshell.sh && chmod +x install-quickshell.sh && ./install-quickshell.sh</code>
  <button onclick="navigator.clipboard.writeText(document.getElementById('qs-install-cmd-no').innerText); this.innerText='Kopiert!'; setTimeout(() => this.innerText='Kopier', 1500);" style="background:#2e92db; color:#1a1c22; border:none; border-radius:6px; padding:6px 14px; font-weight:bold; cursor:pointer; white-space:nowrap;">Kopier</button>
</div>

Trykk «Kopier», lim inn i terminalen din, trykk Enter.

| Modul | Doc | hyprland.lua? |
|---|---|---|
| Statusbar | [quickshell-bar.md](quickshell-bar.md) | 1 autostart-linje |
| App-oversikt | [quickshell-overview.md](quickshell-overview.md) | 1 autostart-linje + 1 keybind |
| Google Kalender (tillegg til bar) | [quickshell-kalender.md](quickshell-kalender.md) | 1 windowrule |
| Virtuelt tastatur, norsk (tillegg til bar) | [wvkbd-norsk.md](wvkbd-norsk.md) | 1 autostart-linje + 1 keybind + 1 layer rule |
| Lag eget tastatur-layout | [wvkbd-eget-layout.md](wvkbd-eget-layout.md) | — (frittstående) |

## Installasjonsscript

```bash
curl -O https://raw.githubusercontent.com/archruud/scripts/main/arch-hyprland/install-quickshell.sh
chmod +x install-quickshell.sh
./install-quickshell.sh
```

Spør interaktivt hvilke moduler du vil ha. Installerer Quickshell selv
+ alle avhengigheter. Idempotent — trygt å kjøre flere ganger,
eksisterende config tas alltid backup av først.

Etter kjøring skriver scriptet ut **alle** `hyprland.lua`-linjene du
trenger for det du valgte, samlet på ett sted — men se de enkelte
doc-filene over for forklaring på *hvorfor* og *hvor i filen* de
skal inn, ikke bare hva de er.

## Etter installasjon

```bash
hyprctl reload   # eller ditt eget reload-bind, f.eks. SUPER+SHIFT+R
```
