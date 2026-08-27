# NVIDIA Datacenter GPU på Proxmox

**For:** Archruud | **Oppdatert:** 2026  
**GPUer:** 2x NVIDIA A2 (15GB) + 1x Tesla T4 (15GB) = 48GB VRAM  
**Server:** Dell PowerEdge T620

---

## GPU-oversikt

| GPU | VRAM | Bruk |
|-----|------|------|
| NVIDIA A2 (#0) | 15GB | AI/LLM |
| NVIDIA A2 (#1) | 15GB | AI/LLM |
| Tesla T4 | 15GB | AI/LLM / Inferens |

---

## Driver installasjon på Proxmox host

```bash
# Sjekk PVE-kernel (må kjøre pve-kernel)
uname -r   # Skal inneholde -pve

# Legg til non-free repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" >> /etc/apt/sources.list

# Installer NVIDIA driver
apt update
apt install pve-headers
apt install nvidia-driver

# Last inn modul
modprobe nvidia
modprobe nvidia-uvm

# Verifiser
nvidia-smi
```

---

## GPU passthrough til LXC

### Konfigurasjon i `/etc/pve/lxc/CTID.conf`

```conf
# GPU devices
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 510:* rwm

# Mount GPU devices
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia2 dev/nvidia2 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

> ⚠️ `c 510:* rwm` er for nvidia-uvm. Feil nummer (f.eks. 507) gir CUDA error 999!

### Finn riktig device-nummer

```bash
# På Proxmox host
ls -la /dev/nvidia-uvm
# crw-rw-rw- 1 root video 510, 0 ...
#                              ^^^
#                              510 = riktig nummer
```

---

## I LXC containeren

```bash
# Installer CUDA runtime (samme versjon som host driver!)
nvidia-smi   # Sjekk versjon på host
apt install nvidia-cuda-toolkit

# Test
nvidia-smi   # Skal vise alle GPUer
python3 -c "import torch; print(torch.cuda.is_available())"
```

---

## Temperaturovervåking

```bash
# Se temperatur
nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader

# Kontinuerlig overvåking
watch -n 2 nvidia-smi

# T4 advarselstemperatur: ~85°C (maks ~95°C)
```

---

## Feilsøking

```bash
# CUDA error 999 (vanligste feil)
# → Feil cgroup-nummer for nvidia-uvm
# Fix: finn riktig nummer og oppdater conf
ls -la /dev/nvidia-uvm   # Se major-nummer

# GPU ikke synlig i LXC
# → Sjekk at mount.entry-linjene finnes
cat /etc/pve/lxc/CTID.conf

# Driver-versjon mismatch
nvidia-smi   # Sjekk versjon på host
nvcc --version   # Sjekk CUDA i LXC
```

---

*Sist oppdatert: 2026 | archruud.org*
