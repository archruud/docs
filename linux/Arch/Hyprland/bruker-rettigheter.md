# Linux Bruker-Administrasjon med Docker-Rettigheter

**For:** Archruud | **Oppdatert:** 2026  
**System:** Arch Linux (primært), Debian/Ubuntu (notert der det er forskjell)

---

## Innholdsfortegnelse

1. [Grunnleggende om brukere](#grunnleggende-om-brukere)
2. [Opprette ny bruker](#opprette-ny-bruker)
3. [Sudo-rettigheter](#sudo-rettigheter)
4. [Docker-gruppe og rettigheter](#docker-gruppe-og-rettigheter)
5. [Mapperettigheter for /opt/docker](#mapperettigheter-for-optdocker)
6. [Bruker TYPE A - Full Docker + sudo](#bruker-type-a---full-docker--sudo)
7. [Bruker TYPE B - Standard sudo](#bruker-type-b---standard-sudo)
8. [Administrere brukere](#administrere-brukere)
9. [Feilsøking](#feilsøking)

---

## Grunnleggende om brukere

### Brukertyper

- **root** (UID 0) — Superbruker. Aldri logg inn direkte.
- **Systembrukere** (UID 1–999) — For tjenester (docker, nginx osv.)
- **Vanlige brukere** (UID 1000+) — Deg og andre admins.

### Viktige filer

```
/etc/passwd     - Brukerliste (ingen passord!)
/etc/shadow     - Krypterte passord
/etc/group      - Gruppemedlemskap
/etc/sudoers    - Hvem kan bruke sudo (ALDRI rediger direkte!)
```

---

## Opprette ny bruker

### Arch Linux

```bash
# Opprett bruker med hjemmemappe og bash
useradd -m -s /bin/bash brukernavn

# Sett passord
passwd brukernavn
```

### Debian/Ubuntu

```bash
# adduser er mer interaktiv og lager alt automatisk
adduser brukernavn

# Eller useradd (som Arch):
useradd -m -s /bin/bash brukernavn
passwd brukernavn
```

### Flagg forklart

```
-m          Lag hjemmemappe (/home/brukernavn)
-s /bin/bash    Sett standard shell til bash
-G docker   Legg til i gruppe ved opprettelse
-c "Navn"   Kommentar/fullt navn
```

---

## Sudo-rettigheter

### Legg til i sudo/wheel-gruppe

```bash
# Arch Linux bruker 'wheel' gruppen
usermod -aG wheel brukernavn

# Debian/Ubuntu bruker 'sudo' gruppen
usermod -aG sudo brukernavn
```

### Aktiver wheel-gruppen (Arch Linux)

```bash
# Rediger sudoers med visudo (ALDRI nano direkte!)
EDITOR=vim visudo

# Finn og uncomment denne linjen:
%wheel ALL=(ALL:ALL) ALL
```

### Verifiser sudo-tilgang

```bash
su - brukernavn
sudo whoami     # Skal returnere: root
```

---

## Docker-gruppe og rettigheter

### Legge bruker til docker-gruppen

```bash
usermod -aG docker brukernavn
```

**Viktig:** Brukeren MÅ logge ut og inn igjen for at endringen tar effekt.

### Verifiser docker-tilgang

```bash
su - brukernavn
docker ps               # Skal fungere uten sudo
docker run hello-world  # Test
```

### Sikkerhetsmerk

> ⚠️ Docker-gruppen gir i praksis root-tilgang til systemet. Kun betrodde brukere bør legges til.

---

## Mapperettigheter for /opt/docker

### Standard oppsett

```bash
# Lag /opt/docker hvis den ikke finnes
sudo mkdir -p /opt/docker

# Eierskap til bruker
sudo chown brukernavn:brukernavn /opt/docker

# Rettigheter: eier full tilgang, gruppe les+kjør
sudo chmod 755 /opt/docker

# Alternativt: gi docker-gruppen skrivetilgang
sudo chown root:docker /opt/docker
sudo chmod 775 /opt/docker
```

### For undermapper

```bash
# Gi eierskap rekursivt
sudo chown -R brukernavn:docker /opt/docker/

# Sett rettigheter rekursivt
sudo chmod -R 775 /opt/docker/
```

---

## Bruker TYPE A — Full Docker + sudo

**Bruksområde:** Docker-admin, fullstendig serverstyring.

```bash
# 1. Opprett bruker
sudo useradd -m -s /bin/bash dockeradmin
sudo passwd dockeradmin

# 2. Legg til i wheel (sudo) og docker grupper
sudo usermod -aG wheel,docker dockeradmin

# 3. Gi tilgang til /opt/docker
sudo chown -R dockeradmin:docker /opt/docker/
sudo chmod -R 775 /opt/docker/

# 4. Verifiser
su - dockeradmin
sudo whoami          # → root
docker ps            # → fungerer
ls -la /opt/docker/  # → tilgang
```

---

## Bruker TYPE B — Standard sudo

**Bruksområde:** Vanlig systemadmin uten Docker-behov.

```bash
# 1. Opprett bruker
sudo useradd -m -s /bin/bash sysadmin
sudo passwd sysadmin

# 2. Legg til i wheel (sudo) gruppen
sudo usermod -aG wheel sysadmin

# 3. Verifiser
su - sysadmin
sudo whoami          # → root
docker ps            # → Permission denied (forventet!)
```

---

## Administrere brukere

```bash
# Vis info om bruker
id brukernavn
groups brukernavn
cat /etc/passwd | grep brukernavn

# List alle brukere
cat /etc/passwd | grep -v '/sbin/nologin' | grep -v '/bin/false'

# Endre shell
usermod -s /bin/zsh brukernavn

# Lås konto
usermod -L brukernavn

# Lås opp konto
usermod -U brukernavn

# Slett bruker
userdel brukernavn          # Beholder hjemmemappe
userdel -r brukernavn       # Sletter hjemmemappe også
```

---

## Feilsøking

### "docker: permission denied"

```bash
# Sjekk at bruker er i docker-gruppen
groups $USER

# Hvis docker ikke vises → logg ut og inn igjen!
# Eller bruk ny shell:
newgrp docker
```

### "sudo: brukernavn is not in the sudoers file"

```bash
# Logg inn som root eller en annen sudo-bruker
su -
usermod -aG wheel brukernavn
```

### Sjekk rettigheter på mappe

```bash
ls -la /opt/docker/
stat /opt/docker/
```

---

*Sist oppdatert: 2026 | archruud.org*
