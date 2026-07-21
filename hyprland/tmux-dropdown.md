# Dropdown Tmux Terminal - Brukerveiledning

**For:** Archruud | **Oppdatert:** 2026  
**System:** Arch Linux, Hyprland, Kitty + Tmux

---

## Hva er det?

En dropdown terminal som:
- Vises/skjules med `SUPER + grave` (tasten til venstre for 1)
- Svever over andre vinduer
- Husker alle kommandoer og output (tmux session)
- Starter automatisk med Hyprland

---

## Installasjon

```bash
sudo pacman -S tmux kitty
```

---

## Hyprland konfigurasjon

Legg til i `~/.config/hypr/hyprland.conf`:

```ini
# Window rules for dropdown terminal
windowrulev2 = float,class:^(dropdown)$
windowrulev2 = size 80% 60%,class:^(dropdown)$
windowrulev2 = move 10% 5%,class:^(dropdown)$
windowrulev2 = workspace special:dropdown,class:^(dropdown)$
windowrulev2 = animation slide,class:^(dropdown)$

# Keybinding
bind = $mainMod, grave, exec, ~/.config/hypr/scripts/toggle-dropdown.sh
```

---

## Toggle script

Lag filen `~/.config/hypr/scripts/toggle-dropdown.sh`:

```bash
#!/bin/bash

# Toggle dropdown terminal med tmux

SESSION="dropdown"

# Sjekk om terminal allerede kjører
if pgrep -f "kitty.*$SESSION" > /dev/null 2>&1; then
    hyprctl dispatch togglespecialworkspace dropdown
else
    # Start ny terminal med tmux session
    kitty --class dropdown -e tmux new-session -A -s "$SESSION" &
fi
```

```bash
chmod +x ~/.config/hypr/scripts/toggle-dropdown.sh
```

---

## Autostart

Legg til i `hyprland.conf` under exec-once:

```ini
exec-once = kitty --class dropdown -e tmux new-session -A -s dropdown
```

---

## Tmux grunnleggende kommandoer

### I dropdown terminalen

```
Ctrl+b %        Del vindu vertikalt (to kolonner)
Ctrl+b "        Del vindu horisontalt (over/under)
Ctrl+b piltast  Bytt panel
Ctrl+b z        Zoom inn/ut på panel
Ctrl+b d        Detach (session kjører fortsatt i bakgrunnen)
Ctrl+b [        Scroll-modus (q for å avslutte)
```

### Sessions

```bash
tmux ls                         # List sessions
tmux new -s navn                # Ny session
tmux attach -t dropdown         # Koble til eksisterende
tmux kill-session -t navn       # Slett session
```

---

## Tips

### Kitty konfig for dropdown

Legg til i `~/.config/kitty/kitty.conf`:

```conf
# For dropdown terminal
background_opacity 0.90
```

### Endre størrelse

I `hyprland.conf`, juster:
```ini
windowrulev2 = size 90% 50%,class:^(dropdown)$   # Bredere, lavere
windowrulev2 = move 5% 5%,class:^(dropdown)$
```

---

## Feilsøking

```bash
# Sjekk om tmux session kjører
tmux ls

# Sjekk window class
hyprctl clients | grep -A5 "dropdown"

# Restart dropdown
pkill -f "kitty.*dropdown"
kitty --class dropdown -e tmux new-session -A -s dropdown &
```

---

*Sist oppdatert: 2026 | archruud.org*
