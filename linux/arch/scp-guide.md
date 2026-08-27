# SCP - Filoverføring mellom maskiner

**For:** Archruud | **Oppdatert:** 2026

---

## Hva er SCP?

SCP (Secure Copy Protocol) kopierer filer mellom maskiner via SSH. Sikkert, enkelt, alltid tilgjengelig.

---

## Grunnleggende syntaks

```bash
scp [kilde] [mål]
```

---

## Kopiere fra lokal til server

```bash
# Én fil
scp fil.txt bruker@server:/mål/mappe/

# Til dokumentserver
scp guide.md archruud@192.168.30.52:/opt/docker/docs/

# Til /tmp (enkel test)
scp fil.txt archruud@192.168.30.52:/tmp/

# Mappe (rekursivt med -r)
scp -r mappe/ archruud@192.168.30.52:/opt/docker/docs/
```

---

## Kopiere fra server til lokal

```bash
# Fil fra server til nåværende mappe (punktum = her)
scp archruud@192.168.30.52:/opt/docker/docs/guide.md .

# Fil til spesifikk lokal mappe
scp archruud@192.168.30.52:/etc/nginx/nginx.conf ~/backup/

# Hel mappe fra server
scp -r archruud@192.168.30.52:/opt/docker/docs/ ~/backup/
```

---

## Nyttige flagg

```bash
-r          # Rekursivt (for mapper)
-p          # Bevar filrettigheter og tidsstempel
-v          # Verbose (se hva som skjer)
-q          # Quiet (ingen output)
-C          # Komprimer under overføring
-P 2222     # Bruk annen port enn 22
```

---

## Sette opp SSH-nøkler (ingen passord)

```bash
# Generer nøkkel (trykk Enter 3 ganger)
ssh-keygen -t ed25519 -C "archruud laptop"

# Kopier nøkkel til server
ssh-copy-id archruud@192.168.30.52

# Test (skal ikke spørre om passord)
ssh archruud@192.168.30.52
```

---

## Deploy-script eksempel

```bash
#!/bin/bash
# deploy-docs.sh - Kopier alle .md filer til dokumentserver

SERVER="archruud@192.168.30.52"
DOCS_DIR="/opt/docker/docs"

echo "Deployer dokumenter..."

for fil in *.md; do
    echo "Kopierer: $fil"
    scp "$fil" "$SERVER:$DOCS_DIR/"
done

echo "Ferdig!"
```

---

## rsync - Bedre alternativ til SCP

`rsync` er smartere: overfører kun endringer.

```bash
# Synk mappe til server
rsync -avz docs/ archruud@192.168.30.52:/opt/docker/docs/

# Med sletting av filer som ikke finnes lokalt
rsync -avz --delete docs/ archruud@192.168.30.52:/opt/docker/docs/

# Dry run (se hva som ville skjedd)
rsync -avzn docs/ archruud@192.168.30.52:/opt/docker/docs/
```

---

*Sist oppdatert: 2026 | archruud.org*
