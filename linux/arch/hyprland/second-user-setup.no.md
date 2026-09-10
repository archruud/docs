# Opprett bruker nr. 2 — Arch Linux + Hyprland (ren config)

🇬🇧 English version: [second-user-setup.md](second-user-setup.md)

Steg-for-steg-guide for å opprette en andre, helt isolert bruker (`test`) på en eksisterende Arch Linux + Hyprland-installasjon — med sudo-rettigheter, tilgang til alle systeminstallerte programmer, men **ingen** arv av bruker 1 sine dotfiles/Hyprland-config. Ment som en førstegangs-/testbruker.

## Mål

- Ny bruker `test`, passord `123456`
- Full sudo-tilgang
- Alle systeminstallerte programmer tilgjengelig (pacman/AUR-pakker, system-Flatpaks)
- Helt ren `~/.config` — ingenting av bruker 1 sin Hyprland/Waybar/shell-config

## Forutsetninger

- Du er logget inn som bruker 1 (eller root) med fungerende sudo
- `wheel`-gruppen er sudo-gruppen din (standard Arch-konvensjon — bekreftes under)

## Steg 1 — Opprett brukeren

```bash
sudo useradd -m -G wheel -s /bin/bash test
```

- `-m` oppretter `/home/test` på nytt fra `/etc/skel` — en helt fersk hjemmemappe, fullstendig adskilt fra bruker 1 sin `/home/<bruker1>`. Dette alene garanterer at ingen Hyprland-config arves, siden ingenting er kopiert over.
- `-G wheel` legger `test` til `wheel`-gruppen for sudo.
- `-s /bin/bash` setter login-shell. Bytt til `/bin/zsh` eller `/usr/bin/fish` hvis du heller vil teste med et annet shell (må allerede være installert).

## Steg 2 — Sett passord

```bash
sudo passwd test
```

Skriv inn `123456` to ganger. `passwd` kommer sannsynligvis til å gi en `BAD PASSWORD: too simple`-advarsel via `pam_pwquality` — som root er dette ikke fatalt, passordet settes likevel.

Valgfritt, hvis du vil bli tvunget til å bytte passord ved første innlogging i stedet for å beholde `123456`:

```bash
sudo passwd -e test
```

## Steg 3 — Bekreft at sudo er aktivert for `wheel`

```bash
sudo EDITOR=nano visudo
```

Sjekk at denne linjen er kommentert ut (uncommented):

```
%wheel ALL=(ALL:ALL) ALL
```

Verifiser at det fungerer:

```bash
su - test
sudo whoami
# skal skrive ut: root
exit
```

## Steg 4 — Bekreft at config er ren

Siden `test` har en helt ny hjemmemappe er det egentlig ingenting å fjerne — men verdt å bekrefte:

```bash
ls -la /home/test/.config 2>/dev/null
```

Denne skal være tom eller ikke eksistere ennå. Når Hyprland startes for `test` uten en `~/.config/hypr/hyprland.conf`, faller den tilbake til Hyprlands egen innebygde standardconfig — ikke bruker 1 sin config.

**Valgfritt:** hvis du heller vil at `test` skal starte fra standard-eksempelconfigen i stedet for Hyprlands helt bare interne defaults:

```bash
sudo -u test mkdir -p /home/test/.config/hypr
sudo cp /usr/share/hypr/hyprland.conf /home/test/.config/hypr/hyprland.conf
sudo chown -R test:test /home/test/.config
```

Hopp over dette hvis du vil ha den reneste mulige "ingenting konfigurert"-testen.

## Steg 5 — Tilgang til installerte programmer

I de fleste tilfeller trenger du ikke gjøre noe ekstra: pacman- og AUR-pakker (yay/paru) installeres i `/usr/bin` og registrerer desktop-oppføringer i `/usr/share/applications`, som begge er delt systemvidt og synlig for alle brukere automatisk, inkludert `test`.

To ting verdt å vite:

| Installasjonsmetode | Synlig for `test`? |
|---|---|
| `pacman -S <pakke>` | ✅ Ja — systemvidt |
| `yay`/`paru -S <aur-pakke>` | ✅ Ja — installeres i `/usr`, systemvidt |
| `flatpak install <app>` (standard, system-remote) | ✅ Ja |
| `flatpak install --user <app>` | ❌ Nei — kun synlig for brukeren som installerte den |
| systemd `--user`-tjenester satt opp av bruker 1 | ❌ Nei — per-bruker med hensikt, kjører ikke for `test` |

Hvis bruker 1 har installert Flatpaks med `--user`, vil ikke akkurat de programmene dukke opp for `test` — det er forventet isolasjon, ikke en feil.

## Steg 6 — Første innlogging i Hyprland

Avhenger av hvordan Hyprland startes på denne maskinen:

**Display-/login manager (SDDM, greetd, ly, osv.):** `test` dukker automatisk opp i brukerlisten så snart kontoen finnes. Velg den, skriv inn `123456`, velg Hyprland-session, logg inn.

**Manuell TTY-oppstart:** bytt til en ledig TTY (f.eks. `Ctrl+Alt+F2`), logg inn som `test`, kjør deretter samme kommando som normalt starter sesjonen (`Hyprland`, eller `uwsm start hyprland` hvis UWSM brukes). Siden ingen config finnes ennå (med mindre du gjorde den valgfrie kopieringen i steg 4), havner du i Hyprlands minimale innebygde defaults — nok til å bekrefte at sesjonen starter, programmer åpner, og sudo fungerer.

## Opprydding / rollback

For å fjerne testkontoen og hjemmemappen fullstendig:

```bash
sudo userdel -r test
```

## ⚠️ Sikkerhetsmerknad

`123456` er et trivielt svakt passord på en konto med full sudo-tilgang. Greit nok for en kort, lokal førstegangstest — **ikke** greit å la stå igjen, spesielt på en maskin som er nåbar over nettverk (SSH, VNC osv.). Enten slett kontoen når testingen er ferdig (`userdel -r` over), eller sett et reelt passord (`sudo passwd test`) før den blir stående.
