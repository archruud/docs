# 04 — Kitty (Terminal + Bash-oppsett)

*[English (primary)](kitty-setup.md) — norsk versjon*

## Hva det er

Setter opp bash (prompt, historikk, aliaser) og kittys eget oppsett (font, farger, tastebindinger) i ett steg.

## Hva det gjør

- Installerer `eza` (moderne erstatning for `ls`) og `bat` (syntax-highlighted `cat`) hvis de mangler
- Tar backup og erstatter `~/.bashrc`: minimalistisk mappenavn-prompt, fullt aliassett (pakkehåndtering via `yay`/`pacman`, git-snarveier, `eza`-baserte `ls`/`l`/`lt`/`lh`, config-redigeringssnarveier)
- Tar backup og erstatter `~/.config/kitty/kitty.conf`: JetBrainsMono Nerd Font i størrelse 18, powerline-fanelinje, 0.95 bakgrunnsopacitet, fornuftige kopier/lim-inn- og fane-tastebindinger

## Innstillinger den trenger for å fungere

- `hyprconf`-alias åpner `~/.config/hypr/hyprland.lua` (ikke `hyprland.conf` — ubrukt siden Lua-migreringen)
- `barconf`-alias åpner Quickshell-barens `Bar.qml` (erstatter det gamle `wayconf`-aliaset, som pekte til Waybar-config — borte siden `02-quickshell-modules`)
- Krever at JetBrainsMono Nerd Font er installert for at ikonene i `eza`-output og fanelinja skal vises riktig (allerede i `pacman-packages.txt` under fonter)

## Installasjon

```bash
chmod +x install-kitty-bash.sh
./install-kitty-bash.sh
source ~/.bashrc
```

## Bugs fikset fra originalscriptet

- `exa` → `eza`: `exa` er et nedlagt, ikke-vedlikeholdt prosjekt; `eza` er den aktivt vedlikeholdte forgreningen og det som faktisk ligger i pakkelisten din.
- `bat`-installasjonssjekken var duplisert (kopier-lim-feil) og sjekket aldri faktisk `exa`/`eza` i den andre kopien — duplikatet fjernet.
- `hyprconf`- og `wayconf`-aliasene pekte til filer som ikke lenger finnes etter Lua-migreringen og Quickshell-overgangen — pekt om til de faktiske filene som gjelder nå.
