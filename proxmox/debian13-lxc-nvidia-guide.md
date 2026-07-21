# debian13-lxc-nvidia-guide.md

# Debian 13 LXC NVIDIA GPU Guide

Denne guiden brukes etter at Proxmox-host allerede har NVIDIA-driver installert og GPU passthrough fungerer.

Testet på:

- Debian 13 (Trixie)
- Privileged LXC
- NVIDIA A2
- NVIDIA Driver 595.71.05
- Ollama

---

## 1. Oppdater container

```bash
apt update
apt full-upgrade -y
reboot
```

---

## 2. Installer verktøy

```bash
apt install -y   build-essential   dkms   wget   curl   nano   pciutils
```

---

## 3. Kontroller GPU-enheter

```bash
ls -la /dev/nvidia*
```

Du skal se:

```text
/dev/nvidia0
/dev/nvidia1
/dev/nvidiactl
/dev/nvidia-uvm
/dev/nvidia-uvm-tools
```

---

## 4. Last ned NVIDIA-driver

Eksempel:

```bash
chmod +x NVIDIA-Linux-x86_64-595.71.05.run
```

Kontroller:

```bash
./NVIDIA-Linux-x86_64-595.71.05.run --info
```

---

## 5. Installer driver

```bash
./NVIDIA-Linux-x86_64-595.71.05.run   --no-kernel-module
```

VIKTIG:

Containeren skal IKKE bygge kernelmodul.

Kernelmodulen ligger på Proxmox-hosten.

---

## 6. Test NVIDIA

```bash
nvidia-smi
```

Eksempel:

```text
GPU 0: NVIDIA A2
GPU 1: NVIDIA A2
```

---

## 7. Installer Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Kontroller:

```bash
ollama --version
```

---

## 8. Tillat nettverkstilgang

Opprett override:

```bash
mkdir -p /etc/systemd/system/ollama.service.d
nano /etc/systemd/system/ollama.service.d/override.conf
```

Innhold:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=15m"
```

Aktiver:

```bash
systemctl daemon-reload
systemctl restart ollama
```

Kontroller:

```bash
ss -tulpn | grep 11434
```

Riktig resultat:

```text
*:11434
```

---

## 9. Last ned modell

Eksempel:

```bash
ollama pull qwen3:14b
ollama pull qwen3:30b
```

Vis modeller:

```bash
ollama list
```

---

## 10. Test modell

```bash
ollama run qwen3:14b
```

---

## 11. GPU-overvåking

I nytt terminalvindu:

```bash
watch -n 1 nvidia-smi
```

---

## Resultat

Containeren fungerer som:

- AI Core
- Ollama Server
- GPU Inference Server
- API Server

Andre containere kan koble seg til:

http://IP-ADRESSE:11434
