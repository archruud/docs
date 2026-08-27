# Open WebUI + Ollama på Proxmox LXC

**For:** Archruud | **Oppdatert:** 2026  
**System:** Proxmox, LXC Container (CT 100), NVIDIA A2 x2 + T4

---

## Arkitektur

```
Proxmox Host
└── LXC Container (CT 100 - 192.168.30.x)
    ├── Ollama (port 11434)
    └── Open WebUI (port 3000)
         ↓ passthrough
NVIDIA GPU: 2x A2 (15GB) + 1x T4 (15GB) = 48GB VRAM
```

---

## LXC Container konfigurasjon

I `/etc/pve/lxc/100.conf` på Proxmox-host:

```conf
# NVIDIA GPU passthrough
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 510:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia2 dev/nvidia2 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

> ⚠️ Viktig: cgroup-linjen for nvidia-uvm MÅ være `c 510:* rwm` (ikke 507!)

---

## Installasjon i LXC

### 1. NVIDIA drivere

```bash
# Sjekk versjon på host
nvidia-smi

# Installer SAMME versjon i LXC (Debian-based)
apt update
apt install nvidia-driver-535 nvidia-cuda-toolkit
```

### 2. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
systemctl enable --now ollama

# Test GPU
ollama run llama3.1:8b --verbose
# Se etter "using CUDA"
```

### 3. Open WebUI (uten Docker)

```bash
apt install python3 python3-pip
pip install open-webui

# Start
open-webui serve --port 3000 --host 0.0.0.0
```

---

## Open WebUI innstillinger (VIKTIG for ytelse)

Disse innstillingene må skrus **AV** for å hindre at Ollama konstant laster av/på modeller:

Gå til `Settings → Interface`:
- ❌ **AutoComplete Generation** → AV
- ❌ **Search Query Generation** → AV
- ❌ **Title Generation** → AV

Gå til `Settings → RAG`:
- ❌ **Enable RAG** → AV (med mindre du bruker det)

---

## Ollama ytelsesinnstillinger

Fil: `/etc/systemd/system/ollama.service.d/override.conf`

```ini
[Service]
Environment="OLLAMA_KEEP_ALIVE=30m"
Environment="OLLAMA_FLASH_ATTENTION=true"
Environment="OLLAMA_NUM_PARALLEL=2"
```

```bash
systemctl daemon-reload
systemctl restart ollama
```

---

## Anbefalte modeller (48GB VRAM)

```bash
ollama pull qwen2.5:14b              # Generell bruk
ollama pull LTG/normistral-11b-thinking   # Norsk
ollama pull deepseek-r1:32b          # Avansert reasoning
ollama pull llama3.1:8b              # Rask og lett
```

---

## Tilgang

| Tjeneste | URL |
|----------|-----|
| Open WebUI | http://192.168.30.X:3000 |
| Ollama API | http://192.168.30.X:11434 |

---

## Feilsøking

```bash
# GPU ikke funnet i LXC
nvidia-smi                      # Skal vise GPU-er
ls /dev/nvidia*                 # Skal vise devices

# CUDA error 999 (vanligste feil)
# → Sjekk cgroup: c 510:* rwm (IKKE 507!)
cat /etc/pve/lxc/100.conf | grep cgroup

# Sjekk Ollama logger
journalctl -u ollama -f

# Sjekk Open WebUI
journalctl -u open-webui -f
```

---

*Sist oppdatert: 2026 | archruud.org*
