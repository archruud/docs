# Søking i Linux - find, grep, locate og mer

**For:** Archruud | **Oppdatert:** 2026  
**System:** Arch Linux

---

## Oversikt

| Kommando | Brukes til |
|----------|-----------|
| `find` | Finn filer basert på navn, type, størrelse, dato |
| `grep` | Søk i innhold av filer |
| `locate` | Rask fil-søk (bruker database) |
| `which` | Finn where et program er installert |
| `whereis` | Finn program og man-sider |
| `history` | Søk i tidligere kommandoer |

---

## find - Finn filer

`find` er den kraftigste søkekommandoen. Den søker rekursivt fra et startpunkt.

### Grunnleggende syntaks

```bash
find [startmappe] [kriterier] [handling]
```

### Søke etter navn

```bash
find / -name "fil.txt"              # Finn fil med eksakt navn
find / -name "*.conf"               # Finn alle .conf filer
find / -iname "fil.txt"             # Søk uten case-sensitivitet
find /home -name "*.md"             # Søk kun i /home
find . -name "*.sh"                 # Søk i nåværende mappe
```

### Søke etter type

```bash
find / -type f                      # Kun filer
find / -type d                      # Kun mapper
find / -type l                      # Kun symlinker
find /home -type f -name "*.txt"    # Filer med .txt i /home
```

### Søke etter størrelse

```bash
find / -size +100M                  # Filer større enn 100 MB
find / -size -1k                    # Filer mindre enn 1 KB
find / -size +1G                    # Filer større enn 1 GB
find /home -size +50M               # Store filer i home
```

### Søke etter dato

```bash
find / -mtime -7                    # Endret siste 7 dager
find / -mtime +30                   # Ikke endret på 30+ dager
find / -newer fil.txt               # Nyere enn fil.txt
find /home -atime -1                # Åpnet siste 24 timer
```

### Søke etter eier og rettigheter

```bash
find / -user archruud               # Eiet av archruud
find / -perm 777                    # Med rettigheter 777
find / -perm -u+x                   # Kjørbare filer
```

### Kombinere kriterier

```bash
find /etc -type f -name "*.conf"                # Type OG navn
find / -size +100M -type f                      # Stor fil
find / -name "*.log" -mtime +7                  # Gamle logger
find / -user root -perm -4000                   # SUID filer
```

### Gjøre noe med resultatene

```bash
find /home -name "*.tmp" -delete                # Slett tmp-filer
find . -name "*.sh" -exec chmod +x {} \;        # Gjør kjørbar
find / -name "core" -exec rm {} \;              # Slett core dumps
find . -name "*.md" -exec wc -l {} \;           # Tell linjer
```

---

## grep - Søk i innhold

`grep` søker i innhold av filer — ikke filnavn.

### Grunnleggende bruk

```bash
grep "søkeord" fil.txt              # Søk i én fil
grep "søkeord" *.txt                # Søk i alle .txt filer
grep -r "søkeord" mappe/            # Søk rekursivt i mappe
grep -r "søkeord" /etc/             # Søk i /etc
```

### Nyttige flagg

```bash
grep -i "ord" fil.txt               # Ignorer store/små bokstaver
grep -n "ord" fil.txt               # Vis linjenummer
grep -c "ord" fil.txt               # Tell antall treff
grep -l "ord" *.conf                # Vis kun filnavn med treff
grep -v "ord" fil.txt               # Vis linjer UTEN søkeordet
grep -w "ord" fil.txt               # Eksakt ord (ikke del av ord)
grep -A 3 "ord" fil.txt             # 3 linjer etter treff
grep -B 3 "ord" fil.txt             # 3 linjer før treff
grep -C 3 "ord" fil.txt             # 3 linjer rundt treff
```

### Kombinere med pipe

```bash
ps aux | grep firefox               # Finn prosess
cat /var/log/syslog | grep error    # Søk i logg
ls -la | grep ".conf"               # Filtrer fillist
history | grep pacman               # Søk i kommandohistorikk
dmesg | grep -i error               # Systemfeil
journalctl | grep -i failed         # Systemd feil
```

### Regulære uttrykk med grep

```bash
grep "^start" fil.txt               # Linjer som STARTER med "start"
grep "slutt$" fil.txt               # Linjer som SLUTTER med "slutt"
grep "^$" fil.txt                   # Tomme linjer
grep "[0-9]" fil.txt                # Linjer med tall
grep "192\.168\.[0-9]*\.[0-9]*"     # IP-adresser
```

---

## locate - Rask søk

`locate` bruker en database og er mye raskere enn `find`. Databasen oppdateres automatisk (eller manuelt).

```bash
# Installer
sudo pacman -S mlocate

# Oppdater database
sudo updatedb

# Søk
locate fil.txt                      # Finn fil
locate "*.conf"                     # Finn alle .conf
locate -i "fil.txt"                 # Ignorer case
locate -n 10 "*.log"                # Maks 10 resultater
locate -c "*.md"                    # Tell antall treff
```

**Merk:** `locate` er rask men viser kanskje slettede filer (inntil databasen oppdateres).

---

## which og whereis

```bash
which python                        # Finn hvor python er installert
which vim                           # Finn vim
whereis vim                         # Finn vim + man-sider + kildekode
type ls                             # Vis om det er alias, funksjon eller program
```

---

## Søk i kommandohistorikk

```bash
history                             # Vis alle kommandoer
history | grep pacman               # Søk i historikk
history | tail -20                  # Siste 20 kommandoer

# Interaktivt søk (mens du skriver i terminalen):
Ctrl+R                              # Begynn å søke bakover
# Skriv søkeord, trykk Enter for å kjøre, Ctrl+C for å avbryte
```

---

## Søk i systemlogger

```bash
journalctl                          # Alle systemlogger
journalctl -f                       # Følg logger live
journalctl -b                       # Siden siste oppstart
journalctl -b -1                    # Forrige oppstart
journalctl -u ssh                   # Logger for SSH-tjeneste
journalctl --since "1 hour ago"     # Siste time
journalctl --since "2026-01-01"     # Fra bestemt dato
journalctl -p err                   # Kun feil
journalctl -p warning               # Kun advarsler
```

---

## Praktiske eksempler

```bash
# Finn alle konfig-filer endret i dag
find /etc -type f -name "*.conf" -mtime -1

# Finn store filer som tar plass
find / -type f -size +500M 2>/dev/null

# Finn alle scripts i hypr-mappen
find ~/.config/hypr -name "*.sh"

# Søk etter feil i systemlogger
journalctl -b | grep -i error | tail -20

# Finn filer som tilhører deg
find /home -user $USER -type f

# Søk etter IP-adresser i konfig-filer
grep -r "192\.168\." /etc/ 2>/dev/null

# Finn tomme mapper
find . -type d -empty

# Finn alle exec-once linjer i hyprland.conf
grep "exec-once" ~/.config/hypr/hyprland.conf
```

---

*Sist oppdatert: 2026 | archruud.org*
