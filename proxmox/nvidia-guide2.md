# NVIDIA GPU i Proxmox LXC Container

> Datasenter driver på Proxmox host + NVIDIA Container Toolkit i privilegert LXC container.
> Ingen manuell device mapping — Toolkit håndterer alt automatisk.

---

## Forutsetninger

| Komponent | Status |
|-----------|--------|
| Proxmox host | NVIDIA Datasenter Driver installert |
| NVIDIA Container Toolkit | Installert på host |
| LXC container | Privilegert (ikke unprivileged) |
| Driver versjon | MÅ matche host! |

Sjekk driver versjon på host:
```bash
nvidia-smi
# Øverst til høyre: driver version, f.eks. 570.124.06
```

---

## LXC Container Config

Legg til i `/etc/pve/lxc/<ID>.conf`:

```bash
# GPU tilgang
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 510:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia2 dev/nvidia2 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

> **Viktig:** `c 510:* rwm` er for `nvidia-uvm` — uten denne får du CUDA error 999!

Start container:
```bash
pct start <ID>
pct enter <ID>
```

---

## Steg 1 — Last ned driver i LXC

Samme versjon som host! Last ned `.run` fil:

```bash
# Eksempel med 570.124.06 — bytt til din versjon
cd /tmp
wget https://us.download.nvidia.com/tesla/570.124.06/NVIDIA-Linux-x86_64-570.124.06.run
chmod +x NVIDIA-Linux-x86_64-570.124.06.run
```

---

## Steg 2 — Installer driver UTEN kernel module

```bash
./NVIDIA-Linux-x86_64-570.124.06.run --no-kernel-module --silent
```

> `--no-kernel-module` er kritisk! Kernel kjører på host — ikke i containeren.

Verifiser:
```bash
nvidia-smi
# Skal vise alle GPU-ene fra host
```

---

## Steg 3 — Installer NVIDIA Container Toolkit

```bash
# Legg til repo
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

apt update
apt install -y nvidia-container-toolkit
```

Konfigurer Docker runtime:
```bash
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker
```

Test:
```bash
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu22.04 nvidia-smi
```

---

## Oppsummering — 3 kommandoer

```bash
# 1. Installer driver uten kernel
./NVIDIA-Linux-x86_64-570.124.06.run --no-kernel-module --silent

# 2. Konfigurer Docker runtime
nvidia-ctk runtime configure --runtime=docker

# 3. Restart Docker
systemctl restart docker
```

✅ Ingen config-endringer nødvendig utover LXC `.conf`
✅ Container Toolkit håndterer resten automatisk

---

## Feilsøking

### CUDA error 999
```bash
# Mangler nvidia-uvm device eller feil cgroup nummer
# Sjekk riktig major number på host:
ls -la /dev/nvidia-uvm
# c 510:0 -> 510 er riktig tall i cgroup linjen
```

### nvidia-smi virker ikke i container
```bash
# Sjekk at devices finnes på host
ls -la /dev/nvidia*

# Sjekk at container er privilegert
pct config <ID> | grep unprivileged
# Skal IKKE vise unprivileged: 1
```

### Driver versjon mismatch
```bash
# Host versjon
nvidia-smi | grep "Driver Version"

# Container versjon
cat /proc/driver/nvidia/version
# Må være identisk!
```

---

*Sist oppdatert: 2026 — Driver 570.124.06 — Proxmox 8.x*
