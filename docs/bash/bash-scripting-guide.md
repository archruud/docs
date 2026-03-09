# Bash Scripting - Fra Scratch til Mester

**For:** Archruud | **Oppdatert:** 2026

---

## Hva er bash scripting?

Et bash script er en tekstfil med kommandoer som kjøres automatisk. I stedet for å skrive 10 kommandoer for hånd, kjører du én fil.

---

## Ditt første script

```bash
#!/bin/bash
# Dette er en kommentar

echo "Hei verden!"
echo "Jeg er et bash script!"
```

### Lagre og kjør

```bash
# Lagre som hello.sh
# Gjør kjørbar
chmod +x hello.sh

# Kjør
./hello.sh
```

### Shebang forklart

`#!/bin/bash` — Første linje i alle scripts. Forteller systemet hvilken tolk som skal brukes.

---

## Variabler

```bash
#!/bin/bash

# Sett variabel (ingen mellomrom rundt =)
NAVN="Archruud"
ALDER=57
MAPPE="/home/archruud"

# Bruk variabel (med $)
echo "Hei, $NAVN!"
echo "Du er $ALDER år gammel"
echo "Hjemmemappen din er $MAPPE"

# Spesielle variabler
echo "Script navn: $0"
echo "Første argument: $1"
echo "Alle argumenter: $@"
echo "Antall argumenter: $#"
echo "PID: $$"
echo "Forrige kommando status: $?"
```

---

## Input fra bruker

```bash
#!/bin/bash

# Les input
read -p "Hva heter du? " NAVN
echo "Hei, $NAVN!"

# Med timeout
read -t 10 -p "Du har 10 sekunder: " SVAR

# Skjult input (passord)
read -s -p "Passord: " PASSORD
echo ""  # Ny linje etter skjult input
```

---

## If/Else - Betingelser

```bash
#!/bin/bash

TALL=10

# Grunnleggende if
if [ $TALL -gt 5 ]; then
    echo "$TALL er større enn 5"
fi

# If/else
if [ $TALL -gt 20 ]; then
    echo "Stort tall"
else
    echo "Lite tall"
fi

# If/elif/else
if [ $TALL -gt 100 ]; then
    echo "Veldig stort"
elif [ $TALL -gt 10 ]; then
    echo "Middels"
else
    echo "Lite"
fi
```

### Sammenligningsoperatorer

```bash
# Tall
-eq     # Equal (lik)
-ne     # Not equal (ulik)
-gt     # Greater than (større)
-lt     # Less than (mindre)
-ge     # Greater or equal
-le     # Less or equal

# Tekst
=       # Lik
!=      # Ulik
-z      # Tom streng
-n      # Ikke tom streng

# Filer
-f      # Er fil
-d      # Er mappe
-e      # Eksisterer
-x      # Er kjørbar
-r      # Er lesbar
-w      # Er skrivbar
```

### Eksempel med filer

```bash
#!/bin/bash

FIL="/etc/hosts"

if [ -f "$FIL" ]; then
    echo "Filen finnes!"
    echo "Innhold:"
    cat "$FIL"
else
    echo "Filen finnes ikke!"
fi
```

---

## Loops

### For-loop

```bash
#!/bin/bash

# Enkel for-loop
for i in 1 2 3 4 5; do
    echo "Nummer: $i"
done

# Range
for i in {1..10}; do
    echo "Teller: $i"
done

# Filer
for fil in *.md; do
    echo "Fant: $fil"
done

# Kopier alle .md filer til server
SERVER="archruud@192.168.30.52"
for fil in *.md; do
    echo "Kopierer: $fil"
    scp "$fil" "$SERVER:/opt/docker/docs/"
done
```

### While-loop

```bash
#!/bin/bash

TELLER=1

while [ $TELLER -le 5 ]; do
    echo "Teller: $TELLER"
    TELLER=$((TELLER + 1))
done
```

---

## Funksjoner

```bash
#!/bin/bash

# Definer funksjon
hilsen() {
    echo "Hei, $1!"
    echo "Velkommen til bash scripting!"
}

installer_pakke() {
    PAKKE=$1
    echo "Installerer $PAKKE..."
    sudo pacman -S --noconfirm "$PAKKE"
    if [ $? -eq 0 ]; then
        echo "✓ $PAKKE installert!"
    else
        echo "✗ Feil ved installasjon av $PAKKE"
    fi
}

# Kall funksjoner
hilsen "Archruud"
installer_pakke "htop"
```

---

## Farger i terminal

```bash
#!/bin/bash

# Fargekoder
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'  # No Color (tilbakestill)

echo -e "${GREEN}✓ Suksess!${NC}"
echo -e "${RED}✗ Feil!${NC}"
echo -e "${YELLOW}⚠ Advarsel!${NC}"
echo -e "${CYAN}ℹ Info${NC}"
```

---

## Error handling

```bash
#!/bin/bash

# Avslutt ved feil
set -e

# Avslutt ved udefinert variabel
set -u

# Sjekk om kommando var vellykket
if ! kommando; then
    echo "Kommando feilet!"
    exit 1
fi

# Sjekk $? (0 = suksess, annet = feil)
ls /tmp/
if [ $? -ne 0 ]; then
    echo "Feil!"
fi

# Trap for å rydde opp
cleanup() {
    echo "Rydder opp..."
    rm -f /tmp/midlertidig_fil
}
trap cleanup EXIT
```

---

## Praktisk: Deploy-script

```bash
#!/bin/bash

# deploy-docs.sh
# Kopierer alle markdown-filer til dokumentserver

RED='\033[0;31m'
GREEN='\033[0;32m'
CYAN='\033[0;36m'
NC='\033[0m'

SERVER="archruud@192.168.30.52"
DOCS_DIR="/opt/docker/docs"
ANTALL=0
FEIL=0

echo -e "${CYAN}=== Deploy til dokumentserver ===${NC}"
echo ""

for fil in *.md; do
    if [ -f "$fil" ]; then
        echo -n "Kopierer $fil... "
        if scp "$fil" "$SERVER:$DOCS_DIR/" 2>/dev/null; then
            echo -e "${GREEN}✓${NC}"
            ANTALL=$((ANTALL + 1))
        else
            echo -e "${RED}✗ FEIL${NC}"
            FEIL=$((FEIL + 1))
        fi
    fi
done

echo ""
echo -e "${GREEN}Fullført: $ANTALL filer${NC}"
if [ $FEIL -gt 0 ]; then
    echo -e "${RED}Feil: $FEIL filer${NC}"
fi
```

---

## Praktisk: Backup-script

```bash
#!/bin/bash

# backup-hypr.sh
# Tar backup av hyprland konfig

BACKUP_DIR="$HOME/backup/hypr"
DATO=$(date +%Y-%m-%d)
KILDE="$HOME/.config/hypr"

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_DIR/hypr-$DATO.tar.gz" "$KILDE"

if [ $? -eq 0 ]; then
    echo "Backup lagret: $BACKUP_DIR/hypr-$DATO.tar.gz"
else
    echo "Backup feilet!"
    exit 1
fi

# Slett backup eldre enn 30 dager
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
echo "Gamle backups slettet"
```

---

## Debugging

```bash
#!/bin/bash

# Se hver kommando som kjøres
set -x

# Kjør script med debug
bash -x script.sh

# Eller slå på/av debug i scriptet
set -x      # Start debug
kommandoer
set +x      # Slutt debug
```

---

## Nyttig cheat sheet

```
$0          Script-navn
$1-$9       Argumenter
$@          Alle argumenter
$#          Antall argumenter
$?          Forrige kommandos status (0=OK)
$$          PID til scriptet
$HOME       Hjemmemappe
$USER       Brukernavn
$HOSTNAME   Maskinnavn

$(kommando) Kjør kommando og bruk output
$((5+3))    Matematikk → 8
${#VAR}     Lengde på variabel
```

---

*Sist oppdatert: 2026 | archruud.org*
