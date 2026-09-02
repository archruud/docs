# 19 — make-executable / `run`-alias

## Hva det er

Et lite hjelpescript som gjør `.sh`-filer kjørbare (`chmod +x`) — enten én fil eller alle i en mappe rekursivt. Installeres som kommandoen `run` i shell-et ditt.

## Hva det gjør

- Kopierer `make-executable.sh` til `~/.config/hypr/scripts/`
- Oppdager hvilken shell du bruker (bash/zsh/fish) og legger til alias `run` i riktig config-fil (`.bashrc`/`.zshrc`/`config.fish`)
- `run` uten argument → gjør alle `.sh`-filer i nåværende mappe kjørbare
- `run mappe/` → gjør alle `.sh`-filer i mappen (rekursivt) kjørbare
- `run fil.sh` → gjør bare den ene fila kjørbar

## Bugs som ble fikset

Begge de to `(j/n)`-spørsmålene i originalscriptet var **skuebrød** — de skrev spørsmålet, men hardkodet svaret til `j` uten noen gang å lese hva du faktisk skrev. Byttet til ekte `read -rp`, så spørsmålene nå faktisk venter på og bruker svaret ditt.

## Avhengigheter

Ingen — ren bash, ingen pakker kreves.

## Installasjon

```bash
chmod +x install-run-alias.sh
./install-run-alias.sh
source ~/.bashrc   # eller ~/.zshrc / config.fish
```

## Bruk etterpå

```bash
run                     # alle .sh i nåværende mappe
run ~/mine-scripts/     # alle .sh i den mappen
run script.sh           # bare den ene fila
```

## Ingen Hyprland-integrasjon nødvendig

Dette scriptet rører ikke `hyprland.lua` i det hele tatt — det er et rent shell-verktøy for terminalbruk.
