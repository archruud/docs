# NVIDIA Driver og Container Toolkit - Installasjonsdokument
## Proxmox 9.x med LXC Containere (Debian 13 Trixie)

Dato: 11. mars 2026  
Driver versjon: 590.48.01  
System: Proxmox 9.1.x med Debian 13 (Trixie) — Kernel 6.17.x

---

## ⚠️ VIKTIGE ENDRINGER FRA GAMMEL GUIDE (Proxmox 8 / Debian 12)

| Hva | Gammel (PVE 8 / Debian 12) | Ny (PVE 9 / Debian 13) |
|-----|--------------------------|----------------------|
| CUDA repo | `debian12` | `debian13` |
| Kernel headers pakke | `linux-headers-$(uname -r)` | `pve-headers-$(uname -r)` |
| `software-properties-common` | Tilgjengelig | **IKKE tilgjengelig i Trixie!** |
| NVIDIA Container Toolkit | Fungerte | **Ødelagt p.g.a. AppArmor 4** |
| `.run --no-kernel-module` i LXC | Fungerte | Fungerer fortsatt ✅ |
| Debian Trixie repo driver | N/A | 550.163.01 (fra `non-free`) |
| CUDA repo driver (590.x) | Tilgjengelig | Krever `debian13` repo |

### Hvorfor er Container Toolkit ødelagt?
Proxmox 9 bruker **AppArmor 4** (fra Debian 13). Dette bryter NVIDIA Container Toolkit sine LXC-hooks. 
Det finnes to løsninger:
1. **Anbefalt**: Bruk manuell device node binding (enklest, mest stabilt)
2. **Alternativ**: Deaktiver AppArmor for containeren (se seksjon 5)

---

## 1. PROXMOX HOST — DRIVER STATUS / NY INSTALLASJON

> **Merk:** Har du allerede kjørende `nvidia-smi` med riktig versjon på hosten, hopp til seksjon 3.

### 1.1 Sjekk nåværende status

```bash
nvidia-smi
# Viser driver versjon — noter denne, LXC container MÅ matche
```

### 1.2 Legg til CUDA repo for Debian 13

```bash
cd /tmp

# VIKTIG: Bruk debian13 (ikke debian12 som i gammel guide!)
wget https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb

# Oppdater pakkelisten
apt update
```

### 1.3 Installer PVE kernel headers

```bash
# VIKTIG: Bruk pve-headers (ikke linux-headers!)
apt install -y pve-headers-$(uname -r) build-essential
```

### 1.4 Installer NVIDIA open kernel modules

```bash
# Anbefalt for A2 og T4 (Ampere / Turing)
apt -V install nvidia-open

# Reboot
reboot
```

### 1.5 Verifiser installasjon

```bash
nvidia-smi
# Skal vise alle GPU-er: Tesla T4, NVIDIA A2 x2
```

---

## 2. PROXMOX HOST — OPPGRADERING AV EKSISTERENDE DRIVER

### 2.1 Sjekk nåværende versjon

```bash
nvidia-smi
nvidia-smi --query-gpu=driver_version --format=csv,noheader
```

### 2.2 Oppdater CUDA repo til debian13

```bash
cd /tmp

# Fjern gammel keyring (debian12)
rm -f /etc/apt/sources.list.d/cuda-*.list

# Last ned ny keyring for debian13
wget https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb

apt update
```

### 2.3 Oppgrader driver

```bash
apt upgrade -y
apt -V install nvidia-open
reboot
```

### 2.4 Verifiser ny versjon

```bash
nvidia-smi
```

---

## 3. LXC CONTAINER — NY INSTALLASJON (ANBEFALT METODE)

> Denne metoden bruker `.run --no-kernel-module` og manuell device binding.  
> **Ingen Container Toolkit** — fungerer pålitelig på Proxmox 9/Debian 13.

### 3.1 Opprett LXC Container

Via Proxmox GUI:
- OS: Debian 13 (Trixie) — eller Ubuntu 22.04/24.04
- Type: **Privileged** (anbefalt for GPU)
- Ressurser: Minimum 2GB RAM, 8GB disk

### 3.2 Konfigurer LXC Container for GPU (PÅ PROXMOX HOST)

```bash
# Stopp containeren
pct stop [CONTAINER-ID]

# Finn riktig major number for nvidia-uvm
ls -la /dev/nvidia-uvm
# Output: crw-rw-rw- 1 root root 507, 0 ...
# Tallet 507 er major number — kan variere!

# Rediger config
nano /etc/pve/lxc/[CONTAINER-ID].conf
```

Legg til disse linjene NEDERST i filen:

```ini
# GPU Device Nodes — juster antall nvidia[N] etter antall GPU i container
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia2 dev/nvidia2 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file

# GPU permissions — 195 = nvidia, 507 = nvidia-uvm (sjekk din faktiske verdi!)
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 507:* rwm
```

> **Tips:** Sjekk riktig major number med:
> ```bash
> ls -la /dev/nvidia* | awk '{print $5, $10}' | grep -v total
> ```

```bash
# Start containeren
pct start [CONTAINER-ID]
```

### 3.3 Installer NVIDIA driver i container med .run fil

```bash
# Gå inn i containeren
pct enter [CONTAINER-ID]

# Installer nødvendige pakker (uten software-properties-common — finnes ikke i Trixie!)
apt update
apt install -y wget curl ca-certificates

# Last ned SAMME versjon som hosten (f.eks. 590.48.01)
# Sjekk versjon på host med: nvidia-smi
cd /tmp
wget https://us.download.nvidia.com/tesla/590.48.01/NVIDIA-Linux-x86_64-590.48.01.run
chmod +x NVIDIA-Linux-x86_64-590.48.01.run

# Installer UTEN kernel modules (kritisk i LXC!)
./NVIDIA-Linux-x86_64-590.48.01.run --no-kernel-module --silent

# Verifiser
nvidia-smi
# Skal vise alle GPU-er som er mappet
```

---

## 4. LXC CONTAINER — CUDA REPO METODE (ALTERNATIV)

> Bruk denne hvis du foretrekker apt-pakker fremfor .run fil.

### 4.1 Legg til CUDA repo i containeren

```bash
pct enter [CONTAINER-ID]

apt update
apt install -y wget curl ca-certificates

cd /tmp
# Bruk debian13 (kritisk!)
wget https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt update
```

### 4.2 Installer compute-only driver (ingen kernel modules)

```bash
# nvidia-driver-cuda = kun compute, ingen kernel modules (perfekt for LXC)
apt install -y nvidia-driver-cuda

# Reboot container
exit
pct reboot [CONTAINER-ID]

# Test
pct enter [CONTAINER-ID]
nvidia-smi
```

---

## 5. NVIDIA CONTAINER TOOLKIT (VALGFRITT — MED APPARMOR FIX)

> **Advarsel:** Container Toolkit er ødelagt på PVE 9 med standard AppArmor.
> Trenger du Container Toolkit (f.eks. for Docker GPU-tilgang), kreves AppArmor-fix.

### 5.1 AppArmor fix for LXC container (PÅ HOST)

```bash
pct stop [CONTAINER-ID]
nano /etc/pve/lxc/[CONTAINER-ID].conf
```

Legg til:
```ini
# AppArmor fix for NVIDIA Container Toolkit på PVE 9
lxc.apparmor.profile = unconfined
```

```bash
pct start [CONTAINER-ID]
```

### 5.2 Installer NVIDIA Container Toolkit

```bash
pct enter [CONTAINER-ID]

# Installer prerequisites
apt update
apt install -y ca-certificates curl gnupg2

# Legg til GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Legg til repository (dette repo er distro-uavhengig — samme URL som før)
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Installer
apt update
apt install -y nvidia-container-toolkit

# Verifiser
nvidia-ctk --version
```

### 5.3 Konfigurer Docker runtime (hvis Docker er installert)

```bash
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker

# Test GPU i Docker
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu22.04 nvidia-smi
```

---

## 6. LXC CONTAINER — OPPGRADERING AV EKSISTERENDE DRIVER

### 6.1 Sjekk versjoner

```bash
# På HOST
nvidia-smi --query-gpu=driver_version --format=csv,noheader

# I containeren
pct enter [CONTAINER-ID]
nvidia-smi --query-gpu=driver_version --format=csv,noheader
# MÅ matche host!
```

### 6.2 Oppgrader med .run metode

```bash
pct enter [CONTAINER-ID]

cd /tmp
# Last ned ny versjon (samme som hosten)
wget https://us.download.nvidia.com/tesla/[NY-VERSJON]/NVIDIA-Linux-x86_64-[NY-VERSJON].run
chmod +x NVIDIA-Linux-x86_64-[NY-VERSJON].run

# Installer (overskriver eksisterende, ingen avinstallasjon nødvendig)
./NVIDIA-Linux-x86_64-[NY-VERSJON].run --no-kernel-module --silent

# Reboot container
exit
pct reboot [CONTAINER-ID]

# Test
pct enter [CONTAINER-ID]
nvidia-smi
```

### 6.3 Oppgrader med CUDA repo metode

```bash
pct enter [CONTAINER-ID]

# Sjekk at debian13 repo er konfigurert (ikke debian12!)
cat /etc/apt/sources.list.d/cuda-*.list | grep -i debian

apt update
apt upgrade -y
apt install -y nvidia-driver-cuda

exit
pct reboot [CONTAINER-ID]
```

---

## 7. VIKTIGE NOTATER — PROXMOX 9 SPESIFIKKE

### Driver Kompatibilitet
- **LXC container driver MÅ matche Proxmox host driver versjon**
- `nvidia-smi` på host og container skal vise identisk versjonsnummer

### Støttede GPU-er (driver 590+)
| GPU | Status |
|-----|--------|
| Tesla T4 (Turing) | ✅ Støttet |
| NVIDIA A2 (Ampere) | ✅ Støttet |
| Alle Ampere, Ada Lovelace, Hopper, Blackwell | ✅ Støttet |
| Quadro M2000 (Maxwell) | ❌ Ikke støttet i 590+ |
| Eldre Pascal og Volta | ❌ Ikke støttet i 590+ |

> For Maxwell/Pascal/Volta: bruk driver 550.x fra Debian Trixie `non-free` repo.

### Debian 13 Trixie Endringer
- `software-properties-common` **finnes ikke** — ikke installer eller avhengigheter til den
- AppArmor 4 bryter Container Toolkit hooks uten `unconfined`-profil
- CUDA debian12 repo avvises fra februar 2026 p.g.a. SHA1-signaturer
- Alltid bruk `debian13` i CUDA repo-URL
- Kernel 6.17 kan ha problemer med NVIDIA GRID/vGPU — pin 6.14 ved behov

### Kernel Pinning (hvis problemer med 6.17)

```bash
# På Proxmox host — tving 6.14 kernel
# Legg til i /etc/default/grub:
# GRUB_DEFAULT="Advanced options for Proxmox VE GNU/Linux>Proxmox VE GNU/Linux, with Linux 6.14.x-pve"
update-grub
reboot
```

---

## 8. FEILSØKING

### Problem: `nvidia-smi` viser "Failed to initialize NVML"
**Løsning:** Sjekk at GPU device nodes er mappet i LXC config (seksjon 3.2) og at containeren er restartet.

### Problem: Feil major number for cgroup
```bash
# Kjør dette på HOST for å finne riktige verdier
ls -la /dev/nvidia* /dev/nvidia-uvm 2>/dev/null | awk '{print $5, $NF}'
# Første tall = major number
```
Oppdater `lxc.cgroup2.devices.allow` linjer med riktig nummer.

### Problem: Container ser ikke GPU etter oppgradering
**Løsning:** Driver i container MATCHER ikke host. Oppgrader container driver til nøyaktig samme versjon.

### Problem: Container Toolkit fungerer ikke
**Løsning:** AppArmor-problem. Legg til `lxc.apparmor.profile = unconfined` i LXC config (seksjon 5.1).

### Problem: apt gir feil om "unusual suites" for nvidia-container-toolkit.list
```bash
# Åpne filen og sjekk at den ikke inneholder bookworm/buster etc.
cat /etc/apt/sources.list.d/nvidia-container-toolkit.list
# Slett og legg til på nytt som beskrevet i seksjon 5.2
```

### Problem: CUDA debian12 repo avvist med signaturfeil (etter feb 2026)
```bash
# Fjern gammel repo og bruk debian13
rm /etc/apt/sources.list.d/cuda-*.list
# Følg seksjon 1.2 / 4.1 med debian13 URL
```

---

## 9. RASK REFERANSE — KOMMANDOER

### På Proxmox Host

```bash
# Sjekk GPU-status
nvidia-smi

# Finn major numbers for cgroup config
ls -la /dev/nvidia* | awk '{print $5, $NF}'

# List alle LXC containere
pct list

# Start/stopp/restart container
pct start [ID]
pct stop [ID]
pct reboot [ID]

# Gå inn i container
pct enter [ID]

# Rediger LXC config
nano /etc/pve/lxc/[ID].conf

# Sjekk kernel versjon
uname -r
```

### I LXC Container

```bash
# Sjekk GPU-status
nvidia-smi

# Sjekk driver versjon
nvidia-smi --query-gpu=driver_version --format=csv,noheader

# Sjekk Container Toolkit versjon (hvis installert)
nvidia-ctk --version

# Test Docker GPU (hvis Docker er installert)
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu22.04 nvidia-smi
```

---

## 10. TEMPLATE FOR NYE CONTAINERE

Etter å ha satt opp første container riktig:

```bash
# På Proxmox host
pct stop [CONTAINER-ID]

# Konverter til template
pct template [CONTAINER-ID]

# Clone template til ny container
pct clone [TEMPLATE-ID] [NEW-ID] --full --hostname ny-container

# Rediger config for ny container (legg til GPU mappings fra seksjon 3.2)
nano /etc/pve/lxc/[NEW-ID].conf

# Start ny container
pct start [NEW-ID]
```

---

## SAMMENDRAG AV ENDRINGER FRA GAMMEL GUIDE

```
GAMMEL → NY
─────────────────────────────────────────────
debian12 → debian13  (i alle CUDA repo-URLer)
linux-headers → pve-headers  (på host)
Container Toolkit for GPU-tilgang → Manuell device binding (anbefalt)
software-properties-common → FJERNES (finnes ikke i Trixie)
```

---

**Sist oppdatert:** 11. mars 2026  
**Testet på:** Proxmox 9.1.x med Debian 13.2 (Trixie), kernel 6.17.x  
**Driver:** NVIDIA 590.48.01  
**GPU-er:** Tesla T4, NVIDIA A2 × 2
