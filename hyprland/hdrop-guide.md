# hdrop - Dropdown vindu for Hyprland

**For:** Archruud | **Oppdatert:** 2026  
**System:** Arch Linux, Hyprland

---

## Hva er hdrop?

hdrop er et bash-script som bruker Hyprlands `special:hdrop` workspace som "skjulested". Du kan droppe hvilken som helst applikasjon — ikke bare terminaler.

**Logikken:**
- Program **ikke kjørende** → start det og vis det
- Program **på annet workspace** → hent det hit
- Program **allerede synlig** → skjul det

---

## Installasjon

```bash
yay -S hdrop-git
```

**Avhengigheter** (installeres automatisk): `bash`, `jq`, `awk`, `hyprctl`

---

## Flagg

```
-b   Start i bakgrunnen ved boot (exec-once)
-c   Sett klassenavn (for å kjøre flere instanser)
-f   Flytendevindu med egendefinert størrelse/posisjon
-g   Gap til skjermkant i piksler
-h   Høyde i prosent (uten %-tegn)
-p   Posisjon: top / bottom / left / right
-w   Bredde i prosent
-H   Klassenavnmatch uten case-sensitivitet
```

---

## Oppsett for Archruud (1920x1200)

### Kitty terminal fra toppen (35% høyde)

```ini
# hyprland.conf

# Autostart
exec-once = hdrop -b -f -h 35 -w 75 -p top -g 57 kitty --class kitty_top

# Keybinding
bind = $mainMod CTRL, RETURN, exec, hdrop -f -h 35 -w 75 -p top -g 57 kitty --class kitty_top
```

### Zed editor fra bunnen (85% høyde)

```ini
exec-once = hdrop -b -f -h 85 -w 90 -p bottom -g 0 zed --class zed_drop

bind = $mainMod CTRL, Down, exec, hdrop -f -h 85 -w 90 -p bottom -g 0 zed --class zed_drop
```

### Filbehandler (midten)

```ini
bind = $mainMod CTRL, F, exec, hdrop -f -h 70 -w 80 -p top -g 57 dolphin
```

---

## Feilsøking

```bash
# Sjekk om hdrop er installert
which hdrop

# Se alle hdrop-prosesser
pgrep -a hdrop

# Sjekk klassenavn på en app
hyprctl clients | grep -A5 "class"

# Manuell test
hdrop -f -h 50 -w 80 -p top -g 0 kitty
```

---

## Sammenligning med tmux dropdown

| | hdrop | tmux dropdown |
|--|-------|---------------|
| Sessjon bevares | ✓ | ✓ |
| Flere applikasjoner | ✓ | ✗ |
| Størrelsesflagg | ✓ | Krever windowrule |
| Enklere oppsett | ✓ | ✗ |

**Anbefaling:** Bruk hdrop til alt som skal droppe ned. Tmux-dropdown er fortsatt bra hvis du vil ha persistent terminal med flere paneler.

---

*Sist oppdatert: 2026 | archruud.org*
