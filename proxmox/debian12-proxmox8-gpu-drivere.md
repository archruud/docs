# NVIDIA Driver og Container Toolkit - Installasjonsdokument
## Proxmox med LXC Containere

Dato: 19. desember 2025  
Driver versjon: 590.44.01  
System: Proxmox med Debian 12 (Bookworm)

---

## 1. PROXMOX HOST - NY INSTALLASJON

### 1.1 Installer NVIDIA Datacenter Driver

```bash
# Last ned cuda-keyring
cd /tmp
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb

# Oppdater pakkelisten
apt update

# Installer Proxmox kernel headers (viktig!)
apt install -y pve-headers-$(uname -r)

# Aktiver contrib repository (hvis ikke allerede gjort)
# Sjekk /etc/apt/sources.list - contrib skal være inkludert

# Installer NVIDIA open kernel modules (anbefalt)
apt -V install nvidia-open

# Reboot
reboot
```

### 1.2 Verifiser installasjon

```bash
# Etter reboot
nvidia-smi

# Skal vise alle GPU-er med korrekt driver versjon
```

### 1.3 Installer NVIDIA Container Toolkit (valgfritt, men anbefalt)

```bash
# Legg til GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Legg til repository
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Installer
apt update
apt install -y nvidia-container-toolkit

# Verifiser
nvidia-ctk --version
```

---

## 2. PROXMOX HOST - OPPGRADERING AV EKSISTERENDE DRIVER

### 2.1 Sjekk nåværende versjon

```bash
nvidia-smi
# Noter versjonsnummer
```

### 2.2 Oppgrader driver

```bash
cd /tmp

# Last ned cuda-keyring hvis ikke allerede gjort
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb

# Oppdater pakkelisten
apt update

# Oppgrader alle systempakker
apt upgrade -y

# Oppgrader NVIDIA driver
apt -V install nvidia-open

# Reboot
reboot
```

### 2.3 Verifiser ny versjon

```bash
nvidia-smi
# Skal vise ny versjon
```

---

## 3. LXC CONTAINER - NY INSTALLASJON

### 3.1 Opprett LXC Container

Via Proxmox GUI:
- OS: Debian 12 (Bookworm)
- Type: **Privileged** (anbefalt for GPU)
- Ressurser: Minimum 2GB RAM, 8GB disk

### 3.2 Installer NVIDIA Driver i Container

```bash
# Gå inn i containeren
pct enter [CONTAINER-ID]

# Oppdater system
apt update
apt upgrade -y

# Installer cuda-keyring
cd /tmp
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb
apt update

# Installer NVIDIA driver (COMPUTE-ONLY, ingen kernel modules)
apt install -y nvidia-driver-cuda

# Reboot container
exit
pct reboot [CONTAINER-ID]
```

### 3.3 Installer NVIDIA Container Toolkit

```bash
# Gå inn i containeren igjen
pct enter [CONTAINER-ID]

# Legg til GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Legg til repository
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Installer
apt update
apt install -y nvidia-container-toolkit

# Verifiser
nvidia-ctk --version
```

### 3.4 Konfigurer LXC Container (PÅ PROXMOX HOST)

```bash
# Stopp containeren
pct stop [CONTAINER-ID]

# Rediger config
nano /etc/pve/lxc/[CONTAINER-ID].conf

# Legg til disse linjene nederst i filen:

# GPU Device Nodes
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia2 dev/nvidia2 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file

# GPU permissions
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 507:* rwm

# Lagre: Ctrl+O, Enter, Ctrl+X

# Start containeren
pct start [CONTAINER-ID]
```

### 3.5 Verifiser GPU-tilgang

```bash
# Gå inn i containeren
pct enter [CONTAINER-ID]

# Test GPU
nvidia-smi

# Skal vise alle GPU-er
```

### 3.6 Konfigurer Docker (valgfritt)

```bash
# Hvis du skal bruke Docker i containeren:

# Installer Docker Community Edition (anbefalt over docker.io)

# Installer nødvendige pakker
apt install -y ca-certificates curl gnupg

# Legg til Docker's offisielle GPG key
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Legg til Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installer Docker CE
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Konfigurer Docker runtime for NVIDIA
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker

# Test GPU i Docker
docker run --rm --gpus all nvidia/cuda:13.1-base-ubuntu22.04 nvidia-smi
```

---

## 4. LXC CONTAINER - OPPGRADERING AV EKSISTERENDE DRIVER

### 4.1 Sjekk nåværende versjon

```bash
# I containeren
nvidia-smi
# Noter versjonsnummer
```

### 4.2 Oppgrader driver

```bash
# I containeren
cd /tmp

# Last ned cuda-keyring hvis ikke allerede gjort
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
dpkg -i cuda-keyring_1.1-1_all.deb

# Oppdater pakkelisten
apt update

# Oppgrader systempakker
apt upgrade -y

# Oppgrader NVIDIA driver
apt install -y nvidia-driver-cuda

# Reboot container
exit
pct reboot [CONTAINER-ID]
```

### 4.3 Verifiser ny versjon

```bash
pct enter [CONTAINER-ID]
nvidia-smi
# Skal vise ny versjon (må matche host driver versjon)
```

---

## 5. VIKTIGE NOTATER

### Driver Kompatibilitet
- **LXC container driver MÅ matche Proxmox host driver versjon**
- Sjekk alltid `nvidia-smi` på både host og container etter oppgradering

### Støttede GPU-er (driver 590+)
- ✅ Tesla T4 (Turing)
- ✅ NVIDIA A2 (Ampere)
- ✅ Alle Ampere, Ada Lovelace, Hopper, Blackwell
- ❌ Quadro M2000 (Maxwell) - IKKE støttet i 590+
- ❌ Eldre Pascal og Volta kort - IKKE støttet i 590+

### Docker Versjon
- **docker-ce** (Docker Community Edition) - Anbefalt, nyeste versjon fra Docker
- **docker.io** (Debian pakke) - Eldre versjon, ikke anbefalt
- Dokumentet bruker docker-ce fra Docker's offisielle repository

### LXC Container Type
- **Privileged containere** anbefales for GPU-bruk
- Unprivileged containere krever ekstra konfigurering med uid/gid mapping

### Troubleshooting

**Problem:** `nvidia-smi` viser "Failed to initialize NVML"
**Løsning:** Sjekk at GPU device nodes er mappet i LXC config (seksjon 3.4)

**Problem:** Container ser ikke GPU-ene
**Løsning:** Verifiser at `lxc.cgroup2.devices.allow` linjer er lagt til i config

**Problem:** Docker ser ikke GPU-ene
**Løsning:** Sjekk at `nvidia-ctk runtime configure --runtime=docker` er kjørt

---

## 6. RASK REFERANSE - KOMMANDOER

### På Proxmox Host

```bash
# Sjekk GPU-status
nvidia-smi

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
```

### I LXC Container

```bash
# Sjekk GPU-status
nvidia-smi

# Sjekk NVIDIA driver versjon
nvidia-smi --query-gpu=driver_version --format=csv,noheader

# Sjekk Container Toolkit versjon
nvidia-ctk --version

# Test Docker GPU
docker run --rm --gpus all nvidia/cuda:13.1-base-ubuntu22.04 nvidia-smi
```

---

## 7. TEMPLATE FOR NYE CONTAINERE

Etter å ha satt opp første container riktig:

```bash
# På Proxmox host
# Stopp containeren
pct stop [CONTAINER-ID]

# Konverter til template
pct template [CONTAINER-ID]

# Clone template til ny container
pct clone [TEMPLATE-ID] [NEW-ID] --full --hostname ny-container

# Rediger config for ny container (legg til GPU mappings)
nano /etc/pve/lxc/[NEW-ID].conf

# Start ny container
pct start [NEW-ID]
```

---

## SLUTT

Dette dokumentet dekker:
- ✅ Ny installasjon av NVIDIA driver på Proxmox host
- ✅ Oppgradering av driver på Proxmox host  
- ✅ Ny installasjon av driver i LXC container
- ✅ Oppgradering av driver i LXC container
- ✅ NVIDIA Container Toolkit setup
- ✅ LXC konfigurering for GPU passthrough
- ✅ Docker integrering

**Sist oppdatert:** 19. desember 2025  
**Testet på:** Proxmox 8.x med Debian 12 (Bookworm)  
**Driver:** NVIDIA 590.44.01
