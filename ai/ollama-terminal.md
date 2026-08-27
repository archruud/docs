# Ollama Terminal Guide

**For:** Archruud | **Oppdatert:** 2026  
**Kategori:** AI Stuff  
**Server:** Proxmox LXC (CT 100) | NVIDIA A2 x2 + T4 = 48GB VRAM

---

## Grunnleggende kommandoer

### Start interaktiv chat

```bash
# Chat med en modell
ollama run qwen2.5:14b

# Chat med norsk modell
ollama run LTG/normistral-11b-thinking

# Avslutt chat
/bye
```

### Last ned modeller

```bash
ollama pull qwen2.5:14b
ollama pull LTG/normistral-11b-thinking
ollama pull llama3.1:8b
```

---

## Administrere modeller

```bash
# List alle installerte modeller
ollama list

# Vis info om modell
ollama show qwen2.5:14b

# Slett modell
ollama rm qwen2.5:14b

# Sjekk hvilke modeller som kjører
ollama ps
```

---

## Systemd-tjeneste

```bash
# Status
systemctl status ollama

# Restart
sudo systemctl restart ollama

# Logger
journalctl -u ollama -f
```

---

## API-bruk fra terminal

```bash
# Enkel test
curl http://localhost:11434/api/generate \
  -d '{"model": "qwen2.5:14b", "prompt": "Hei! Hva er 2+2?", "stream": false}'

# Chat-format (bedre for samtaler)
curl http://localhost:11434/api/chat \
  -d '{
    "model": "qwen2.5:14b",
    "messages": [{"role": "user", "content": "Forklar meg hva Linux er på norsk"}],
    "stream": false
  }'

# List modeller via API
curl http://localhost:11434/api/tags
```

---

## Norske modeller

### NorMistral-11B (UiO)

```bash
# Last ned
ollama pull LTG/normistral-11b-thinking

# Test
ollama run LTG/normistral-11b-thinking
# Skriv: "Forklar meg hva Hyprland er"
```

Trent på norsk (bokmål og nynorsk) av Universitetet i Oslo. Best for norske tekster, lovverk og faglig norsk.

---

## Anbefalte modeller for 48GB VRAM

| Modell | Størrelse | Bruk |
|--------|-----------|------|
| `qwen2.5:14b` | ~9GB | Generell bruk, koding |
| `LTG/normistral-11b-thinking` | ~7GB | Norsk tekst |
| `llama3.1:8b` | ~5GB | Rask, generell |
| `qwen2.5:72b` | ~45GB | Maksimal kvalitet |
| `deepseek-r1:32b` | ~20GB | Reasoning |

---

## Ytelsestips

```bash
# Hold modell i minnet (ikke last av)
OLLAMA_KEEP_ALIVE=30m

# Sett i /etc/systemd/system/ollama.service.d/override.conf:
[Service]
Environment="OLLAMA_KEEP_ALIVE=30m"
Environment="OLLAMA_FLASH_ATTENTION=true"
```

---

## Feilsøking

```bash
# Sjekk GPU-bruk
nvidia-smi

# Se om Ollama bruker GPU
ollama run llama3.1:8b --verbose
# Se etter: "using CUDA"

# Sjekk logger
journalctl -u ollama -n 50
```

---

*Sist oppdatert: 2026 | archruud.org*
