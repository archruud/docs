# Arch Linux Kommandoer - Komplett Guide

**For:** Archruud | **Oppdatert:** 2026  
**System:** Arch Linux, Hyprland

---

## 📋 Innholdsfortegnelse

1. [Navigasjon](#navigasjon)
2. [Filer og mapper](#filer-og-mapper)
3. [Pakkehåndtering med pacman](#pakkehåndtering-med-pacman)
4. [AUR med yay/paru](#aur-med-yayparu)
5. [Systemd og tjenester](#systemd-og-tjenester)
6. [Prosesser](#prosesser)
7. [Disk og lagring](#disk-og-lagring)
8. [Nettverk](#nettverk)
9. [Brukere og rettigheter](#brukere-og-rettigheter)
10. [Tekstbehandling](#tekstbehandling)

---

## Navigasjon

```bash
pwd             # Vis nåværende mappe (Print Working Directory)
ls              # List filer i mappen
ls -l           # List med detaljer (rettigheter, størrelse, dato)
ls -la          # List ALT, inkl. skjulte filer (starter med .)
ls -lah         # Som over, men filstørrelse lesbar (KB, MB osv)
cd /etc         # Gå til /etc mappen
cd ~            # Gå til hjemmemappen din
cd ..           # Gå ett nivå opp
cd -            # Gå tilbake til forrige mappe
```

---

## Filer og mapper

```bash
# Opprette
touch fil.txt           # Lag tom fil
mkdir mappe             # Lag mappe
mkdir -p a/b/c          # Lag hele mappestien på én gang

# Kopiere og flytte
cp fil.txt kopi.txt     # Kopier fil
cp -r mappe/ kopi/      # Kopier mappe (rekursivt)
mv fil.txt ny-fil.txt   # Flytt/rename fil
mv mappe/ /opt/         # Flytt mappe til /opt/

# Slette
rm fil.txt              # Slett fil
rm -r mappe/            # Slett mappe og alt inni
rm -rf mappe/           # Slett uten spørsmål (FORSIKTIG!)

# Se innhold
cat fil.txt             # Vis hele filen
less fil.txt            # Vis side for side (q for å avslutte)
head -n 20 fil.txt      # Vis de første 20 linjene
tail -n 20 fil.txt      # Vis de siste 20 linjene
tail -f /var/log/syslog # Følg en loggfil live

# Finne filer
find / -name "fil.txt"              # Finn fil med navn
find /home -name "*.md"             # Finn alle .md filer i /home
find /etc -type f -name "*.conf"    # Finn konfig-filer
```

---

## Pakkehåndtering med pacman

```bash
# Installere
sudo pacman -S pakkenavn            # Installer pakke
sudo pacman -S pakke1 pakke2        # Installer flere
sudo pacman -Sy                     # Oppdater pakkeliste
sudo pacman -Syu                    # Full systemoppdatering (VIKTIG!)

# Fjerne
sudo pacman -R pakkenavn            # Fjern pakke
sudo pacman -Rs pakkenavn           # Fjern pakke OG avhengigheter
sudo pacman -Rns pakkenavn          # Fjern pakke, avh. og konfig-filer

# Søke
pacman -Ss søkeord                  # Søk i repository
pacman -Qs søkeord                  # Søk blant installerte pakker
pacman -Qi pakkenavn                # Info om installert pakke
pacman -Si pakkenavn                # Info om pakke i repo

# Liste
pacman -Q                           # List alle installerte pakker
pacman -Qe                          # List manuelt installerte pakker
pacman -Qm                          # List AUR-pakker
pacman -Qdt                         # List foreldreløse pakker

# Fiks keyring (kjøres ved signaturfeil)
sudo pacman -Sy archlinux-keyring
sudo pacman-key --init
sudo pacman-key --populate archlinux
```

---

## AUR med yay/paru

```bash
# yay fungerer likt som pacman, men søker også AUR
yay -S pakkenavn                    # Installer fra AUR
yay -Syu                            # Oppdater ALT (inkl. AUR)
yay -Ss søkeord                     # Søk i AUR
yay -R pakkenavn                    # Fjern AUR-pakke

# paru (raskere, skrevet i Rust)
paru -S pakkenavn
paru -Syu
```

---

## Systemd og tjenester

```bash
# Start/stopp
sudo systemctl start tjeneste       # Start tjeneste
sudo systemctl stop tjeneste        # Stopp tjeneste
sudo systemctl restart tjeneste     # Restart tjeneste
sudo systemctl reload tjeneste      # Last inn konfig på nytt

# Enable/disable (ved oppstart)
sudo systemctl enable tjeneste      # Aktiver ved oppstart
sudo systemctl disable tjeneste     # Deaktiver ved oppstart
sudo systemctl enable --now tjeneste # Aktiver OG start nå

# Status og logging
systemctl status tjeneste           # Vis status
journalctl -u tjeneste              # Vis logger for tjeneste
journalctl -u tjeneste -f           # Følg logger live
journalctl -b                       # Logger siden siste oppstart
journalctl -p err                   # Kun feilmeldinger
```

---

## Prosesser

```bash
ps aux                  # List alle kjørende prosesser
ps aux | grep firefox   # Finn spesifikk prosess
top                     # Live prosessliste (q for å avslutte)
htop                    # Bedre versjon av top (installer: pacman -S htop)
btop                    # Enda bedre (installer: pacman -S btop)

kill PID                # Avslutt prosess med PID
kill -9 PID             # Tvangsavslutt prosess
pkill programnavn       # Avslutt prosess med navn
killall programnavn     # Avslutt alle prosesser med navn
```

---

## Disk og lagring

```bash
df -h                   # Vis diskbruk (lesbar format)
du -sh mappe/           # Vis størrelse på mappe
du -sh *                # Vis størrelse på alt i nåværende mappe
lsblk                   # List alle blokk-enheter (disker, USB)
lsblk -f                # Med filsystem-info
blkid                   # Vis UUID for alle partisjoner

# Montering
sudo mount /dev/sdb1 /mnt           # Monter partisjon
sudo umount /mnt                    # Avmonter
mount | grep sdb                    # Sjekk hva som er montert
```

---

## Nettverk

```bash
ip addr                 # Vis IP-adresser (alle grensesnitt)
ip addr show eth0       # Vis spesifikt grensesnitt
ip route                # Vis ruting-tabell
ping 8.8.8.8            # Test tilkobling
ping -c 4 google.com    # Ping 4 ganger
ss -tuln                # Vis åpne porter
curl ifconfig.me        # Vis ekstern IP
nmap 192.168.1.0/24     # Skann nettverk (installer: pacman -S nmap)
```

---

## Brukere og rettigheter

```bash
whoami                  # Hvem er du logget inn som
id                      # Vis bruker-ID og grupper
groups                  # Vis hvilke grupper du er med i
sudo kommando           # Kjør kommando som root

# Filrettigheter
chmod +x fil.sh         # Gjør fil kjørbar
chmod 755 fil.sh        # rwxr-xr-x (eier full, andre les+kjør)
chmod 644 fil.txt       # rw-r--r-- (standard for filer)
chown bruker fil.txt    # Endre eier
chown bruker:gruppe fil # Endre eier og gruppe

# Forklaring av rettigheter:
# r = les (4), w = skriv (2), x = kjør (1)
# 7 = rwx, 6 = rw-, 5 = r-x, 4 = r--
```

---

## Tekstbehandling

```bash
grep "søkeord" fil.txt          # Søk i fil
grep -r "søkeord" mappe/        # Søk rekursivt
grep -i "søkeord" fil.txt       # Søk uten case-sensitivitet
grep -n "søkeord" fil.txt       # Vis linjenummer

cat fil.txt | grep "ord"        # Filtrer output
ps aux | grep firefox           # Finn prosess
history | grep pacman           # Søk i kommandohistorikk

# Erstatt tekst
sed -i 's/gammelt/nytt/g' fil.txt       # Erstatt i fil
sed 's/gammelt/nytt/g' fil.txt          # Erstatt og vis (ikke endre fil)

# Sortering
sort fil.txt                    # Sorter linjer alfabetisk
sort -n fil.txt                 # Sorter numerisk
sort -r fil.txt                 # Reverser sortering
uniq fil.txt                    # Fjern duplikate linjer
wc -l fil.txt                   # Tell antall linjer
```

---

## Nyttige snarveier

```bash
Ctrl+C          # Avbryt kommando
Ctrl+Z          # Pause kommando (bg for å sende til bakgrunn)
Ctrl+R          # Søk i kommandohistorikk
Tab             # Autofullfør
!!              # Kjør forrige kommando igjen
sudo !!         # Kjør forrige kommando som sudo
history         # Vis kommandohistorikk
!123            # Kjør kommando nr 123 fra history
```

---

*Sist oppdatert: 2026 | archruud.org*
